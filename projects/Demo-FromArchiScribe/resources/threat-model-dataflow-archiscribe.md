# ArchiMate View name: dataflow-view

## Elements

### Internet
- Type: Grouping
- Properties:
  - Stereotype: Trust  Boundary

### Mobile App
- Type: ApplicationComponent

### Cloud Host
- Type: Grouping
- Properties:
  - Stereotype: Trust  Boundary

### STS
- Type: ApplicationComponent
- Documentation: Secure Token Service

### API Gateway
- Type: ApplicationComponent

### App Service
- Type: ApplicationComponent

### Database
- Type: SystemSoftware

### 3rd Party Host
- Type: Grouping
- Properties:
  - Stereotype: Trust  Boundary

### AI Agent Service
- Type: ApplicationComponent
- Documentation: LLM based AI service

## Relationships

- From **Mobile App** to **STS**
  - Type: Flow
  - Name: 1: Auth Request
  - Properties:
    - Protocol: HTTPS

- From **Mobile App** to **API Gateway**
  - Type: Flow
  - Name: 3: Request
  - Properties:
    - Protocol: HTTPS

- From **STS** to **Mobile App**
  - Type: Flow
  - Name: 2: Auth Token
  - Properties:
    - Protocol: HTTPS

- From **API Gateway** to **App Service**
  - Type: Flow
  - Name: 4: Request
  - Properties:
    - Protocol: HTTP

- From **API Gateway** to **Mobile App**
  - Type: Flow
  - Name: 10: Response
  - Properties:
    - Payload: JSON

- From **App Service** to **Database**
  - Type: Flow
  - Name: 5: SQL Query
  - Documentation: SQL query via an ORM.

- From **App Service** to **AI Agent Service**
  - Type: Flow
  - Name: 7: Request
  - Properties:
    - Protocol: HTTPS

- From **App Service** to **API Gateway**
  - Type: Flow
  - Name: 9: Response
  - Properties:
    - Payload: JSON

- From **Database** to **App Service**
  - Type: Flow
  - Name: 6: Result Set

- From **AI Agent Service** to **App Service**
  - Type: Flow
  - Name: 8: Response
  - Properties:
    - Payload: Markdown

- From **Internet** to **Mobile App**
  - Type: Aggregation (implicit from view nesting)

- From **Cloud Host** to **STS**
  - Type: Aggregation (implicit from view nesting)

- From **Cloud Host** to **API Gateway**
  - Type: Aggregation (implicit from view nesting)

- From **Cloud Host** to **App Service**
  - Type: Aggregation (implicit from view nesting)

- From **Cloud Host** to **Database**
  - Type: Aggregation (implicit from view nesting)

- From **3rd Party Host** to **AI Agent Service**
  - Type: Aggregation (implicit from view nesting)
