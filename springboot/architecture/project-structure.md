# Project Structure

## Resource-Based Organization

**IMPORTANT**: Each layer must be organized by database resources in separate directories.

**ALL DIRECTORIES IN LOWERCASE**

```
src/main/java/com/example/myapp/
├── controllers/
│   ├── user/
│   │   ├── CreateUserController.java
│   │   ├── GetUserController.java
│   │   ├── UpdateUserController.java
│   │   └── DeleteUserController.java
│   ├── product/
│   │   ├── CreateProductController.java
│   │   ├── GetProductController.java
│   │   ├── UpdateProductController.java
│   │   └── DeleteProductController.java
│   ├── order/
│   │   ├── CreateOrderController.java
│   │   └── GetOrderController.java
│   └── auth/
│       ├── LoginController.java
│       ├── SignupController.java
│       └── SignoutController.java
├── services/
│   ├── user/
│   │   ├── CreateUserService.java
│   │   ├── GetUserService.java
│   │   ├── UpdateUserService.java
│   │   └── DeleteUserService.java
│   ├── product/
│   │   ├── CreateProductService.java
│   │   └── GetProductService.java
│   ├── auth/
│   │   ├── LoginService.java
│   │   └── SignupService.java
│   └── mqtt/
│       ├── ConnectMqttService.java
│       └── PublishMqttService.java
├── repositories/
│   ├── user/
│   │   ├── UserRepository.java
│   │   └── UserSpecifications.java
│   ├── product/
│   │   ├── ProductRepository.java
│   │   └── ProductSpecifications.java
│   └── order/
│       ├── OrderRepository.java
│       └── OrderSpecifications.java
├── entities/
│   ├── User.java
│   ├── Product.java
│   └── Order.java
├── dto/
│   ├── UserDTO.java
│   └── ProductDTO.java
├── mappers/
│   ├── UserMapper.java
│   └── ProductMapper.java
├── exceptions/
│   ├── ResourceNotFoundException.java
│   ├── ResourceAlreadyExistsException.java
│   └── GlobalExceptionHandler.java
├── config/
│   ├── OpenApiConfig.java
│   ├── WebSecurityConfig.java
│   └── ModelMapperConfig.java
├── security/
│   └── jwt/
│       ├── JwtUtils.java
│       └── AuthTokenFilter.java
├── constants/
│   ├── Constants.java
│   └── RolesConstants.java
└── utils/
    └── Utils.java
```

## Tests

```
src/test/java/com/example/myapp/
├── user/
│   ├── controller/
│   │   └── CreateUserControllerTest.java
│   ├── service/
│   │   └── CreateUserServiceTest.java
│   └── repository/
│       └── UserRepositoryTest.java
├── integration/
│   └── UserIntegrationTest.java
└── models/
    └── UserTest.java
```

## Resources

```
src/main/resources/
├── application.yml
├── application-dev.yml
├── application-test.yml
├── application-prod.yml
└── db/
    └── migration/
        └── V1__initial_schema.sql
```

## Organization Rules

1. **Directories always in lowercase**: user, product, auth, mqtt
2. **One resource = One directory** in each layer
3. **Special features** (auth, mqtt) have their own directory
4. **Separate mappers** in their own package
5. **Centralized config** in config package
6. **Global exceptions** in exceptions package
7. **Specifications with repositories** in same directory as repository
