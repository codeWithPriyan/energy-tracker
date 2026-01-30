# Home Energy Tracker

A sophisticated microservices-based home energy monitoring system that tracks energy consumption from IoT devices, provides real-time monitoring, intelligent insights, and automated alerting when energy thresholds are exceeded.

## 🏗️ Architecture Overview

This system demonstrates modern microservices architecture with event-driven design, real-time data processing, AI integration, and comprehensive energy monitoring capabilities.

### Technology Stack

- **Backend**: Spring Boot 4.0/3.5.9 with Java 21
- **Databases**: MySQL 8.3.0 (relational), InfluxDB 2.7 (time-series)
- **Event Streaming**: Apache Kafka with Kafka UI
- **AI Integration**: Ollama with DeepSeek-Coder model
- **Containerization**: Docker & Docker Compose
- **Email Service**: Mailpit for testing
- **Build Tool**: Maven
- **Version Control**: Git

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Java 21 (for local development)
- Maven 3.8+
- Git

## 📥 Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/home-energy-tracker.git
cd home-energy-tracker
```

### 2. Project Structure Overview

```bash
home-energy-tracker/
├── user-service/          # User management microservice
├── device-service/        # Device registry microservice
├── ingestion-service/     # Data ingestion microservice
├── usage-service/         # Usage processing microservice
├── alert-service/         # Alert notification microservice
├── insight-service/       # AI insights microservice
├── docker/               # Database initialization scripts
├── docker-compose.yml    # Infrastructure services
└── README.md            # This file
```

### 3. Environment Setup

```bash
# Verify Java version
java -version  # Should show Java 21

# Verify Maven version
mvn -version   # Should show Maven 3.8+

# Verify Docker
docker --version
docker-compose --version
```

### 4. Running the System

#### Option A: Development Mode (Recommended for contributors)

1. **Start infrastructure services**:

```bash
docker compose -v up -d
```

2. **Start individual microservices** (in separate terminals):

```bash
# User Service
cd user-service && ./mvnw spring-boot:run

# Device Service
cd device-service && ./mvnw spring-boot:run

# Ingestion Service
cd ingestion-service && ./mvnw spring-boot:run

# Usage Service
cd usage-service && ./mvnw spring-boot:run

# Alert Service
cd alert-service && ./mvnw spring-boot:run

# Insight Service
cd insight-service && ./mvnw spring-boot:run
```

#### Option B: Quick Test (Docker only)

```bash
# Build and run all services with Docker
docker compose -f docker-compose.yml -f docker-compose.services.yml up -d
```

### 5. Stop Services

```bash
# Stop all services
docker compose down

# Stop services and remove volumes (clean start)
docker compose down -v
```

### Troubleshooting

If encountering database issues, delete pre-existing volumes:

```bash
docker compose down -v
docker system prune -f
```

## 📋 Services Overview

### 1. User Service (Port 8080)

**Purpose**: Manages user profiles and alert preferences

**Features**:

- User CRUD operations
- Alert threshold configuration
- Email preferences management
- AOP-based logging and performance monitoring

**API Endpoints**:

- `POST /api/v1/user` - Create user
- `GET /api/v1/user/{id}` - Get user by ID
- `PUT /api/v1/user/{id}` - Update user
- `DELETE /api/v1/user/{id}` - Delete user

### 2. Device Service (Port 8081)

**Purpose**: Manages IoT devices and their associations with users

**Features**:

- Device registry management
- Device type categorization (SPEAKER, CAMERA, THERMOSTAT, LIGHT, LOCK, DOORBELL)
- User-device associations
- Comprehensive exception handling

**API Endpoints**:

- `GET /api/v1/device/{id}` - Get device by ID
- `POST /api/v1/device/create` - Create device
- `PUT /api/v1/device/{id}` - Update device
- `DELETE /api/v1/device/{id}` - Delete device
- `GET /api/v1/device/user/{userId}` - Get all devices for user

### 3. Ingestion Service (Port 8082)

**Purpose**: Receives energy usage data and publishes to Kafka

**Features**:

- High-performance data ingestion
- Parallel data simulation (configurable threads)
- Kafka event publishing
- Real-time data processing

**API Endpoints**:

- `POST /api/v1/ingestion` - Ingest energy usage data

**Data Simulation**:

- Configurable requests per interval (default: 10,000)
- Parallel processing with adjustable threads
- Mock IoT device data generation

### 4. Usage Service (Port 8083)

**Purpose**: Processes energy usage data, stores in InfluxDB, and monitors thresholds

**Features**:

- Kafka consumer for energy events
- Time-series data storage in InfluxDB
- Scheduled aggregation (every 10 seconds)
- Threshold monitoring and alert generation
- Inter-service communication

**API Endpoints**:

- `GET /api/v1/usage/{userId}?days=N` - Get N days of usage for user

**Processing Logic**:

1. Consumes energy usage events from Kafka
2. Stores time-series data in InfluxDB
3. Aggregates hourly usage per user
4. Monitors thresholds and triggers alerts

### 5. Alert Service (Port 8084)

**Purpose**: Handles alert notifications via email

**Features**:

- Kafka consumer for alert events
- Email notification system
- Alert history tracking
- Mailpit integration for testing

**Database Schema**:

- Tracks sent alerts with timestamps
- User association for alert history

### 6. Insight Service (Port 8085)

**Purpose**: Provides AI-powered energy efficiency insights

**Features**:

- AI integration with Ollama (DeepSeek-Coder model)
- Energy saving recommendations
- Usage pattern analysis
- Comparative analysis with average households

**API Endpoints**:

- `GET /api/v1/insight/saving-tips/{userId}` - Get AI-powered saving tips
- `GET /api/v1/insight/overview/{userId}` - Get usage overview with insights

## 🔄 Data Flow Architecture

### Energy Data Flow

1. **Ingestion**: IoT devices send energy data to ingestion service
2. **Kafka Publishing**: Data published as `EnergyUsageEvent` to `energy-usage` topic
3. **Processing**: Usage service consumes events, stores in InfluxDB
4. **Aggregation**: Every 10 seconds, hourly usage is aggregated per user
5. **Threshold Check**: If usage > threshold, `AlertingEvent` published to `energy-alerts`
6. **Alerting**: Alert service consumes events, sends email notifications

### Kafka Events

```java
EnergyUsageEvent: {
    deviceId: Long,
    energyConsumed: double,
    timestamp: Instant
}

AlertingEvent: {
    userId: Long,
    message: String,
    threshold: double,
    energyConsumed: double,
    email: String
}
```

## 🗄️ Database Schema

### MySQL Tables

```sql
user: id, name, surname, email, address, alerting, energy_alerting_threshold
device: id, name, type, location, user_id (FK)
alert: id, user_id, sent, created_at
```

### InfluxDB Structure

- **Measurement**: `energy_usage`
- **Tags**: `deviceId`
- **Fields**: `energyConsumed`
- **Time**: Timestamp of energy usage

## 🔧 Configuration

### Infrastructure Services

- **MySQL**: Port 3306, Database: `home_energy_tracker`
- **Kafka**: Ports 9092 (internal), 9094 (external)
- **Kafka UI**: Port 8070 for management
- **InfluxDB**: Port 8072, Org: `leetjourney`, Bucket: `usage-bucket`
- **Mailpit**: Ports 8025 (web), 1025 (SMTP)

### Service Ports

- User Service: 8080
- Device Service: 8081
- Ingestion Service: 8082
- Usage Service: 8083
- Alert Service: 8084
- Insight Service: 8085

## 🧪 Testing

### Running Tests

```bash
# Run tests for all services
find . -name "pom.xml" -execdir mvnw test \;

# Run tests for specific service
cd user-service && ./mvnw test
```

### Test Coverage

- Unit tests for all service layers
- Integration tests for repositories
- Spring Boot test slices for controllers
- Kafka test infrastructure

## 📊 Monitoring & Observability

### Available Tools

- **Kafka UI**: http://localhost:8070 - Message monitoring
- **Mailpit**: http://localhost:8025 - Email testing
- **InfluxDB**: http://localhost:8072 - Time-series data

### Logging

- Structured logging across all services
- AOP-based performance monitoring
- Request/response logging

## 🚀 Performance Features

### High-Throughput Processing

- Parallel data ingestion (configurable threads)
- 10,000+ requests per interval capability
- Efficient time-series data handling
- Event-driven architecture for scalability

### Optimization Techniques

- Connection pooling for databases
- Kafka consumer group management
- Scheduled batch processing
- Caching strategies

## 🤖 AI Integration

### Ollama Configuration

- Model: DeepSeek-Coder
- Purpose: Energy efficiency analysis
- Features:
  - Personalized saving tips
  - Usage pattern recognition
  - Comparative analysis

## 🔒 Security Considerations

### Current Implementation

- Input validation in all endpoints
- Exception handling for error responses
- Database connection security
- Kafka authentication (development setup)

### Production Recommendations

- API gateway implementation
- OAuth 2.0/JWT authentication
- Network policies
- Secrets management

## 📈 Scalability

### Horizontal Scaling

- Stateless service design
- Kafka partitioning support
- Database connection pooling
- Container orchestration ready

### Performance Tuning

- Configurable thread pools
- Batch processing optimization
- Memory management
- Connection timeout configuration

## 🛠️ Development

### Local Development Setup

1. Clone repository
2. Start infrastructure with Docker Compose
3. Run services individually or use IDE
4. Access services via respective ports

### Code Structure

```
├── user-service/          # User management
├── device-service/        # Device registry
├── ingestion-service/     # Data ingestion
├── usage-service/         # Usage processing
├── alert-service/         # Alert notifications
├── insight-service/       # AI insights
├── docker/               # Database init scripts
└── docker-compose.yml    # Infrastructure setup
```

## 📝 Project Highlights

### Advanced Backend Concepts

- **Microservices Architecture**: 6 specialized services
- **Event-Driven Design**: Kafka-based messaging
- **Time-Series Data**: InfluxDB for analytics
- **AI Integration**: Ollama for intelligent insights
- **Real-Time Processing**: Live energy monitoring
- **Automated Alerting**: Threshold-based notifications

### Production-Ready Features

- **Containerization**: Docker deployment
- **Database Migrations**: Flyway integration
- **Comprehensive Testing**: Unit and integration tests
- **Monitoring**: Logging and performance tracking
- **Error Handling**: Global exception management
- **Configuration Management**: Externalized configuration

## 🤝 Contributing

We welcome contributions to the Home Energy Tracker project! This guide will help you get started.

### 🎯 How to Contribute

#### 1. Fork and Clone

```bash
# Fork the repository on GitHub first, then clone your fork
git clone https://github.com/YOUR-USERNAME/home-energy-tracker.git
cd home-energy-tracker

# Add the original repository as upstream
git remote add upstream https://github.com/original-owner/home-energy-tracker.git
```

#### 2. Set Up Development Environment

```bash
# Create a new branch for your feature
git checkout -b feature/your-feature-name

# Follow the installation steps above
# Install dependencies and start services
```

#### 3. Make Changes

- Follow the existing code style and patterns
- Add tests for new functionality
- Update documentation as needed
- Ensure all tests pass

#### 4. Test Your Changes

```bash
# Run tests for all services
find . -name "pom.xml" -execdir ./mvnw test \;

# Run tests for specific service
cd user-service && ./mvnw test

# Build all services
find . -name "pom.xml" -execdir ./mvnw clean compile \;
```

#### 5. Commit and Push

```bash
# Stage your changes
git add .

# Commit with a descriptive message
git commit -m "feat: add new energy monitoring feature"

# Push to your fork
git push origin feature/your-feature-name
```

#### 6. Create Pull Request

- Go to your fork on GitHub
- Click "New Pull Request"
- Provide a clear description of your changes
- Link any relevant issues
- Wait for code review

### 📝 Development Guidelines

#### Code Style

- Follow Spring Boot best practices
- Use meaningful variable and method names
- Keep methods small and focused
- Add Javadoc comments for public APIs

#### Testing Requirements

- **Unit Tests**: Test all service layer methods
- **Integration Tests**: Test database operations and external service calls
- **API Tests**: Test all REST endpoints
- **Coverage**: Maintain minimum 80% test coverage

#### Code Structure

```
src/main/java/com/leetjourney/service_name/
├── controller/     # REST controllers
├── service/        # Business logic
├── repository/     # Data access layer
├── entity/         # JPA entities
├── dto/           # Data transfer objects
├── config/        # Configuration classes
└── exception/     # Custom exceptions
```

### 🐛 Bug Reports

#### How to Report a Bug

1. **Check existing issues** first
2. **Use the bug report template**:
   - Title: Clear description of the bug
   - Environment: OS, Java version, service versions
   - Steps to reproduce
   - Expected vs actual behavior
   - Logs/error messages
   - Screenshots if applicable

#### Bug Fix Process

1. Create issue with detailed description
2. Assign bug to yourself (if you want to fix it)
3. Create branch: `bugfix/issue-number-description`
4. Implement fix with tests
5. Submit pull request

### ✨ Feature Requests

#### Requesting New Features

1. **Check existing issues** and roadmap
2. **Create feature request** with:
   - Clear description of the feature
   - Use case and benefits
   - Proposed implementation approach
   - Acceptance criteria

#### Implementing Features

1. Discuss implementation in issue comments
2. Create branch: `feature/feature-name`
3. Follow development guidelines
4. Include comprehensive tests
5. Update documentation

### 🔧 Development Tools

#### IDE Configuration

- **IntelliJ IDEA**: Recommended for Spring Boot development
- **VS Code**: Works well with Java extensions
- **Eclipse**: Supported but less optimal

#### Useful Plugins

- Lombok plugin for annotation processing
- Spring Boot helper plugins
- Docker integration
- Git integration

#### Debugging

```bash
# Debug individual service
cd user-service && ./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"

# Debug with IDE
# Set remote debugging on port 5005
```

### 📋 Pull Request Template

```markdown
## Description

Brief description of changes made.

## Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Added new tests for new functionality

## Checklist

- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
```

### 🏷️ Release Process

#### Versioning

- Follow Semantic Versioning (SemVer)
- Major.Minor.Patch format
- Update version in all `pom.xml` files

#### Release Steps

1. Update version numbers
2. Update CHANGELOG.md
3. Create release branch
4. Tag release
5. Deploy to staging
6. Test thoroughly
7. Deploy to production
8. Create GitHub release

### 📚 Resources for Contributors

#### Documentation

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [InfluxDB Documentation](https://docs.influxdata.com/)
- [Docker Documentation](https://docs.docker.com/)

#### Learning Resources

- Microservices architecture patterns
- Event-driven design principles
- Time-series database concepts
- Docker containerization

### 🚀 Getting Help

#### Community Support

- **GitHub Issues**: For bugs and feature requests
- **Discussions**: For general questions and ideas
- **Wiki**: For detailed documentation

#### Contact Maintainers

- **Primary Maintainer**: [maintainer-email@example.com]
- **Backup Maintainer**: [backup-email@example.com]

### 🏆 Recognition

#### Contributor Recognition

- Contributors listed in README
- Featured in release notes
- Special recognition for significant contributions
- Invitation to core team for regular contributors

#### Types of Contributions

- Code contributions
- Documentation improvements
- Bug reports and testing
- Feature suggestions
- Community support

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

By contributing to this project, you agree that your contributions will be licensed under the same license.

## 📞 Support

For questions or support regarding this project:

- **Documentation**: Check this README and Wiki
- **Issues**: Create a GitHub issue
- **Discussions**: Start a GitHub discussion
- **Email**: support@home-energy-tracker.com
