# Movie Info Service - Architecture Diagram

This diagram shows the high-level architecture of the Movie Info Service application based on the assessment analysis.

## Application Architecture

```mermaid
graph TB
    subgraph "External Clients"
        Client[HTTP Client/Browser]
    end

    subgraph "Movie Info Service - Spring Boot 2.3.2"
        subgraph "Presentation Layer"
            Controller[MovieInfoController<br/>REST API Endpoints]
        end
        
        subgraph "Business Layer"
            Service[MovieInfoService<br/>Business Logic]
            Factory[ClientFactory<br/>API Client Selection]
        end
        
        subgraph "Integration Layer"
            OmdbClient[OmdbClient<br/>OMDb API Integration]
            MoviedbClient[MoviedbClient<br/>TheMovieDB API Integration]
            WebClient[WebClientHelper<br/>Reactive HTTP Client]
        end
        
        subgraph "Data Layer"
            DataModel[Movie Data Models<br/>Domain Objects]
        end
    end

    subgraph "External Services"
        OmdbAPI[OMDb API<br/>www.omdbapi.com]
        MovieDBAPI[TheMovieDB API<br/>api.themoviedb.org]
    end

    subgraph "Data Storage"
        Redis[Redis Cache<br/>spring-boot-starter-data-redis]
    end

    Client -->|GET /movies/synchron/{api}| Controller
    Client -->|GET /movies/flux/{api}| Controller
    Controller -->|Delegates| Service
    Service -->|Gets Client| Factory
    Factory -->|Returns| OmdbClient
    Factory -->|Returns| MoviedbClient
    OmdbClient -->|Uses| WebClient
    MoviedbClient -->|Uses| WebClient
    OmdbClient -->|HTTP Requests| OmdbAPI
    MoviedbClient -->|HTTP Requests| MovieDBAPI
    Service -->|Creates| DataModel
    Service -->|Cache Operations| Redis
    OmdbClient -->|Converts to| DataModel
    MoviedbClient -->|Converts to| DataModel

    style Controller fill:#e1f5ff
    style Service fill:#e1f5ff
    style Factory fill:#e1f5ff
    style OmdbClient fill:#fff4e1
    style MoviedbClient fill:#fff4e1
    style WebClient fill:#fff4e1
    style DataModel fill:#f0f0f0
    style Redis fill:#ffe1e1
    style OmdbAPI fill:#e1ffe1
    style MovieDBAPI fill:#e1ffe1
```

## Technology Stack

### Core Framework
- **Spring Boot**: 2.3.2.RELEASE
- **Java Version**: 1.8 (Java 8)

### Key Dependencies
- **Spring WebFlux**: Reactive web framework for non-blocking HTTP
- **Spring Boot Starter Data Redis**: Redis integration for caching
- **Spring Boot Starter Tomcat**: Servlet container
- **Project Lombok**: Code generation for boilerplate reduction
- **Reactive Streams**: Reactor Core for reactive programming

### External Integrations
- **OMDb API**: Open Movie Database API (www.omdbapi.com)
- **TheMovieDB API**: The Movie Database API (api.themoviedb.org)

### Build & Deployment
- **Maven**: Build automation tool
- **Docker**: Containerization with openjdk:8-jdk-alpine base image
- **Dockerfile Maven Plugin**: Container image building (version 1.2.2)

## Application Flow

1. **Request Handling**: 
   - Client sends HTTP GET requests to REST endpoints
   - Supports both synchronous (`/synchron/{api}`) and reactive streaming (`/flux/{api}`) modes

2. **Business Processing**:
   - Controller delegates to Service layer
   - Service uses ClientFactory to select appropriate API client (OMDb or TheMovieDB)
   
3. **External API Integration**:
   - Selected client makes HTTP requests to external movie APIs
   - Uses WebFlux reactive HTTP client for non-blocking calls
   - Converts API responses to internal Movie data models

4. **Caching**:
   - Redis integration available via spring-boot-starter-data-redis
   - Provides caching layer for frequently accessed movie data

5. **Response**:
   - Synchronous mode: Returns complete movie list as JSON
   - Reactive mode: Streams movie data as Server-Sent Events (SSE)

## Architecture Patterns

- **Layered Architecture**: Clear separation between Presentation, Business, and Integration layers
- **Factory Pattern**: ClientFactory for dynamic API client selection
- **Reactive Programming**: Support for non-blocking reactive streams with Flux
- **REST API**: RESTful endpoint design with path variables and query parameters
- **External Service Integration**: Multiple third-party API integrations with abstraction layer

## Key Assessment Findings

Based on the AppCAT assessment report, the following modernization opportunities were identified:

1. **Java Version Upgrade**: Currently using Java 8 (EOL), should upgrade to Java 11/17/21
2. **Spring Boot Upgrade**: Version 2.3.2 is outdated, consider upgrading to latest 2.x or 3.x
3. **Security Concerns**: API keys hardcoded in application.properties
4. **Docker Base Image**: Using deprecated openjdk:8-jdk-alpine image
5. **Deprecated APIs**: Some Spring APIs may be deprecated in current version

## Deployment Targets

The application is assessed for deployment to:
- Azure Kubernetes Service (AKS)
- Azure App Service
- Azure Container Apps

All platforms support the Spring Boot architecture with containerization.
