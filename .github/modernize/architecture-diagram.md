# Movie Info Service - Architecture Diagram

## System Architecture Overview

This document provides visual representations of the Movie Info Service architecture using Mermaid diagrams.

## High-Level System Architecture

```mermaid
graph TB
    subgraph "Client Applications"
        WebClient[Web Browser]
        APIClient[API Client]
    end

    subgraph "Movie Info Service"
        Controller[MovieInfoController<br/>REST Endpoints]
        Service[MovieInfoService<br/>Business Logic]
        Factory[ClientFactory<br/>API Client Factory]
        
        subgraph "API Clients"
            OmdbClient[OmdbClient]
            MoviedbClient[MoviedbClient]
        end
        
        Helper[WebClientHelper<br/>HTTP Utility]
        
        subgraph "Data Models"
            Movie[Movie Domain Model]
            OmdbDTO[OMDB DTOs]
            MoviedbDTO[MovieDB DTOs]
            Converter[Data Converters]
        end
    end

    subgraph "External Dependencies"
        Redis[(Redis Cache)]
        OmdbAPI[OMDB API<br/>omdbapi.com]
        MovieDBAPI[TheMovieDB API<br/>themoviedb.org]
    end

    WebClient --> Controller
    APIClient --> Controller
    
    Controller --> Service
    Service --> Factory
    Factory --> OmdbClient
    Factory --> MoviedbClient
    
    OmdbClient --> Helper
    MoviedbClient --> Helper
    
    Helper --> OmdbAPI
    Helper --> MovieDBAPI
    
    OmdbClient --> Converter
    MoviedbClient --> Converter
    Converter --> Movie
    
    Service -.-> Redis
    OmdbClient -.-> Redis
    MoviedbClient -.-> Redis

    style Controller fill:#90EE90
    style Service fill:#87CEEB
    style Factory fill:#DDA0DD
    style OmdbClient fill:#FFB6C1
    style MoviedbClient fill:#FFB6C1
    style Redis fill:#FFE4B5
    style OmdbAPI fill:#F0E68C
    style MovieDBAPI fill:#F0E68C
```

## Request Flow - Synchronous Mode

```mermaid
sequenceDiagram
    participant Client
    participant Controller as MovieInfoController
    participant Service as MovieInfoService
    participant Factory as ClientFactory
    participant APIClient as IMovieClient
    participant WebHelper as WebClientHelper
    participant ExtAPI as External API
    participant Redis

    Client->>Controller: GET /movies/synchron/{api}?title={title}
    activate Controller
    
    Controller->>Service: getMovieList(title, api)
    activate Service
    
    Service->>Factory: get(apiName)
    activate Factory
    Factory-->>Service: IMovieClient instance
    deactivate Factory
    
    Service->>APIClient: getMovieList(title)
    activate APIClient
    
    APIClient->>APIClient: getMovieFlux(title)
    APIClient->>WebHelper: webGet() - search
    activate WebHelper
    
    WebHelper->>Redis: Check cache
    activate Redis
    Redis-->>WebHelper: Cache miss
    deactivate Redis
    
    WebHelper->>ExtAPI: HTTP GET request
    activate ExtAPI
    ExtAPI-->>WebHelper: Search results
    deactivate ExtAPI
    
    WebHelper->>Redis: Cache result
    activate Redis
    deactivate Redis
    
    WebHelper-->>APIClient: SearchResult
    deactivate WebHelper
    
    loop For each movie in search
        APIClient->>WebHelper: webGet() - details
        activate WebHelper
        WebHelper->>ExtAPI: Get movie details
        activate ExtAPI
        ExtAPI-->>WebHelper: Movie details
        deactivate ExtAPI
        WebHelper-->>APIClient: DetailedMovie
        deactivate WebHelper
        
        APIClient->>APIClient: Convert to Movie
    end
    
    APIClient->>APIClient: collectList().block()
    APIClient-->>Service: List<Movie>
    deactivate APIClient
    
    Service-->>Controller: Map<String, List<Movie>>
    deactivate Service
    
    Controller-->>Client: JSON Response
    deactivate Controller
```

## Request Flow - Reactive (Flux) Mode

```mermaid
sequenceDiagram
    participant Client
    participant Controller as MovieInfoController
    participant Service as MovieInfoService
    participant Factory as ClientFactory
    participant APIClient as IMovieClient
    participant WebHelper as WebClientHelper
    participant ExtAPI as External API

    Client->>Controller: GET /movies/flux/{api}?title={title}
    activate Controller
    
    Controller->>Service: getMovieFlux(title, api)
    activate Service
    
    Service->>Factory: get(apiName)
    activate Factory
    Factory-->>Service: IMovieClient instance
    deactivate Factory
    
    Service->>APIClient: getMovieFlux(title)
    activate APIClient
    
    APIClient->>WebHelper: webGet() - search (Mono)
    activate WebHelper
    WebHelper->>ExtAPI: HTTP GET
    ExtAPI-->>WebHelper: SearchResult
    WebHelper-->>APIClient: Mono<SearchResult>
    deactivate WebHelper
    
    APIClient->>APIClient: flatMapMany - convert to Flux
    
    loop Stream each movie (non-blocking)
        APIClient->>WebHelper: webGet() - details (Mono)
        activate WebHelper
        WebHelper->>ExtAPI: HTTP GET
        ExtAPI-->>WebHelper: DetailedMovie
        WebHelper-->>APIClient: Mono<DetailedMovie>
        deactivate WebHelper
        
        APIClient->>APIClient: map to Movie
        APIClient-->>Service: Movie (streamed)
        Service-->>Controller: Movie (streamed)
        Controller-->>Client: Movie (SSE stream)
    end
    
    deactivate APIClient
    deactivate Service
    deactivate Controller
```

## Component Architecture

```mermaid
graph LR
    subgraph "Presentation Layer"
        MC[MovieInfoController]
    end
    
    subgraph "Business Logic Layer"
        MS[MovieInfoService]
        CF[ClientFactory]
    end
    
    subgraph "Integration Layer"
        IMC[IMovieClient Interface]
        OC[OmdbClient]
        TM[MoviedbClient]
        WCH[WebClientHelper]
    end
    
    subgraph "Data Layer"
        MOD[Movie Model]
        OMDB[OMDB DTOs]
        TMDB[TheMovieDB DTOs]
        CONV[Converters]
        SER[Serializers]
    end
    
    MC --> MS
    MS --> CF
    CF --> IMC
    IMC --> OC
    IMC --> TM
    OC --> WCH
    TM --> WCH
    OC --> CONV
    TM --> CONV
    CONV --> MOD
    CONV --> OMDB
    CONV --> TMDB
    MOD --> SER

    style MC fill:#90EE90
    style MS fill:#87CEEB
    style CF fill:#DDA0DD
    style OC fill:#FFB6C1
    style TM fill:#FFB6C1
    style MOD fill:#F0E68C
```

## Technology Stack Diagram

```mermaid
graph TB
    subgraph "Runtime Environment"
        JRE[Java 8 Runtime]
        Docker[Docker Container]
    end
    
    subgraph "Framework"
        SB[Spring Boot 2.3.2]
        SW[Spring WebFlux]
        SDR[Spring Data Redis]
        R[Reactor Core]
    end
    
    subgraph "Build & Dependencies"
        MVN[Maven]
        LMK[Lombok]
        WC[WebClient]
    end
    
    subgraph "External Services"
        Redis[(Redis)]
        OMDB[OMDB API]
        TMDB[TheMovieDB API]
    end
    
    Docker --> JRE
    JRE --> SB
    SB --> SW
    SB --> SDR
    SW --> R
    SW --> WC
    
    MVN --> SB
    LMK --> SB
    
    SDR --> Redis
    WC --> OMDB
    WC --> TMDB

    style JRE fill:#FF6B6B
    style SB fill:#4ECDC4
    style SW fill:#45B7D1
    style MVN fill:#96CEB4
```

## Deployment Architecture

```mermaid
graph TB
    subgraph "Container Registry"
        DockerHub[Docker Hub<br/>borkutip/movie-info]
    end
    
    subgraph "Build Pipeline"
        SRC[Source Code]
        MVN[Maven Build]
        PKG[Package JAR]
        IMG[Docker Build]
    end
    
    subgraph "Runtime Environment"
        Container[Docker Container<br/>Port 8080]
        App[Movie Info Service<br/>Spring Boot App]
    end
    
    subgraph "External Dependencies"
        Redis[(Redis Server)]
        OMDB[OMDB API]
        TMDB[TheMovieDB API]
    end
    
    subgraph "Clients"
        Browser[Web Browser]
        API[API Clients]
    end
    
    SRC --> MVN
    MVN --> PKG
    PKG --> IMG
    IMG --> DockerHub
    DockerHub --> Container
    Container --> App
    
    App --> Redis
    App --> OMDB
    App --> TMDB
    
    Browser --> Container
    API --> Container

    style Container fill:#90EE90
    style App fill:#87CEEB
    style Redis fill:#FFE4B5
    style DockerHub fill:#2496ED
```

## Data Flow - Movie Search

```mermaid
flowchart TD
    Start([Client Request]) --> Validate{Validate API<br/>Parameter}
    
    Validate -->|Valid| GetClient[Get API Client<br/>from Factory]
    Validate -->|Invalid| Error1[Throw Exception:<br/>Illegal API name]
    
    GetClient --> Search[Search Movies<br/>by Title]
    Search --> CheckCache{Check Redis<br/>Cache}
    
    CheckCache -->|Hit| ReturnCached[Return Cached<br/>Search Results]
    CheckCache -->|Miss| CallAPI[Call External<br/>API Search]
    
    CallAPI --> CacheSearch[Cache Search<br/>Results]
    CacheSearch --> ParseResults[Parse Search<br/>Results]
    ReturnCached --> ParseResults
    
    ParseResults --> Loop{For Each<br/>Movie}
    
    Loop -->|Next Movie| GetDetails[Get Movie<br/>Details by ID]
    GetDetails --> CheckCache2{Check Redis<br/>Cache}
    
    CheckCache2 -->|Hit| ReturnCached2[Return Cached<br/>Details]
    CheckCache2 -->|Miss| CallAPI2[Call External<br/>API Details]
    
    CallAPI2 --> CacheDetails[Cache Movie<br/>Details]
    CacheDetails --> Convert[Convert to<br/>Movie Model]
    ReturnCached2 --> Convert
    
    Convert --> Aggregate[Aggregate Movie]
    Aggregate --> Loop
    
    Loop -->|Complete| Return[Return Movie<br/>List/Flux]
    Return --> End([Response to Client])
    
    Error1 --> End

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style GetClient fill:#87CEEB
    style CheckCache fill:#FFE4B5
    style CheckCache2 fill:#FFE4B5
    style Convert fill:#DDA0DD
```

## Class Relationship Diagram

```mermaid
classDiagram
    class MovieInfoController {
        -MovieInfoService service
        +getMovieList(api, title) Map
        +getMovieFlux(api, title) Flux
    }
    
    class MovieInfoService {
        -ClientFactory clientFactory
        +getMovieList(movieTitle, apiName) Map
        +getMovieFlux(movieTitle, apiName) Flux
    }
    
    class ClientFactory {
        -List~IMovieClient~ clients
        -Map~String,IMovieClient~ clientCache
        +initClientCache() void
        +get(apiName) IMovieClient
    }
    
    class IMovieClient {
        <<interface>>
        +getApiName() String
        +getMovieList(searchString) List
        +getMovieFlux(searchString) Flux
    }
    
    class OmdbClient {
        -String BASE_URL
        -String API_KEY
        -WebClient client
        -WebClientHelper helper
        +getApiName() String
        +getMovieList(searchString) List
        +getMovieFlux(searchString) Flux
        -getPage(searchString) Mono
        -getMovieDetails(movie) Mono
    }
    
    class MoviedbClient {
        -String BASE_URL
        -String API_KEY
        -WebClient client
        -WebClientHelper helper
        +getApiName() String
        +getMovieList(searchString) List
        +getMovieFlux(searchString) Flux
    }
    
    class WebClientHelper {
        +webGet(client, url, params, type, fallback) Mono
    }
    
    class Movie {
        -String title
        -String year
        -String director
        -String poster
    }
    
    class Converter {
        +toMovie(detailedMovie, movie) Movie
    }
    
    MovieInfoController --> MovieInfoService
    MovieInfoService --> ClientFactory
    ClientFactory --> IMovieClient
    IMovieClient <|.. OmdbClient
    IMovieClient <|.. MoviedbClient
    OmdbClient --> WebClientHelper
    MoviedbClient --> WebClientHelper
    OmdbClient --> Converter
    MoviedbClient --> Converter
    Converter --> Movie
```

## Future State Architecture (Cloud Native)

```mermaid
graph TB
    subgraph "Edge Layer"
        LB[Load Balancer /<br/>API Gateway]
        CDN[CDN for Static Assets]
    end
    
    subgraph "Application Layer"
        Container1[Movie Info Service<br/>Instance 1]
        Container2[Movie Info Service<br/>Instance 2]
        Container3[Movie Info Service<br/>Instance 3]
    end
    
    subgraph "Caching Layer"
        RedisCluster[(Redis Cluster<br/>Managed Service)]
    end
    
    subgraph "Observability"
        Metrics[Metrics<br/>Prometheus/Azure Monitor]
        Logs[Centralized Logging<br/>ELK/Azure Logs]
        Tracing[Distributed Tracing<br/>Zipkin/OpenTelemetry]
    end
    
    subgraph "Security"
        Secrets[Secret Management<br/>Key Vault/Secrets Manager]
        Auth[Authentication<br/>OAuth2/JWT]
    end
    
    subgraph "External APIs"
        OMDB[OMDB API]
        TMDB[TheMovieDB API]
    end
    
    Client[Clients] --> LB
    LB --> Container1
    LB --> Container2
    LB --> Container3
    
    Container1 --> RedisCluster
    Container2 --> RedisCluster
    Container3 --> RedisCluster
    
    Container1 --> OMDB
    Container2 --> OMDB
    Container3 --> OMDB
    
    Container1 --> TMDB
    Container2 --> TMDB
    Container3 --> TMDB
    
    Container1 -.-> Metrics
    Container1 -.-> Logs
    Container1 -.-> Tracing
    
    Container2 -.-> Metrics
    Container2 -.-> Logs
    Container2 -.-> Tracing
    
    Container3 -.-> Metrics
    Container3 -.-> Logs
    Container3 -.-> Tracing
    
    Container1 -.-> Secrets
    Container2 -.-> Secrets
    Container3 -.-> Secrets
    
    Container1 -.-> Auth
    Container2 -.-> Auth
    Container3 -.-> Auth

    style LB fill:#90EE90
    style Container1 fill:#87CEEB
    style Container2 fill:#87CEEB
    style Container3 fill:#87CEEB
    style RedisCluster fill:#FFE4B5
    style Metrics fill:#FFB6C1
    style Logs fill:#FFB6C1
    style Tracing fill:#FFB6C1
    style Secrets fill:#DDA0DD
```

## Notes

All diagrams are created using Mermaid syntax and can be rendered by:
- GitHub (native support)
- Mermaid Live Editor (https://mermaid.live)
- VS Code with Mermaid extension
- Documentation tools that support Mermaid

### Diagram Descriptions

1. **High-Level System Architecture**: Shows the overall system components and their relationships
2. **Request Flow - Synchronous Mode**: Sequence diagram for blocking API calls
3. **Request Flow - Reactive (Flux) Mode**: Sequence diagram for non-blocking streaming responses
4. **Component Architecture**: Layered architecture with component dependencies
5. **Technology Stack Diagram**: Technology dependencies and relationships
6. **Deployment Architecture**: Current deployment setup with Docker
7. **Data Flow - Movie Search**: Flowchart showing the movie search process with caching
8. **Class Relationship Diagram**: UML-style class diagram showing relationships
9. **Future State Architecture**: Proposed cloud-native architecture with scalability and observability

### Key Architectural Patterns

- **Layered Architecture**: Clear separation between presentation, business, and data layers
- **Factory Pattern**: ClientFactory for creating API clients
- **Strategy Pattern**: IMovieClient interface with multiple implementations
- **Reactive Programming**: Non-blocking I/O with Project Reactor
- **Repository Pattern**: (Implied) for data access through Redis
- **Adapter Pattern**: Converters to adapt external API models to internal domain models
