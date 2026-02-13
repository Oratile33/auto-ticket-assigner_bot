# ServiceNow Auto-Assignment Bot - Integration Guide

## Architecture Overview

### System Components

1. **ServiceNow Instance** - Your existing ServiceNow platform
2. **Assignment Bot API** - Your custom bot service (Node.js/Python recommended)
3. **Database** - PostgreSQL/MongoDB for storing assignment rules and history
4. **Admin Dashboard** - Web UI for managing rules and monitoring

### Integration Flow

```
ServiceNow Ticket Created
    ↓
Business Rule triggers webhook
    ↓
Bot API receives ticket data
    ↓
Assignment Engine processes rules
    ↓
API updates ServiceNow ticket
    ↓
ServiceNow assigns to agent
```

## ServiceNow Setup

### 1. Create Integration User

Create a dedicated ServiceNow user with appropriate roles:
- `itil` role for incident management
- `rest_api_explorer` for API access
- Custom role for your specific assignment needs

### 2. Generate API Credentials

**OAuth 2.0 Method (Recommended)**:
- Navigate to: System OAuth > Application Registry
- Create new OAuth API endpoint
- Note: Client ID and Client Secret
- Set redirect URI to your bot's callback endpoint

**Basic Authentication Method**:
- Use integration user credentials
- Less secure, but simpler for development

### 3. Create Business Rule

Navigate to: System Definition > Business Rules

**Name**: Auto-Assignment Trigger
**Table**: Incident [incident]
**When**: after, insert
**Filter Conditions**: Assignment group is empty OR Assigned to is empty

**Script**:
```javascript
(function executeRule(current, previous) {
    try {
        var request = new sn_ws.RESTMessageV2();
        request.setEndpoint('https://your-bot-api.com/api/assign');
        request.setHttpMethod('POST');
        request.setRequestHeader('Content-Type', 'application/json');
        request.setRequestHeader('Authorization', 'Bearer YOUR_API_TOKEN');
        
        var payload = {
            sys_id: current.sys_id.toString(),
            number: current.number.toString(),
            short_description: current.short_description.toString(),
            description: current.description.toString(),
            priority: current.priority.toString(),
            category: current.category.toString(),
            urgency: current.urgency.toString(),
            impact: current.impact.toString(),
            caller_id: current.caller_id.sys_id.toString()
        };
        
        request.setRequestBody(JSON.stringify(payload));
        var response = request.execute();
        var responseBody = response.getBody();
        var httpStatus = response.getStatusCode();
        
        gs.info('Auto-assignment triggered for ' + current.number + 
                ' - Status: ' + httpStatus);
                
    } catch(ex) {
        gs.error('Auto-assignment failed: ' + ex.message);
    }
})(current, previous);
```

### 4. Create REST Message for Updates

Your bot will need to update tickets back to ServiceNow.

**Endpoint**: `https://your-instance.service-now.com/api/now/table/incident/{sys_id}`
**Method**: PATCH
**Headers**:
- `Content-Type: application/json`
- `Accept: application/json`
- `Authorization: Basic <base64_credentials>` or OAuth token

**Sample Payload**:
```json
{
    "assigned_to": "agent_sys_id",
    "assignment_group": "group_sys_id",
    "work_notes": "Auto-assigned by Assignment Bot based on skills match",
    "state": "2"
}
```

## Bot API Endpoints

### Receive Assignment Request

**POST** `/api/assign`

Request:
```json
{
    "sys_id": "abc123",
    "number": "INC0010001",
    "short_description": "Cannot access email",
    "description": "User reports unable to log into Outlook...",
    "priority": "2",
    "category": "Email",
    "urgency": "2",
    "impact": "2",
    "caller_id": "user123"
}
```

Response:
```json
{
    "success": true,
    "assigned_to": "john.doe",
    "assigned_to_sys_id": "xyz789",
    "assignment_group": "Email Support",
    "assignment_group_sys_id": "grp456",
    "reason": "Skills match: Email expertise, Workload: 3 tickets",
    "confidence": 0.92
}
```

### Assignment Rules Management

**GET** `/api/rules` - List all assignment rules
**POST** `/api/rules` - Create new rule
**PUT** `/api/rules/{id}` - Update rule
**DELETE** `/api/rules/{id}` - Delete rule

### Analytics Endpoints

**GET** `/api/analytics/assignments` - Assignment history
**GET** `/api/analytics/agents` - Agent performance metrics
**GET** `/api/analytics/accuracy` - Assignment accuracy rates

## Database Schema

### assignments table
```sql
CREATE TABLE assignments (
    id SERIAL PRIMARY KEY,
    ticket_sys_id VARCHAR(32) NOT NULL,
    ticket_number VARCHAR(40) NOT NULL,
    assigned_to_sys_id VARCHAR(32),
    assigned_to_name VARCHAR(100),
    assignment_group_sys_id VARCHAR(32),
    assignment_group_name VARCHAR(100),
    assignment_reason TEXT,
    confidence_score DECIMAL(3,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    servicenow_updated BOOLEAN DEFAULT FALSE
);
```

### assignment_rules table
```sql
CREATE TABLE assignment_rules (
    id SERIAL PRIMARY KEY,
    rule_name VARCHAR(100) NOT NULL,
    priority INTEGER DEFAULT 0,
    conditions JSONB NOT NULL,
    action JSONB NOT NULL,
    active BOOLEAN DEFAULT TRUE,
    created_by VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### agents table
```sql
CREATE TABLE agents (
    id SERIAL PRIMARY KEY,
    sys_id VARCHAR(32) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    skills JSONB,
    current_workload INTEGER DEFAULT 0,
    max_capacity INTEGER DEFAULT 10,
    timezone VARCHAR(50),
    active BOOLEAN DEFAULT TRUE,
    availability_schedule JSONB
);
```

## Security Considerations

1. **API Authentication**: Use API keys or OAuth tokens, rotate regularly
2. **Encryption**: All API calls should use HTTPS/TLS
3. **Rate Limiting**: Implement rate limiting on your bot API (100 req/min recommended)
4. **Input Validation**: Sanitize all inputs from ServiceNow
5. **Audit Logging**: Log all assignment decisions and API calls
6. **Secrets Management**: Use environment variables or secret management tools (AWS Secrets Manager, HashiCorp Vault)

## Testing Strategy

### Unit Tests
- Test assignment logic algorithms
- Test rule evaluation engine
- Test API request/response handling

### Integration Tests
- Mock ServiceNow API responses
- Test OAuth token refresh
- Test webhook payload handling

### End-to-End Tests
- Use ServiceNow developer instance
- Create test tickets
- Verify assignments complete successfully
- Test error scenarios (agent unavailable, no matching skills)

## Deployment Checklist

- [ ] ServiceNow integration user created
- [ ] OAuth credentials generated and secured
- [ ] Business rule deployed to ServiceNow
- [ ] Bot API deployed and accessible
- [ ] Database schema created and migrated
- [ ] Initial assignment rules configured
- [ ] Agent data synchronized from ServiceNow
- [ ] Monitoring and alerting configured
- [ ] Documentation completed
- [ ] Team trained on admin dashboard
- [ ] Pilot with 10-20 tickets
- [ ] Monitor for 1 week before full rollout

## Troubleshooting

**Problem**: Tickets not triggering bot
- Check Business Rule is active
- Verify filter conditions match your tickets
- Check ServiceNow script logs

**Problem**: Bot cannot update ServiceNow
- Verify API credentials are correct
- Check user permissions in ServiceNow
- Review API rate limits

**Problem**: Incorrect assignments
- Review assignment rule priority
- Check agent skills data is current
- Analyze assignment logs for patterns

## Maintenance

**Daily**: Monitor error logs and failed assignments
**Weekly**: Review assignment accuracy metrics
**Monthly**: Update agent skills and availability
**Quarterly**: Analyze patterns and optimize rules
