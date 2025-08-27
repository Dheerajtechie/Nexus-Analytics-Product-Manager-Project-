# 🏗️ System Architecture - DataInsight Pro

## Architecture Overview

DataInsight Pro is built on a modern, cloud-native architecture designed for scalability, reliability, and performance. The system follows microservices patterns with event-driven communication, enabling rapid development and deployment while maintaining high availability.

**Key Architectural Principles:**
- **Microservices Architecture**: Loosely coupled services for independent scaling
- **Event-Driven Design**: Asynchronous communication for better resilience
- **API-First Approach**: Every feature accessible via RESTful APIs
- **Cloud-Native**: Built for containerized deployment and auto-scaling
- **Security by Design**: Security considerations at every architectural layer

---

## 🎯 High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Applications                       │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Web App       │   Mobile App    │   Third-party Integrations │
│   (React)       │   (PWA)         │   (API Clients)            │
└─────────────────┴─────────────────┴─────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                     API Gateway (Kong)                          │
│  • Authentication  • Rate Limiting  • Load Balancing           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Microservices Layer                          │
├──────────────┬──────────────┬──────────────┬─────────────────────┤
│ User Service │ Dashboard    │ Query Engine │ Integration Service │
│              │ Service      │              │                     │
├──────────────┼──────────────┼──────────────┼─────────────────────┤
│ Report       │ Alert        │ Analytics    │ Notification        │
│ Service      │ Service      │ Service      │ Service             │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Message Queue (Kafka)                      │
│         • Event Streaming  • Service Communication              │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Data Layer                                │
├──────────────┬──────────────┬──────────────┬─────────────────────┤
│ PostgreSQL   │ Redis Cache  │ Elasticsearch│ Data Warehouse      │
│ (Primary DB) │ (Sessions)   │ (Search)     │ (Analytics)         │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

---

## 🔧 Technology Stack

### **Frontend Technologies**

#### **Web Application**
- **Framework**: React 18+ with TypeScript
- **State Management**: Redux Toolkit with RTK Query
- **UI Components**: Material-UI (MUI) with custom theme
- **Build Tool**: Vite with TypeScript compilation
- **Testing**: Jest + React Testing Library + Cypress

#### **Data Visualization**
- **Charts**: Chart.js with custom React wrappers
- **Interactive Graphics**: D3.js for complex visualizations
- **Maps**: Mapbox GL JS for geographic data
- **Real-time Updates**: WebSocket connections for live data

#### **Mobile Experience**
- **Type**: Progressive Web App (PWA)
- **Service Worker**: Workbox for offline functionality
- **Push Notifications**: Web Push API
- **App Shell**: Cached for instant loading

### **Backend Technologies**

#### **API Services**
- **Runtime**: Node.js 20+ with Express.js framework
- **Language**: TypeScript for type safety
- **API Documentation**: OpenAPI 3.0 with Swagger UI
- **Validation**: Joi for request/response validation
- **Authentication**: Passport.js with JWT strategy

#### **Database Systems**
- **Primary Database**: PostgreSQL 15+ with connection pooling
- **Caching**: Redis 7+ for session and query caching
- **Search Engine**: Elasticsearch 8+ for full-text search
- **Data Warehouse**: Amazon Redshift for analytics storage
- **Time Series**: InfluxDB for metrics and monitoring

#### **Message Queue & Processing**
- **Message Broker**: Apache Kafka for event streaming
- **Task Queue**: Bull Queue with Redis backend
- **Workflow Orchestration**: Apache Airflow for ETL
- **Stream Processing**: Apache Kafka Streams

### **Infrastructure & DevOps**

#### **Cloud Platform**
- **Primary**: Amazon Web Services (AWS)
- **Compute**: Amazon EKS (Kubernetes) for container orchestration
- **Storage**: Amazon S3 for file storage with CloudFront CDN
- **Networking**: Application Load Balancer with Auto Scaling

#### **Monitoring & Observability**
- **APM**: DataDog for application performance monitoring
- **Logging**: Centralized logging with ELK Stack
- **Metrics**: Prometheus with Grafana dashboards
- **Error Tracking**: Sentry for error monitoring and alerting

#### **Security & Compliance**
- **Web Application Firewall**: AWS WAF
- **Secrets Management**: AWS Secrets Manager
- **Certificate Management**: AWS Certificate Manager
- **Vulnerability Scanning**: Snyk for dependency scanning

---

## 🏛️ Microservices Architecture

### **Core Services**

#### **1. User Management Service**
**Responsibility**: User authentication, authorization, and profile management
**Technology**: Node.js + Express + PostgreSQL
**Endpoints**: `/auth/*`, `/users/*`, `/teams/*`

**Key Features:**
- JWT-based authentication with refresh tokens
- Role-based access control (RBAC)
- Multi-tenant organization management
- Single Sign-On (SSO) integration
- Password policies and security enforcement

**Database Schema:**
```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL DEFAULT 'user',
  organization_id UUID NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  last_login TIMESTAMP,
  is_active BOOLEAN DEFAULT true
);

-- Organizations table
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  plan VARCHAR(50) NOT NULL DEFAULT 'starter',
  billing_email VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **2. Dashboard Service**
**Responsibility**: Dashboard creation, management, and rendering
**Technology**: Node.js + Express + PostgreSQL + Redis
**Endpoints**: `/dashboards/*`, `/widgets/*`, `/templates/*`

**Key Features:**
- Drag-and-drop dashboard builder
- Widget library with 25+ visualization types
- Template management and sharing
- Real-time collaboration features
- Dashboard versioning and rollback

**Database Schema:**
```sql
-- Dashboards table
CREATE TABLE dashboards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  owner_id UUID NOT NULL REFERENCES users(id),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  configuration JSONB NOT NULL DEFAULT '{}',
  visibility VARCHAR(20) DEFAULT 'private',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Widgets table
CREATE TABLE widgets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dashboard_id UUID NOT NULL REFERENCES dashboards(id),
  type VARCHAR(50) NOT NULL,
  title VARCHAR(255) NOT NULL,
  position JSONB NOT NULL,
  configuration JSONB NOT NULL DEFAULT '{}',
  data_source_id UUID,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **3. Query Engine Service**
**Responsibility**: Data querying, caching, and result optimization
**Technology**: Node.js + Express + Redis + Connection Pooling
**Endpoints**: `/query/*`, `/data-sources/*`, `/cache/*`

**Key Features:**
- SQL query generation from visual interface
- Intelligent query caching and optimization
- Connection pooling for data sources
- Query result pagination and streaming
- Real-time query performance monitoring

**Query Processing Pipeline:**
```
User Request → Query Builder → Cache Check → Data Source Query → Result Processing → Response
     ↓              ↓             ↓              ↓                ↓              ↓
 Dashboard     SQL Generation   Redis Lookup   Database        Formatting    JSON Response
   Widget       Optimization    (Hit/Miss)     Execution       Aggregation   with Metadata
```

#### **4. Integration Service**
**Responsibility**: External data source connections and synchronization
**Technology**: Node.js + Express + Bull Queue + Airflow
**Endpoints**: `/integrations/*`, `/connectors/*`, `/sync/*`

**Key Features:**
- 100+ pre-built data source connectors
- OAuth 2.0 and API key authentication
- Scheduled and real-time data synchronization
- Schema discovery and data type mapping
- Error handling and retry mechanisms

**Connector Architecture:**
```javascript
// Base Connector Interface
class BaseConnector {
  constructor(credentials, config) {
    this.credentials = credentials;
    this.config = config;
  }

  async authenticate() {
    // Implement authentication logic
  }

  async discoverSchema() {
    // Discover available tables and columns
  }

  async syncData(table, lastSync) {
    // Sync data from external source
  }

  async validateConnection() {
    // Test connection health
  }
}

// Salesforce Connector Example
class SalesforceConnector extends BaseConnector {
  async authenticate() {
    const oauth = new OAuth2();
    return await oauth.authenticate(this.credentials);
  }

  async syncData(table, lastSync) {
    const soql = this.buildSOQLQuery(table, lastSync);
    return await this.salesforceAPI.query(soql);
  }
}
```

#### **5. Analytics Service**
**Responsibility**: Advanced analytics, statistical calculations, and ML
**Technology**: Python + FastAPI + Pandas + Scikit-learn
**Endpoints**: `/analytics/*`, `/forecasting/*`, `/insights/*`

**Key Features:**
- Statistical analysis (correlation, regression, significance testing)
- Time series forecasting and trend analysis
- Cohort analysis and customer segmentation
- Anomaly detection and alerting
- Machine learning model training and inference

**Analytics Pipeline:**
```python
# Analytics Processing Pipeline
class AnalyticsEngine:
    def __init__(self):
        self.processors = {
            'statistical': StatisticalProcessor(),
            'forecasting': ForecastingProcessor(),
            'segmentation': SegmentationProcessor(),
            'anomaly': AnomalyDetectionProcessor()
        }
    
    async def process_request(self, request_type, data, parameters):
        processor = self.processors.get(request_type)
        if not processor:
            raise ValueError(f"Unknown analytics type: {request_type}")
        
        return await processor.execute(data, parameters)

class ForecastingProcessor:
    def __init__(self):
        self.models = {
            'arima': ARIMAModel(),
            'linear': LinearRegressionModel(),
            'prophet': ProphetModel()
        }
    
    async def execute(self, data, parameters):
        model_type = parameters.get('model', 'arima')
        model = self.models[model_type]
        
        # Prepare data
        df = pd.DataFrame(data)
        
        # Train model
        model.fit(df)
        
        # Generate forecast
        forecast = model.predict(parameters.get('periods', 30))
        
        return {
            'forecast': forecast.to_dict(),
            'confidence_intervals': model.confidence_intervals(),
            'model_metrics': model.get_metrics()
        }
```

### **Supporting Services**

#### **6. Report Service**
**Responsibility**: Automated report generation and distribution
**Technology**: Node.js + Express + Puppeteer + Bull Queue
**Endpoints**: `/reports/*`, `/schedules/*`, `/exports/*`

#### **7. Notification Service**
**Responsibility**: Email, SMS, and push notifications
**Technology**: Node.js + Express + SendGrid + AWS SNS
**Endpoints**: `/notifications/*`, `/alerts/*`, `/webhooks/*`

#### **8. File Service**
**Responsibility**: File upload, storage, and CDN management
**Technology**: Node.js + Express + AWS S3 + CloudFront
**Endpoints**: `/files/*`, `/uploads/*`, `/exports/*`

---

## 💾 Data Architecture

### **Data Flow Architecture**

```
External Data Sources → Integration Service → Data Warehouse → Query Engine → Frontend
        ↓                      ↓                   ↓              ↓            ↓
    Salesforce             ETL Pipeline        Redshift       Cache Layer   Dashboard
    HubSpot               Data Validation     PostgreSQL      (Redis)       Widgets
    Google Analytics      Schema Mapping      Elasticsearch   Query Results  Reports
```

### **Database Design**

#### **Primary Database (PostgreSQL)**
**Purpose**: Operational data, user management, configurations
**Schema Design**: Normalized with proper indexing for performance

```sql
-- Core entities and relationships
CREATE TABLE organizations (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  plan VARCHAR(50) NOT NULL,
  settings JSONB DEFAULT '{}'
);

CREATE TABLE users (
  id UUID PRIMARY KEY,
  organization_id UUID REFERENCES organizations(id),
  email VARCHAR(255) UNIQUE NOT NULL,
  role VARCHAR(50) NOT NULL
);

CREATE TABLE data_sources (
  id UUID PRIMARY KEY,
  organization_id UUID REFERENCES organizations(id),
  name VARCHAR(255) NOT NULL,
  type VARCHAR(50) NOT NULL,
  credentials JSONB NOT NULL,
  configuration JSONB DEFAULT '{}',
  status VARCHAR(20) DEFAULT 'active',
  last_sync TIMESTAMP
);

CREATE TABLE dashboards (
  id UUID PRIMARY KEY,
  organization_id UUID REFERENCES organizations(id),
  owner_id UUID REFERENCES users(id),
  name VARCHAR(255) NOT NULL,
  configuration JSONB NOT NULL
);

-- Indexes for performance
CREATE INDEX idx_users_organization ON users(organization_id);
CREATE INDEX idx_dashboards_owner ON dashboards(owner_id);
CREATE INDEX idx_data_sources_org_type ON data_sources(organization_id, type);
```

#### **Data Warehouse (Amazon Redshift)**
**Purpose**: Analytics data storage and complex querying
**Schema Design**: Star schema optimized for analytical queries

```sql
-- Fact table for analytics events
CREATE TABLE fact_user_actions (
  action_id BIGINT IDENTITY(1,1) PRIMARY KEY,
  user_id UUID NOT NULL,
  organization_id UUID NOT NULL,
  action_type VARCHAR(50) NOT NULL,
  dashboard_id UUID,
  widget_id UUID,
  timestamp TIMESTAMP NOT NULL,
  session_id VARCHAR(255),
  properties JSON
) DISTKEY(organization_id) SORTKEY(timestamp);

-- Dimension tables
CREATE TABLE dim_users (
  user_id UUID PRIMARY KEY,
  organization_id UUID NOT NULL,
  email VARCHAR(255),
  role VARCHAR(50),
  created_date DATE
) DISTSTYLE ALL;

CREATE TABLE dim_dashboards (
  dashboard_id UUID PRIMARY KEY,
  organization_id UUID NOT NULL,
  name VARCHAR(255),
  category VARCHAR(100),
  created_date DATE
) DISTSTYLE ALL;
```

#### **Caching Layer (Redis)**
**Purpose**: Query result caching, session storage, real-time data
**Data Structures**: Hash sets, sorted sets, pub/sub channels

```javascript
// Cache key patterns
const CACHE_PATTERNS = {
  QUERY_RESULT: 'query:result:{hash}',
  USER_SESSION: 'session:{user_id}',
  DASHBOARD_CONFIG: 'dashboard:config:{dashboard_id}',
  REAL_TIME_DATA: 'realtime:{data_source}:{table}',
  RATE_LIMIT: 'ratelimit:{user_id}:{endpoint}'
};

// Cache implementation
class CacheManager {
  constructor() {
    this.redis = new Redis(process.env.REDIS_URL);
  }

  async cacheQueryResult(queryHash, result, ttl = 3600) {
    const key = CACHE_PATTERNS.QUERY_RESULT.replace('{hash}', queryHash);
    await this.redis.setex(key, ttl, JSON.stringify(result));
  }

  async getCachedQuery(queryHash) {
    const key = CACHE_PATTERNS.QUERY_RESULT.replace('{hash}', queryHash);
    const cached = await this.redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }
}
```

### **Data Synchronization Strategy**

#### **Real-time Synchronization**
- **Webhooks**: For data sources that support real-time notifications
- **Streaming APIs**: For continuous data ingestion (Kafka, Kinesis)
- **WebSocket Connections**: For bidirectional real-time updates

#### **Batch Synchronization**
- **Scheduled Jobs**: Hourly, daily, weekly sync based on data source
- **Incremental Updates**: Only sync changed records since last update
- **Full Refresh**: Complete data reload for accuracy verification

#### **Data Quality & Validation**
```python
# Data validation pipeline
class DataValidator:
    def __init__(self):
        self.rules = {
            'required_fields': self.validate_required_fields,
            'data_types': self.validate_data_types,
            'value_ranges': self.validate_value_ranges,
            'referential_integrity': self.validate_references
        }
    
    def validate_batch(self, data, schema):
        results = {
            'valid_records': [],
            'invalid_records': [],
            'validation_errors': []
        }
        
        for record in data:
            validation_result = self.validate_record(record, schema)
            if validation_result.is_valid:
                results['valid_records'].append(record)
            else:
                results['invalid_records'].append(record)
                results['validation_errors'].extend(validation_result.errors)
        
        return results
    
    def validate_record(self, record, schema):
        errors = []
        
        for rule_name, validator in self.rules.items():
            if rule_name in schema:
                rule_result = validator(record, schema[rule_name])
                if not rule_result.is_valid:
                    errors.extend(rule_result.errors)
        
        return ValidationResult(len(errors) == 0, errors)
```

---

## 🔐 Security Architecture

### **Security Layers**

#### **1. Network Security**
- **VPC Configuration**: Isolated network with private subnets
- **Security Groups**: Restrictive inbound/outbound rules
- **WAF Protection**: DDoS protection and request filtering
- **Load Balancer**: SSL termination with security headers

#### **2. Application Security**
- **Authentication**: Multi-factor authentication support
- **Authorization**: Role-based access control (RBAC)
- **Input Validation**: Comprehensive request validation
- **Output Encoding**: XSS prevention and data sanitization

#### **3. Data Security**
- **Encryption at Rest**: AES-256 for all stored data
- **Encryption in Transit**: TLS 1.3 for all communications
- **Key Management**: AWS KMS for encryption key rotation
- **Data Classification**: Sensitive data identification and protection

### **Authentication & Authorization**

#### **JWT Token Implementation**
```javascript
// JWT token structure
const tokenPayload = {
  sub: 'user_id',           // Subject (user identifier)
  iss: 'datainsight.pro',   // Issuer
  aud: 'api.datainsight.pro', // Audience
  exp: Math.floor(Date.now() / 1000) + (60 * 60), // Expiration (1 hour)
  iat: Math.floor(Date.now() / 1000), // Issued at
  jti: 'unique_token_id',   // JWT ID
  role: 'admin',            // User role
  org: 'organization_id',   // Organization context
  permissions: [            // Specific permissions
    'dashboards:read',
    'dashboards:write',
    'reports:read'
  ]
};

// Token validation middleware
const validateToken = async (req, res, next) => {
  try {
    const token = req.headers.authorization?.replace('Bearer ', '');
    if (!token) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await getUserById(decoded.sub);
    
    if (!user || !user.is_active) {
      return res.status(401).json({ error: 'Invalid or inactive user' });
    }

    req.user = user;
    req.permissions = decoded.permissions;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};
```

#### **Permission-Based Access Control**
```javascript
// Permission checking middleware
const requirePermission = (permission) => {
  return (req, res, next) => {
    if (!req.permissions || !req.permissions.includes(permission)) {
      return res.status(403).json({ 
        error: 'Insufficient permissions',
        required: permission,
        granted: req.permissions || []
      });
    }
    next();
  };
};

// Usage in routes
app.get('/dashboards', 
  validateToken, 
  requirePermission('dashboards:read'), 
  getDashboards
);

app.post('/dashboards', 
  validateToken, 
  requirePermission('dashboards:write'), 
  createDashboard
);
```

### **Data Privacy & Compliance**

#### **GDPR Compliance Features**
- **Data Minimization**: Collect only necessary data
- **Consent Management**: Explicit consent for data processing
- **Right to Access**: API endpoints for data export
- **Right to Deletion**: Secure data deletion processes
- **Data Portability**: Standardized data export formats

#### **SOC 2 Compliance Controls**
- **Security**: Access controls, encryption, monitoring
- **Availability**: Uptime monitoring, disaster recovery
- **Processing Integrity**: Data validation, error handling
- **Confidentiality**: Data classification, access logging
- **Privacy**: Data handling policies, consent management

---

## 📈 Scalability & Performance

### **Horizontal Scaling Strategy**

#### **Auto Scaling Configuration**
```yaml
# Kubernetes Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: dashboard-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: dashboard-service
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

#### **Database Scaling**
- **Read Replicas**: PostgreSQL read replicas for query distribution
- **Connection Pooling**: PgBouncer for connection management
- **Query Optimization**: Automated index recommendations
- **Partitioning**: Time-based table partitioning for large datasets

### **Caching Strategy**

#### **Multi-Level Caching**
```javascript
// Cache hierarchy implementation
class CacheHierarchy {
  constructor() {
    this.levels = [
      new MemoryCache(),      // L1: In-memory cache (fastest)
      new RedisCache(),       // L2: Redis cache (fast)
      new DatabaseCache()     // L3: Database cache (slower but persistent)
    ];
  }

  async get(key) {
    for (let i = 0; i < this.levels.length; i++) {
      const value = await this.levels[i].get(key);
      if (value !== null) {
        // Promote to higher cache levels
        for (let j = 0; j < i; j++) {
          await this.levels[j].set(key, value);
        }
        return value;
      }
    }
    return null;
  }

  async set(key, value, ttl) {
    // Set in all cache levels
    const promises = this.levels.map(cache => 
      cache.set(key, value, ttl)
    );
    await Promise.all(promises);
  }
}
```

#### **Query Result Caching**
- **Smart Cache Keys**: Hash-based keys including query and parameters
- **Cache Invalidation**: Event-driven cache invalidation on data updates
- **TTL Strategy**: Adaptive TTL based on data freshness requirements
- **Cache Warming**: Proactive cache population for common queries

### **Performance Monitoring**

#### **Key Performance Metrics**
```javascript
// Performance monitoring middleware
const performanceMonitor = (req, res, next) => {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    const metrics = {
      method: req.method,
      route: req.route?.path || req.url,
      status_code: res.statusCode,
      duration_ms: duration,
      user_id: req.user?.id,
      organization_id: req.user?.organization_id,
      timestamp: new Date().toISOString()
    };
    
    // Send metrics to monitoring system
    metricsCollector.record('api_request', metrics);
    
    // Alert on slow requests
    if (duration > 5000) {
      alertManager.sendAlert('slow_request', {
        ...metrics,
        threshold: 5000
      });
    }
  });
  
  next();
};
```

---

## 🔄 DevOps & Deployment

### **CI/CD Pipeline**

#### **GitHub Actions Workflow**
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '20'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm run test:coverage
    
    - name: Run security audit
      run: npm audit --audit-level high
    
    - name: Build application
      run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Deploy to EKS
      run: |
        kubectl set image deployment/api-service \
          api-service=$ECR_REGISTRY/api-service:$GITHUB_SHA
        kubectl rollout status deployment/api-service
```

### **Infrastructure as Code**

#### **Terraform Configuration**
```hcl
# EKS Cluster configuration
resource "aws_eks_cluster" "datainsight" {
  name     = "datainsight-production"
  role_arn = aws_iam_role.cluster.arn
  version  = "1.28"

  vpc_config {
    subnet_ids              = aws_subnet.private[*].id
    endpoint_private_access = true
    endpoint_public_access  = true
    public_access_cidrs     = ["0.0.0.0/0"]
  }

  encryption_config {
    provider {
      key_arn = aws_kms_key.eks.arn
    }
    resources = ["secrets"]
  }

  enabled_cluster_log_types = [
    "api", "audit", "authenticator", 
    "controllerManager", "scheduler"
  ]

  depends_on = [
    aws_iam_role_policy_attachment.cluster_policy,
    aws_iam_role_policy_attachment.vpc_resource_controller,
  ]
}

# RDS PostgreSQL configuration
resource "aws_db_instance" "primary" {
  identifier = "datainsight-primary"
  
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.r6g.xlarge"
  
  allocated_storage     = 100
  max_allocated_storage = 1000
  storage_type          = "gp3"
  storage_encrypted     = true
  
  db_name  = "datainsight"
  username = "postgres"
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  skip_final_snapshot = false
  final_snapshot_identifier = "datainsight-final-snapshot"
  
  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn        = aws_iam_role.rds_monitoring.arn
}
```

### **Container Orchestration**

#### **Kubernetes Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
  namespace: datainsight
spec:
  replicas: 5
  selector:
    matchLabels:
      app: api-service
  template:
    metadata:
      labels:
        app: api-service
    spec:
      containers:
      - name: api-service
        image: datainsight/api-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

## 🚨 Monitoring & Alerting

### **Observability Stack**

#### **Application Performance Monitoring**
```javascript
// APM integration with DataDog
const tracer = require('dd-trace').init({
  service: 'datainsight-api',
  env: process.env.NODE_ENV,
  version: process.env.APP_VERSION,
  analytics: true,
  runtimeMetrics: true,
  profiling: true
});

// Custom metrics collection
const { StatsD } = require('node-statsd');
const metrics = new StatsD({
  host: process.env.STATSD_HOST,
  port: process.env.STATSD_PORT,
  prefix: 'datainsight.api.'
});

// Business metrics tracking
class MetricsCollector {
  recordDashboardView(userId, dashboardId) {
    metrics.increment('dashboard.views', 1, {
      user_id: userId,
      dashboard_id: dashboardId
    });
  }

  recordQueryExecution(duration, success, dataSource) {
    metrics.histogram('query.duration', duration, {
      success: success.toString(),
      data_source: dataSource
    });
  }

  recordUserSignup(plan, source) {
    metrics.increment('user.signup', 1, {
      plan: plan,
      source: source
    });
  }
}
```

#### **Health Check Implementation**
```javascript
// Comprehensive health checks
class HealthChecker {
  constructor() {
    this.checks = {
      database: this.checkDatabase,
      redis: this.checkRedis,
      external_apis: this.checkExternalAPIs,
      disk_space: this.checkDiskSpace,
      memory: this.checkMemoryUsage
    };
  }

  async checkHealth() {
    const results = {};
    const overallHealth = { status: 'healthy', checks: results };

    for (const [name, check] of Object.entries(this.checks)) {
      try {
        const result = await check.call(this);
        results[name] = { status: 'healthy', ...result };
      } catch (error) {
        results[name] = { 
          status: 'unhealthy', 
          error: error.message,
          timestamp: new Date().toISOString()
        };
        overallHealth.status = 'unhealthy';
      }
    }

    return overallHealth;
  }

  async checkDatabase() {
    const start = Date.now();
    await db.query('SELECT 1');
    const duration = Date.now() - start;
    
    return {
      response_time_ms: duration,
      connection_pool: {
        total: db.pool.totalCount,
        idle: db.pool.idleCount,
        waiting: db.pool.waitingCount
      }
    };
  }
}
```

### **Alerting Configuration**

#### **Alert Rules**
```yaml
# Prometheus alerting rules
groups:
- name: datainsight.rules
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
    for: 5m
    labels:
      severity: critical
      service: api
    annotations:
      summary: "High error rate detected"
      description: "Error rate is {{ $value }} errors per second"

  - alert: DatabaseConnectionPoolExhausted
    expr: postgresql_connection_pool_used / postgresql_connection_pool_max > 0.9
    for: 2m
    labels:
      severity: warning
      service: database
    annotations:
      summary: "Database connection pool nearly exhausted"
      description: "{{ $value }}% of database connections in use"

  - alert: SlowQueryDetected
    expr: histogram_quantile(0.95, query_duration_seconds) > 5
    for: 1m
    labels:
      severity: warning
      service: query-engine
    annotations:
      summary: "Slow queries detected"
      description: "95th percentile query time is {{ $value }} seconds"
```

---

*This system architecture documentation represents enterprise-grade technical design suitable for scaling to millions of users and petabytes of data. It demonstrates comprehensive understanding of modern cloud-native architecture patterns, security best practices, and operational excellence.*

**Next Steps**: Review the Metrics & Analytics documentation for KPI tracking and success measurement frameworks.
