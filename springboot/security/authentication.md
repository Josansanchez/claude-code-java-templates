# Authentication with JWT

## Overview
The system uses JWT (JSON Web Tokens) stored in HTTP cookies for authentication.

## JWT Configuration

### Application Properties
```yaml
myapp:
  app:
    jwtSecret: ${JWT_SECRET}  # From environment variable
    jwtExpirationMs: 86400000  # 24 hours
    jwtCookieName: myapp
```

**IMPORTANT**: In production, ALWAYS use environment variables for `jwtSecret`.

## JWT Cookie Strategy

### Why Cookies?
- More secure than localStorage (HttpOnly, Secure flags)
- Automatic sending with requests
- Protection against XSS attacks
- Can be configured with SameSite policy

### Cookie Configuration
```java
@Configuration
public class JwtConfig {

    @Value("${myapp.app.jwtCookieName}")
    private String jwtCookieName;

    @Value("${myapp.app.jwtExpirationMs}")
    private int jwtExpirationMs;

    public ResponseCookie generateJwtCookie(String jwt) {
        return ResponseCookie.from(jwtCookieName, jwt)
            .path("/api")
            .maxAge(24 * 60 * 60)  // 24 hours
            .httpOnly(true)
            .secure(true)  // HTTPS only in production
            .sameSite("Strict")
            .build();
    }

    public ResponseCookie getCleanJwtCookie() {
        return ResponseCookie.from(jwtCookieName, "")
            .path("/api")
            .maxAge(0)
            .httpOnly(true)
            .build();
    }
}
```

## Authentication Flow

### 1. Login
```java
@RestController
@RequestMapping("/api/v1/auth")
public class LoginController {

    private final LoginService loginService;

    @PostMapping("/signin")
    public ResponseEntity<UserInfoResponse> login(@Valid @RequestBody LoginRequest request) {
        UserInfoResponse response = loginService.login(request);
        return ResponseEntity.ok()
            .header(HttpHeaders.SET_COOKIE, response.getJwtCookie().toString())
            .body(response);
    }
}
```

### 2. Login Service
```java
@Service
@Slf4j
public class LoginService {

    private final AuthenticationManager authenticationManager;
    private final JwtUtils jwtUtils;
    private final UserDetailsServiceImpl userDetailsService;

    @Transactional
    public UserInfoResponse login(LoginRequest request) {
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            )
        );

        SecurityContextHolder.getContext().setAuthentication(authentication);

        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        String jwt = jwtUtils.generateJwtToken(userDetails);
        ResponseCookie jwtCookie = jwtUtils.generateJwtCookie(jwt);

        List<String> roles = userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList());

        return UserInfoResponse.builder()
            .id(userDetails.getId())
            .username(userDetails.getUsername())
            .email(userDetails.getEmail())
            .roles(roles)
            .jwtCookie(jwtCookie)
            .build();
    }
}
```

### 3. JWT Token Generation
```java
@Component
public class JwtUtils {

    @Value("${myapp.app.jwtSecret}")
    private String jwtSecret;

    @Value("${myapp.app.jwtExpirationMs}")
    private int jwtExpirationMs;

    public String generateJwtToken(UserDetailsImpl userPrincipal) {
        return Jwts.builder()
            .setSubject(userPrincipal.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpirationMs))
            .signWith(SignatureAlgorithm.HS512, jwtSecret)
            .compact();
    }

    public String getUsernameFromJwtToken(String token) {
        return Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }

    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(authToken);
            return true;
        } catch (SignatureException e) {
            log.error("Invalid JWT signature: {}", e.getMessage());
        } catch (MalformedJwtException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            log.error("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            log.error("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            log.error("JWT claims string is empty: {}", e.getMessage());
        }
        return false;
    }
}
```

## JWT Authentication Filter

```java
@Component
public class AuthTokenFilter extends OncePerRequestFilter {

    @Autowired
    private JwtUtils jwtUtils;

    @Autowired
    private UserDetailsServiceImpl userDetailsService;

    @Value("${myapp.app.jwtCookieName}")
    private String jwtCookieName;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        try {
            String jwt = getJwtFromCookies(request);

            if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
                String username = jwtUtils.getUsernameFromJwtToken(jwt);

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );

                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                );

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            logger.error("Cannot set user authentication: {}", e);
        }

        filterChain.doFilter(request, response);
    }

    private String getJwtFromCookies(HttpServletRequest request) {
        Cookie[] cookies = request.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (jwtCookieName.equals(cookie.getName())) {
                    return cookie.getValue();
                }
            }
        }
        return null;
    }
}
```

## UserDetailsService Implementation

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserRepository userRepository;

    public UserDetailsServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    @Transactional
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() ->
                new UsernameNotFoundException("User not found: " + username)
            );

        return UserDetailsImpl.build(user);
    }
}
```

## UserDetails Implementation

```java
public class UserDetailsImpl implements UserDetails {

    private Long id;
    private String username;
    private String email;
    private String password;
    private Collection<? extends GrantedAuthority> authorities;

    public static UserDetailsImpl build(User user) {
        List<GrantedAuthority> authorities = user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority(role.getName().name()))
            .collect(Collectors.toList());

        return new UserDetailsImpl(
            user.getId(),
            user.getUsername(),
            user.getEmail(),
            user.getPassword(),
            authorities
        );
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    // Getters, equals, hashCode
}
```

## Logout

```java
@RestController
@RequestMapping("/api/v1/auth")
public class SignoutController {

    private final JwtUtils jwtUtils;

    @PostMapping("/signout")
    public ResponseEntity<MessageResponse> logout() {
        ResponseCookie cookie = jwtUtils.getCleanJwtCookie();
        return ResponseEntity.ok()
            .header(HttpHeaders.SET_COOKIE, cookie.toString())
            .body(new MessageResponse("You've been signed out!"));
    }
}
```

## Getting Current User

Utility method to get logged-in user:

```java
public class Utils {
    public static String getLoggedUsername() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication != null && authentication.getPrincipal() instanceof UserDetailsImpl) {
            UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
            return userDetails.getUsername();
        }
        throw new UnauthorizedException("User not authenticated");
    }
}
```

## Password Encoding

**ALWAYS** use `BCryptPasswordEncoder` for passwords:

```java
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Encoding Passwords
```java
@Service
public class SignupService {

    private final PasswordEncoder passwordEncoder;

    public void createUser(SignupRequest request) {
        User user = User.builder()
            .username(request.getUsername())
            .password(passwordEncoder.encode(request.getPassword()))  // ✅ Encode
            .build();
        // ...
    }
}
```

## Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class WebSecurityConfig {

    @Autowired
    private AuthTokenFilter authTokenFilter;

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            );

        http.addFilterBefore(authTokenFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

## Best Practices

1. ✅ Use environment variables for JWT secret in production
2. ✅ Store JWT in HttpOnly cookies
3. ✅ Use secure cookies (HTTPS) in production
4. ✅ Set appropriate cookie expiration
5. ✅ Always encode passwords with BCryptPasswordEncoder
6. ✅ Validate JWT on every request
7. ✅ Use stateless session management
8. ✅ Clear cookies on logout
9. ❌ NO hardcoded secrets
10. ❌ NO storing JWT in localStorage (use cookies)
11. ❌ NO plain text passwords

## Related Documentation
- See [authorization.md](./authorization.md) for role-based access control
- See [cors.md](./cors.md) for CORS configuration
- See [profiles.md](../configuration/profiles.md) for environment configuration
