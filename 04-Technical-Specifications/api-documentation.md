# 🔧 API Documentation - DataInsight Pro

## API Overview

DataInsight Pro provides a comprehensive RESTful API that enables developers to integrate analytics capabilities into their applications, automate data workflows, and build custom solutions on top of our platform.

**Base URL**: `https://api.datainsight.pro/v1`
**Authentication**: Bearer Token (JWT)
**Rate Limiting**: 1000 requests/hour per API key
**Response Format**: JSON
**API Version**: v1.0

---

## 🔐 Authentication

### Bearer Token Authentication
All API requests must include a valid Bearer token in the Authorization header.

```http
Authorization: Bearer {your_api_token}
```

### API Key Management
API keys can be generated and managed through the DataInsight Pro dashboard:
1. Navigate to Settings → API Keys
2. Click "Generate New Key"
3. Copy and securely store your API key
4. Use the key in the Authorization header

### Token Expiration
- **Access Tokens**: Expire after 24 hours
- **Refresh Tokens**: Expire after 30 days
- **API Keys**: No expiration (can be revoked manually)

---

## 📊 Core API Endpoints

### **Dashboards API**

#### GET /dashboards
Retrieve all dashboards for the authenticated user.

**Parameters:**
- `limit` (optional): Number of dashboards to return (default: 50, max: 100)
- `offset` (optional): Number of dashboards to skip (default: 0)
- `sort` (optional): Sort order (created_at, updated_at, name)
- `order` (optional): Sort direction (asc, desc)

**Response:**
```json
{
  "data": [
    {
      "id": "dash_1234567890",
      "name": "Sales Performance Dashboard",
      "description": "Monthly sales metrics and KPIs",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-20T14:45:00Z",
      "owner": {
        "id": "user_0987654321",
        "name": "Sarah Chen",
        "email": "sarah@company.com"
      },
      "visibility": "team",
      "widget_count": 12,
      "last_viewed": "2024-01-22T09:15:00Z"
    }
  ],
  "pagination": {
    "total": 25,
    "limit": 50,
    "offset": 0,
    "has_more": false
  }
}
```

#### POST /dashboards
Create a new dashboard.

**Request Body:**
```json
{
  "name": "Marketing ROI Dashboard",
  "description": "Track marketing campaign performance and ROI",
  "visibility": "team",
  "template_id": "tmpl_marketing_001",
  "tags": ["marketing", "roi", "campaigns"]
}
```

**Response:**
```json
{
  "data": {
    "id": "dash_2345678901",
    "name": "Marketing ROI Dashboard",
    "description": "Track marketing campaign performance and ROI",
    "created_at": "2024-01-22T16:20:00Z",
    "updated_at": "2024-01-22T16:20:00Z",
    "owner": {
      "id": "user_0987654321",
      "name": "Sarah Chen",
      "email": "sarah@company.com"
    },
    "visibility": "team",
    "widget_count": 0,
    "share_url": "https://app.datainsight.pro/dashboard/dash_2345678901"
  }
}
```

#### GET /dashboards/{dashboard_id}
Retrieve a specific dashboard by ID.

**Parameters:**
- `include_widgets` (optional): Include widget configurations (default: false)

**Response:**
```json
{
  "data": {
    "id": "dash_1234567890",
    "name": "Sales Performance Dashboard",
    "description": "Monthly sales metrics and KPIs",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-20T14:45:00Z",
    "owner": {
      "id": "user_0987654321",
      "name": "Sarah Chen",
      "email": "sarah@company.com"
    },
    "visibility": "team",
    "widgets": [
      {
        "id": "widget_1111111111",
        "type": "line_chart",
        "title": "Monthly Revenue Trend",
        "position": { "x": 0, "y": 0, "width": 6, "height": 4 },
        "data_source": "salesforce",
        "configuration": {
          "x_axis": "month",
          "y_axis": "revenue",
          "filters": [
            {
              "field": "stage",
              "operator": "equals",
              "value": "closed_won"
            }
          ]
        }
      }
    ]
  }
}
```

### **Data Sources API**

#### GET /data-sources
Retrieve all connected data sources.

**Response:**
```json
{
  "data": [
    {
      "id": "ds_salesforce_001",
      "name": "Salesforce Production",
      "type": "salesforce",
      "status": "connected",
      "last_sync": "2024-01-22T15:30:00Z",
      "sync_frequency": "hourly",
      "tables": [
        {
          "name": "opportunities",
          "record_count": 15420,
          "last_updated": "2024-01-22T15:30:00Z"
        },
        {
          "name": "accounts",
          "record_count": 8930,
          "last_updated": "2024-01-22T15:30:00Z"
        }
      ],
      "configuration": {
        "instance_url": "https://company.salesforce.com",
        "api_version": "v58.0"
      }
    }
  ]
}
```

#### POST /data-sources
Connect a new data source.

**Request Body:**
```json
{
  "name": "HubSpot Marketing",
  "type": "hubspot",
  "credentials": {
    "access_token": "your_hubspot_token",
    "portal_id": "12345678"
  },
  "sync_frequency": "daily",
  "tables": ["contacts", "deals", "companies", "campaigns"]
}
```

#### GET /data-sources/{source_id}/schema
Retrieve the schema for a specific data source.

**Response:**
```json
{
  "data": {
    "tables": [
      {
        "name": "opportunities",
        "display_name": "Sales Opportunities",
        "record_count": 15420,
        "columns": [
          {
            "name": "id",
            "display_name": "Opportunity ID",
            "type": "string",
            "nullable": false,
            "primary_key": true
          },
          {
            "name": "amount",
            "display_name": "Deal Amount",
            "type": "decimal",
            "nullable": true,
            "format": "currency"
          },
          {
            "name": "close_date",
            "display_name": "Close Date",
            "type": "date",
            "nullable": true
          }
        ]
      }
    ]
  }
}
```

### **Query API**

#### POST /query
Execute a data query and return results.

**Request Body:**
```json
{
  "data_source": "salesforce",
  "query": {
    "table": "opportunities",
    "select": [
      "close_date",
      "amount",
      "stage_name"
    ],
    "filters": [
      {
        "field": "close_date",
        "operator": "between",
        "value": ["2024-01-01", "2024-01-31"]
      },
      {
        "field": "stage_name",
        "operator": "equals",
        "value": "Closed Won"
      }
    ],
    "group_by": ["close_date"],
    "aggregations": [
      {
        "field": "amount",
        "function": "sum",
        "alias": "total_revenue"
      }
    ],
    "order_by": [
      {
        "field": "close_date",
        "direction": "asc"
      }
    ],
    "limit": 100
  }
}
```

**Response:**
```json
{
  "data": {
    "columns": [
      {
        "name": "close_date",
        "type": "date"
      },
      {
        "name": "total_revenue",
        "type": "decimal"
      }
    ],
    "rows": [
      {
        "close_date": "2024-01-01",
        "total_revenue": 125000.00
      },
      {
        "close_date": "2024-01-02",
        "total_revenue": 87500.00
      }
    ],
    "metadata": {
      "total_rows": 31,
      "execution_time_ms": 245,
      "cached": false
    }
  }
}
```

### **Reports API**

#### GET /reports
Retrieve all reports for the authenticated user.

**Response:**
```json
{
  "data": [
    {
      "id": "report_1234567890",
      "name": "Weekly Sales Report",
      "description": "Automated weekly sales performance report",
      "dashboard_id": "dash_1234567890",
      "schedule": {
        "frequency": "weekly",
        "day_of_week": "monday",
        "time": "09:00",
        "timezone": "America/New_York"
      },
      "format": "pdf",
      "recipients": [
        {
          "email": "ceo@company.com",
          "name": "John Smith"
        }
      ],
      "last_sent": "2024-01-22T09:00:00Z",
      "next_scheduled": "2024-01-29T09:00:00Z",
      "status": "active"
    }
  ]
}
```

#### POST /reports
Create a new automated report.

**Request Body:**
```json
{
  "name": "Monthly Executive Dashboard",
  "description": "Monthly executive summary with key metrics",
  "dashboard_id": "dash_1234567890",
  "schedule": {
    "frequency": "monthly",
    "day_of_month": 1,
    "time": "08:00",
    "timezone": "America/New_York"
  },
  "format": "pdf",
  "recipients": [
    {
      "email": "ceo@company.com",
      "name": "John Smith"
    },
    {
      "email": "cfo@company.com",
      "name": "Jane Doe"
    }
  ],
  "include_comments": true,
  "branding": {
    "logo_url": "https://company.com/logo.png",
    "company_name": "Acme Corp"
  }
}
```

### **Users & Teams API**

#### GET /users/me
Retrieve current user information.

**Response:**
```json
{
  "data": {
    "id": "user_0987654321",
    "name": "Sarah Chen",
    "email": "sarah@company.com",
    "role": "admin",
    "team": {
      "id": "team_1111111111",
      "name": "Operations Team",
      "plan": "professional"
    },
    "permissions": [
      "dashboards:read",
      "dashboards:write",
      "reports:read",
      "reports:write",
      "data_sources:read",
      "data_sources:write",
      "users:read"
    ],
    "preferences": {
      "timezone": "America/New_York",
      "date_format": "MM/DD/YYYY",
      "currency": "USD"
    },
    "last_login": "2024-01-22T08:30:00Z"
  }
}
```

#### GET /teams/{team_id}/members
Retrieve team members.

**Response:**
```json
{
  "data": [
    {
      "id": "user_0987654321",
      "name": "Sarah Chen",
      "email": "sarah@company.com",
      "role": "admin",
      "status": "active",
      "last_login": "2024-01-22T08:30:00Z",
      "joined_at": "2023-06-15T10:00:00Z"
    },
    {
      "id": "user_1234567890",
      "name": "Marcus Rodriguez",
      "email": "marcus@company.com",
      "role": "editor",
      "status": "active",
      "last_login": "2024-01-21T16:45:00Z",
      "joined_at": "2023-08-20T14:30:00Z"
    }
  ]
}
```

---

## 🔌 Webhook API

### Webhook Events
DataInsight Pro can send webhook notifications for various events:

- `dashboard.created`
- `dashboard.updated` 
- `dashboard.deleted`
- `report.generated`
- `report.failed`
- `data_source.connected`
- `data_source.sync_completed`
- `data_source.sync_failed`
- `alert.triggered`

### Webhook Configuration

#### POST /webhooks
Create a new webhook endpoint.

**Request Body:**
```json
{
  "url": "https://your-app.com/webhooks/datainsight",
  "events": [
    "dashboard.created",
    "report.generated",
    "alert.triggered"
  ],
  "secret": "your_webhook_secret",
  "active": true
}
```

### Webhook Payload Example

```json
{
  "event": "report.generated",
  "timestamp": "2024-01-22T10:00:00Z",
  "data": {
    "report_id": "report_1234567890",
    "name": "Weekly Sales Report",
    "dashboard_id": "dash_1234567890",
    "format": "pdf",
    "file_url": "https://files.datainsight.pro/reports/report_1234567890.pdf",
    "recipients": ["ceo@company.com"],
    "generated_at": "2024-01-22T10:00:00Z"
  }
}
```

---

## 📈 Analytics & Metrics API

### **Usage Analytics**

#### GET /analytics/usage
Retrieve platform usage analytics.

**Parameters:**
- `start_date`: Start date (YYYY-MM-DD)
- `end_date`: End date (YYYY-MM-DD)
- `granularity`: Time granularity (day, week, month)

**Response:**
```json
{
  "data": {
    "period": {
      "start_date": "2024-01-01",
      "end_date": "2024-01-31",
      "granularity": "day"
    },
    "metrics": [
      {
        "date": "2024-01-01",
        "active_users": 45,
        "dashboard_views": 234,
        "queries_executed": 567,
        "reports_generated": 12
      },
      {
        "date": "2024-01-02",
        "active_users": 52,
        "dashboard_views": 287,
        "queries_executed": 623,
        "reports_generated": 8
      }
    ],
    "totals": {
      "total_active_users": 1420,
      "total_dashboard_views": 8950,
      "total_queries_executed": 18430,
      "total_reports_generated": 234
    }
  }
}
```

### **Performance Metrics**

#### GET /analytics/performance
Retrieve platform performance metrics.

**Response:**
```json
{
  "data": {
    "query_performance": {
      "average_execution_time_ms": 245,
      "p50_execution_time_ms": 180,
      "p95_execution_time_ms": 450,
      "p99_execution_time_ms": 890
    },
    "dashboard_performance": {
      "average_load_time_ms": 1200,
      "p50_load_time_ms": 950,
      "p95_load_time_ms": 2100,
      "p99_load_time_ms": 3400
    },
    "data_freshness": {
      "average_lag_minutes": 15,
      "sources_real_time": 8,
      "sources_hourly": 12,
      "sources_daily": 5
    }
  }
}
```

---

## 🚨 Error Handling

### HTTP Status Codes
- `200 OK`: Request successful
- `201 Created`: Resource created successfully
- `400 Bad Request`: Invalid request parameters
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

### Error Response Format
```json
{
  "error": {
    "code": "INVALID_PARAMETERS",
    "message": "The request contains invalid parameters",
    "details": [
      {
        "field": "start_date",
        "message": "Date format must be YYYY-MM-DD"
      }
    ],
    "request_id": "req_1234567890",
    "timestamp": "2024-01-22T10:30:00Z"
  }
}
```

### Common Error Codes
- `AUTHENTICATION_REQUIRED`: Valid authentication token required
- `INSUFFICIENT_PERMISSIONS`: User lacks required permissions
- `RESOURCE_NOT_FOUND`: Requested resource does not exist
- `INVALID_PARAMETERS`: Request parameters are invalid
- `RATE_LIMIT_EXCEEDED`: Too many requests in time window
- `DATA_SOURCE_UNAVAILABLE`: Data source is temporarily unavailable
- `QUERY_TIMEOUT`: Query execution exceeded timeout limit

---

## 🔄 Rate Limiting

### Rate Limits by Endpoint
- **Authentication**: 10 requests/minute
- **Dashboards**: 100 requests/hour
- **Data Sources**: 50 requests/hour
- **Query Execution**: 200 requests/hour
- **Reports**: 25 requests/hour
- **Webhooks**: 10 requests/minute

### Rate Limit Headers
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1642867200
X-RateLimit-Window: 3600
```

### Rate Limit Exceeded Response
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 45 minutes.",
    "retry_after": 2700,
    "limit": 100,
    "window": 3600
  }
}
```

---

## 📝 SDK & Code Examples

### JavaScript/Node.js SDK

#### Installation
```bash
npm install @datainsight/api-client
```

#### Basic Usage
```javascript
const DataInsightAPI = require('@datainsight/api-client');

const client = new DataInsightAPI({
  apiKey: 'your_api_key',
  baseURL: 'https://api.datainsight.pro/v1'
});

// Get all dashboards
const dashboards = await client.dashboards.list({
  limit: 10,
  sort: 'updated_at',
  order: 'desc'
});

// Create a new dashboard
const newDashboard = await client.dashboards.create({
  name: 'Sales Analytics',
  description: 'Real-time sales performance metrics',
  visibility: 'team'
});

// Execute a query
const results = await client.query.execute({
  data_source: 'salesforce',
  query: {
    table: 'opportunities',
    select: ['amount', 'close_date'],
    filters: [
      {
        field: 'stage_name',
        operator: 'equals',
        value: 'Closed Won'
      }
    ]
  }
});
```

### Python SDK

#### Installation
```bash
pip install datainsight-api
```

#### Basic Usage
```python
from datainsight import DataInsightClient

client = DataInsightClient(
    api_key='your_api_key',
    base_url='https://api.datainsight.pro/v1'
)

# Get all dashboards
dashboards = client.dashboards.list(
    limit=10,
    sort='updated_at',
    order='desc'
)

# Create a new dashboard
new_dashboard = client.dashboards.create({
    'name': 'Marketing ROI Dashboard',
    'description': 'Track marketing campaign performance',
    'visibility': 'team'
})

# Execute a query
results = client.query.execute({
    'data_source': 'hubspot',
    'query': {
        'table': 'deals',
        'select': ['amount', 'dealstage'],
        'filters': [
            {
                'field': 'createdate',
                'operator': 'between',
                'value': ['2024-01-01', '2024-01-31']
            }
        ]
    }
})
```

### cURL Examples

#### Authentication
```bash
curl -X GET "https://api.datainsight.pro/v1/dashboards" \
  -H "Authorization: Bearer your_api_token" \
  -H "Content-Type: application/json"
```

#### Create Dashboard
```bash
curl -X POST "https://api.datainsight.pro/v1/dashboards" \
  -H "Authorization: Bearer your_api_token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Executive Dashboard",
    "description": "High-level business metrics",
    "visibility": "private"
  }'
```

#### Execute Query
```bash
curl -X POST "https://api.datainsight.pro/v1/query" \
  -H "Authorization: Bearer your_api_token" \
  -H "Content-Type: application/json" \
  -d '{
    "data_source": "salesforce",
    "query": {
      "table": "opportunities",
      "select": ["amount", "close_date", "stage_name"],
      "limit": 100
    }
  }'
```

---

## 🔒 Security & Compliance

### API Security Features
- **TLS 1.3 Encryption**: All API communications encrypted in transit
- **JWT Authentication**: Secure token-based authentication
- **Rate Limiting**: Prevents abuse and ensures fair usage
- **IP Whitelisting**: Optional IP-based access restrictions
- **Audit Logging**: Complete audit trail of all API requests

### Compliance Standards
- **SOC 2 Type II**: Annual compliance audits
- **GDPR**: EU data protection compliance
- **HIPAA**: Healthcare data protection (optional)
- **ISO 27001**: Information security management

### Data Protection
- **Encryption at Rest**: AES-256 encryption for stored data
- **Data Residency**: Choose data storage location
- **Data Retention**: Configurable data retention policies
- **Right to Deletion**: GDPR-compliant data deletion

---

## 📞 API Support

### Getting Help
- **Documentation**: https://docs.datainsight.pro/api
- **Support Email**: api-support@datainsight.pro
- **Developer Forum**: https://community.datainsight.pro/developers
- **Status Page**: https://status.datainsight.pro

### Response Times
- **Critical Issues**: 2 hours
- **High Priority**: 8 hours
- **Medium Priority**: 24 hours
- **Low Priority**: 72 hours

### API Changelog
All API changes are documented and communicated through:
- Email notifications to API users
- Changelog on documentation site
- Version headers in API responses
- Deprecation warnings with 6-month notice

---

*This API documentation represents a comprehensive technical interface designed for enterprise-grade integrations and developer productivity. Updated with each API version release.*

**Next Steps**: Review the Technical Architecture document for system design and implementation details.
