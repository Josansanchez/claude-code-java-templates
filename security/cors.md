# CORS Configuration

## Overview
CORS (Cross-Origin Resource Sharing) must be configured globally in `WebSecurityConfig`. **DO NOT use `@CrossOrigin` on individual controllers.**

## Global CORS Configuration

### WebSecurityConfig
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class WebSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );

        http.addFilterBefore(authTokenFilter(), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        // Allowed origins
        configuration.setAllowedOrigins(Arrays.asList(
            "http://localhost:4200",      // Angular dev
            "http://localhost:3000",      // React dev
            "https://yourdomain.com"      // Production
        ));

        // Allowed HTTP methods
        configuration.setAllowedMethods(Arrays.asList(
            "GET",
            "POST",
            "PUT",
            "DELETE",
            "OPTIONS"
        ));

        // Allowed headers
        configuration.setAllowedHeaders(Arrays.asList("*"));

        // Allow credentials (cookies, authorization headers)
        configuration.setAllowCredentials(true);

        // How long browser can cache preflight response (in seconds)
        configuration.setMaxAge(3600L);

        // Expose headers to the client
        configuration.setExposedHeaders(Arrays.asList(
            "Authorization",
            "Set-Cookie"
        ));

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);

        return source;
    }
}
```

## Configuration Options

### Allowed Origins
```java
// Single origin
configuration.setAllowedOrigins(Arrays.asList("http://localhost:4200"));

// Multiple origins
configuration.setAllowedOrigins(Arrays.asList(
    "http://localhost:4200",
    "https://app.example.com"
));

// All origins (NOT recommended for production)
configuration.setAllowedOrigins(Arrays.asList("*"));

// Pattern-based origins
configuration.setAllowedOriginPatterns(Arrays.asList("https://*.example.com"));
```

### Allowed Methods
```java
// Specific methods
configuration.setAllowedMethods(Arrays.asList(
    "GET",
    "POST",
    "PUT",
    "DELETE",
    "OPTIONS"
));

// All methods (NOT recommended)
configuration.setAllowedMethods(Arrays.asList("*"));
```

### Allowed Headers
```java
// All headers
configuration.setAllowedHeaders(Arrays.asList("*"));

// Specific headers
configuration.setAllowedHeaders(Arrays.asList(
    "Content-Type",
    "Authorization",
    "X-Requested-With"
));
```

### Allow Credentials
```java
// Required for cookies and authorization headers
configuration.setAllowCredentials(true);

// Note: Cannot use allowedOrigins("*") with allowCredentials(true)
// Use allowedOriginPatterns instead
```

### Exposed Headers
```java
// Headers that client can access
configuration.setExposedHeaders(Arrays.asList(
    "Authorization",
    "Set-Cookie",
    "Location"
));
```

### Max Age
```java
// Cache preflight response for 1 hour
configuration.setMaxAge(3600L);
```

## Environment-Specific CORS

### Development
```java
@Configuration
@Profile("dev")
public class CorsConfigDev {

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList(
            "http://localhost:4200",
            "http://localhost:3000"
        ));
        configuration.setAllowedMethods(Arrays.asList("*"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

### Production
```java
@Configuration
@Profile("prod")
public class CorsConfigProd {

    @Value("${app.cors.allowed-origins}")
    private String[] allowedOrigins;

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList(allowedOrigins));
        configuration.setAllowedMethods(Arrays.asList(
            "GET",
            "POST",
            "PUT",
            "DELETE"
        ));
        configuration.setAllowedHeaders(Arrays.asList(
            "Content-Type",
            "Authorization"
        ));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);
        return source;
    }
}
```

### application.yml
```yaml
# application-prod.yml
app:
  cors:
    allowed-origins:
      - https://app.example.com
      - https://admin.example.com
```

## Common CORS Issues

### Issue 1: Credentials with Wildcard
```java
// ❌ INCORRECT - Cannot combine credentials with wildcard
configuration.setAllowedOrigins(Arrays.asList("*"));
configuration.setAllowCredentials(true);

// ✅ CORRECT - Use specific origins or patterns
configuration.setAllowedOriginPatterns(Arrays.asList("*"));
configuration.setAllowCredentials(true);

// ✅ BETTER - Use specific origins
configuration.setAllowedOrigins(Arrays.asList("http://localhost:4200"));
configuration.setAllowCredentials(true);
```

### Issue 2: Missing OPTIONS Method
```java
// ✅ Always include OPTIONS for preflight requests
configuration.setAllowedMethods(Arrays.asList(
    "GET",
    "POST",
    "PUT",
    "DELETE",
    "OPTIONS"  // Required for preflight
));
```

### Issue 3: CSRF Disabled
```java
// When using JWT in cookies, CSRF can be disabled
http.csrf(csrf -> csrf.disable());

// But consider enabling for non-JWT endpoints
http.csrf(csrf -> csrf
    .ignoringRequestMatchers("/api/v1/auth/**")
);
```

## Testing CORS

### Manual Test with cURL
```bash
# Preflight request
curl -X OPTIONS http://localhost:8080/api/v1/user \
  -H "Origin: http://localhost:4200" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  -v

# Actual request
curl -X POST http://localhost:8080/api/v1/user \
  -H "Origin: http://localhost:4200" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test"}' \
  -v
```

### Integration Test
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class CorsIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void shouldAllowCorsFromAllowedOrigin() {
        HttpHeaders headers = new HttpHeaders();
        headers.setOrigin("http://localhost:4200");

        ResponseEntity<String> response = restTemplate.exchange(
            "/api/v1/user",
            HttpMethod.OPTIONS,
            new HttpEntity<>(headers),
            String.class
        );

        assertThat(response.getHeaders().getAccessControlAllowOrigin())
            .isEqualTo("http://localhost:4200");
    }

    @Test
    void shouldBlockCorsFromDisallowedOrigin() {
        HttpHeaders headers = new HttpHeaders();
        headers.setOrigin("http://malicious-site.com");

        ResponseEntity<String> response = restTemplate.exchange(
            "/api/v1/user",
            HttpMethod.OPTIONS,
            new HttpEntity<>(headers),
            String.class
        );

        assertThat(response.getHeaders().getAccessControlAllowOrigin())
            .isNull();
    }
}
```

## Best Practices

1. ✅ Configure CORS globally in `WebSecurityConfig`
2. ✅ Use specific allowed origins in production
3. ✅ Include `OPTIONS` method for preflight
4. ✅ Set `allowCredentials(true)` when using cookies
5. ✅ Use environment-specific configurations
6. ✅ Set appropriate `maxAge` for preflight caching
7. ✅ Expose necessary headers to client
8. ✅ Use `allowedOriginPatterns` with credentials
9. ❌ NO `@CrossOrigin` on controllers
10. ❌ NO wildcard origins with credentials
11. ❌ NO overly permissive CORS in production

## Security Considerations

### Production Security
```java
// ✅ GOOD - Specific origins
configuration.setAllowedOrigins(Arrays.asList(
    "https://app.example.com",
    "https://admin.example.com"
));

// ✅ GOOD - Pattern matching
configuration.setAllowedOriginPatterns(Arrays.asList(
    "https://*.example.com"
));

// ❌ BAD - Wildcard in production
configuration.setAllowedOrigins(Arrays.asList("*"));
```

### Minimal Permissions
```java
// Only allow necessary methods
configuration.setAllowedMethods(Arrays.asList(
    "GET",
    "POST",
    "PUT",
    "DELETE",
    "OPTIONS"
));

// Only expose necessary headers
configuration.setExposedHeaders(Arrays.asList(
    "Authorization",
    "Set-Cookie"
));
```

## Related Documentation
- See [authentication.md](./authentication.md) for JWT cookie setup
- See [authorization.md](./authorization.md) for access control
- See [profiles.md](../configuration/profiles.md) for environment configuration
