# Auth Service - SuperFS

A Spring Boot microservice providing JWT-based authentication functionality for the SuperFS system. This service handles user registration, login, logout, and token validation.

## 🚀 Features

- User registration with email validation
- JWT-based authentication
- Secure HTTP-only cookie management
- RESTful API endpoints
- Microservice architecture ready
- Kubernetes/Minikube deployment support

## 📋 Prerequisites

### Development Environment Requirements

- **Java**: 17 or higher
- **Maven**: 3.6+ or Gradle
- **Spring Boot**: 3.x
- **Database**: MySQL/PostgreSQL (based on your configuration)
- **Docker**: For containerization
- **Kubernetes/Minikube**: For deployment

### Dependencies

- Spring Boot Starter Web
- Spring Boot Starter Data JPA
- Spring Boot Starter Security
- JWT Library
- Database Driver (MySQL/PostgreSQL)
- Thymeleaf (if using server-side rendering)

## 🛠️ Setup Instructions

### 1. Clone the Repository

```bash
git clone <repository-url>
cd auth-service-superfs
```

### 2. Configure Application Properties

Create `application.yml` or `application.properties`:

```
spring.application.name=auth-service-superfs
server.port=8010

eureka.instance.hostname=auth-service-superfs
eureka.instance.prefer-ip-address=false
eureka.instance.instance-id=${spring.application.name}:${server.port}


eureka.client.service-url.defaultZone=http://eureka-service-superfs:8761/eureka/


spring.config.import=optional:configserver:http://cloud-config-superfs:8888

spring.datasource.url=jdbc:mysql://mysql:3306/superfs
spring.datasource.username=root
spring.datasource.password=6969
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA settings
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect


springdoc.api-docs.path=/v3/api-docs
springdoc.swagger-ui.path=/swagger-ui.html

management.endpoints.web.exposure.include=health,info,prometheus

management.endpoint.health.show-details=always

```

### 3. Environment Variables

Set the following environment variables:

```bash
export DB_USERNAME=your_db_username
export DB_PASSWORD=your_db_password
export JWT_SECRET_KEY=your-super-secret-jwt-key
```

### 4. Build and Run

#### Using Maven:
```bash
# Build
mvn install -DskipTests 

```

The service will start on port `8010`.

## 📚 API Documentation

### Base URL
```
http://localhost:8010/auth
```

### Endpoints

#### 1. Test Endpoint
- **GET** `/test`
- **Description**: Health check and configuration test
- **Response**: String with service status and configuration

#### 2. User Registration
- **POST** `/signup`
- **Content-Type**: `application/json`
- **Request Body**:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```
- **Success Response** (200):
```json
{
  "status": "success"
}
```
- **Error Response** (400):
```json
{
  "status": "fail",
  "message": "Email already exists"
}
```

#### 3. User Login
- **POST** `/login`
- **Content-Type**: `application/json`
- **Request Body**:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```
- **Success Response**: Sets JWT cookie and returns user data
- **Error Response** (401): Authentication failed

#### 4. User Logout
- **POST** `/logout`
- **Description**: Clears JWT cookie
- **Response**: "Logout successful"

#### 5. Protected Test Route
- **GET** `/blah`
- **Description**: Test endpoint for authenticated users
- **Authentication**: Requires valid JWT cookie
- **Response**: "wow u successfully logged in"

## 🐳 Docker Deployment

### 1. Create Dockerfile

```dockerfile

FROM maven:3.9.0-eclipse-temurin-17-alpine AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]

```

### 2. Build Docker Image

```bash
# Build the application
mvn clean package

# Build Docker image
docker build -t auth-service-superfs:latest .

# Run container
docker run -p 8010:8010 \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=password \
  -e JWT_SECRET_KEY=your-secret-key \
  auth-service-superfs:latest
```

## ☸️ Kubernetes/Minikube Deployment

### 1. Start Minikube

```bash
minikube start
eval $(minikube docker-env)
```

### 2. Build Image in Minikube

```bash
docker build -t auth-service-superfs:latest .
```

### 3. Create Kubernetes Manifests

#### ConfigMap (auth-configmap.yaml)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: auth-service-config
data:
  DB_USERNAME: "root"
  JWT_SECRET_KEY: "your-super-secret-jwt-key"
```

#### Secret (auth-secret.yaml)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: auth-service-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ= # base64 encoded 'password'
```

#### Deployment (auth-deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: auth-service
  template:
    metadata:
      labels:
        app: auth-service
    spec:
      containers:
      - name: auth-service
        image: auth-service-superfs:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 8010
        envFrom:
        - configMapRef:
            name: auth-service-config
        - secretRef:
            name: auth-service-secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

#### Service (auth-service.yaml)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  selector:
    app: auth-service
  ports:
  - port: 8010
    targetPort: 8010
    nodePort: 30010
  type: NodePort
```

### 4. Deploy to Minikube

```bash
# Apply configurations
kubectl apply -f auth-configmap.yaml
kubectl apply -f auth-secret.yaml
kubectl apply -f auth-deployment.yaml
kubectl apply -f auth-service.yaml

# Check deployment status
kubectl get pods
kubectl get services

# Get service URL
minikube service auth-service --url
```

### 5. Deployment Commands Summary

```bash
# Build and deploy
mvn clean package
docker build -t auth-service-superfs:latest .
kubectl apply -f k8s/

# Scale deployment
kubectl scale deployment auth-service --replicas=3

# View logs
kubectl logs -f deployment/auth-service

# Delete deployment
kubectl delete -f k8s/
```

## 🧪 Testing

### Manual Testing

```bash
# Test signup
curl -X POST http://localhost:8010/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Test login
curl -X POST http://localhost:8010/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}' \
  -c cookies.txt

# Test protected endpoint
curl -X GET http://localhost:8010/auth/blah \
  -b cookies.txt
```

## 📁 Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/elu/authservicesuperfs/
│   │       ├── controller/
│   │       │   └── AuthController.java
│   │       ├── service/
│   │       │   └── AuthService.java
│   │       ├── repo/
│   │       │   └── UserRepo.java
│   │       ├── dto/
│   │       │   └── UserRequestDto.java
│   │       └── AuthServiceSuperfSApplication.java
│   └── resources/
│       ├── application.yml
│       └── templates/ (if using Thymeleaf)
├── test/
└── Dockerfile
k8s/
├── auth-configmap.yaml
├── auth-secret.yaml
├── auth-deployment.yaml
└── auth-service.yaml
```

## 🔐 Security Notes

- JWT secret key should be stored securely (use Kubernetes secrets in production)
- Enable HTTPS in production environments
- Configure proper CORS settings
- Implement rate limiting for authentication endpoints
- Use strong password policies
- Regular security audits recommended

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📞 Support

For questions or support, please contact the development team or create an issue in the repository.

---

**Service Port**: 8010  
**Environment**: Minikube  
**Architecture**: Microservice
