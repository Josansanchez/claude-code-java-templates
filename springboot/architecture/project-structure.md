# Estructura del Proyecto

## Organización por Recursos

**IMPORTANTE**: Cada capa debe organizarse por recursos de base de datos en directorios separados.

**TODOS LOS DIRECTORIOS EN MINÚSCULAS**

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
│   │   └── UserRepository.java
│   ├── product/
│   │   └── ProductRepository.java
│   └── user/
│       └── UserRepository.java
├── entities/
│   ├── User.java
│   ├── Product.java
│   └── User.java
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

## Recursos

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

## Reglas de Organización

1. **Directorios siempre en minúsculas**: user, product, auth, mqtt
2. **Un recurso = Un directorio** en cada capa
3. **Funcionalidades especiales** (auth, mqtt) tienen su propio directorio
4. **Mappers separados** en paquete propio
5. **Config centralizada** en paquete config
6. **Excepciones globales** en paquete exceptions
