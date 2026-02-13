# ServiceNow Assignment Bot - Team Coding Guidelines

## Table of Contents
1. [Development Workflow](#development-workflow)
2. [Code Style & Standards](#code-style--standards)
3. [Git & Version Control](#git--version-control)
4. [Testing Requirements](#testing-requirements)
5. [Documentation Standards](#documentation-standards)
6. [Security Best Practices](#security-best-practices)
7. [Code Review Process](#code-review-process)
8. [Performance Guidelines](#performance-guidelines)

---

## Development Workflow

### Branch Strategy

We use **GitHub Flow** with the following conventions:

**Main Branches:**
- `main` - Production-ready code. Protected branch requiring 2 approvals.
- `develop` - Integration branch for features. Deploy to staging environment.

**Supporting Branches:**
- `feature/[ticket-number]-[brief-description]` - New features
  - Example: `feature/SNOW-123-assignment-algorithm`
- `fix/[ticket-number]-[brief-description]` - Bug fixes
  - Example: `fix/SNOW-456-null-pointer-error`
- `hotfix/[brief-description]` - Emergency production fixes
  - Example: `hotfix/servicenow-auth-failure`
- `refactor/[brief-description]` - Code refactoring
  - Example: `refactor/assignment-engine-cleanup`

### Development Cycle

1. **Create Branch** from `develop`
2. **Develop** with frequent, small commits
3. **Test** locally and write unit tests
4. **Push** to remote and create Pull Request
5. **Code Review** by at least 2 team members
6. **CI/CD** passes all automated checks
7. **Merge** to `develop` after approval
8. **Deploy** to staging for integration testing
9. **Merge** to `main` for production deployment

---

## Code Style & Standards

### Language-Specific Guidelines

#### JavaScript/TypeScript

**Use TypeScript** for all new code to ensure type safety.

```typescript
// Good - Strong typing
interface TicketAssignment {
  ticketId: string;
  assignedTo: string;
  confidence: number;
  timestamp: Date;
}

function assignTicket(ticket: ServiceNowTicket): TicketAssignment {
  // Implementation
}

// Bad - No types
function assignTicket(ticket) {
  return { ticketId: ticket.id, assignedTo: getAgent() };
}
```

**ESLint Configuration:**
```json
{
  "extends": ["airbnb-typescript", "prettier"],
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "max-len": ["error", { "code": 100 }],
    "complexity": ["error", 10]
  }
}
```

#### Python

**Follow PEP 8** standards strictly.

```python
# Good - Clear, typed, documented
from typing import Dict, Optional
from dataclasses import dataclass

@dataclass
class AssignmentResult:
    """Result of ticket assignment operation."""
    ticket_id: str
    assigned_to: str
    confidence: float
    
def assign_ticket(ticket: Dict[str, any]) -> Optional[AssignmentResult]:
    """
    Assign a ServiceNow ticket to an appropriate agent.
    
    Args:
        ticket: Dictionary containing ticket data
        
    Returns:
        AssignmentResult if successful, None otherwise
    """
    # Implementation
    pass

# Bad - No types, unclear
def assign(t):
    return {'id': t['id'], 'agent': get_agent()}
```

**Use Black for formatting** and **Pylint for linting**.

### Naming Conventions

**Variables and Functions:** camelCase (JS/TS) or snake_case (Python)
```javascript
const ticketPriority = 1;
function calculateConfidenceScore() {}
```

**Classes:** PascalCase
```javascript
class AssignmentEngine {}
class TicketProcessor {}
```

**Constants:** UPPER_SNAKE_CASE
```javascript
const MAX_ASSIGNMENT_RETRIES = 3;
const DEFAULT_CONFIDENCE_THRESHOLD = 0.85;
```

**Private Members:** Prefix with underscore
```javascript
class Agent {
  _internalState = {};
  _calculateWorkload() {}
}
```

### File Organization

```
project/
├── src/
│   ├── api/              # API routes and controllers
│   │   ├── routes/
│   │   └── controllers/
│   ├── services/         # Business logic
│   │   ├── assignment/
│   │   ├── servicenow/
│   │   └── analytics/
│   ├── models/           # Data models and schemas
│   ├── utils/            # Utility functions
│   ├── config/           # Configuration files
│   └── types/            # TypeScript type definitions
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/                 # Documentation
└── scripts/              # Build and deployment scripts
```

---

## Git & Version Control

### Commit Messages

Follow **Conventional Commits** specification:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**
```
feat(assignment): add skills-based routing algorithm

Implement new algorithm that matches ticket requirements with agent skills.
Uses cosine similarity for skill matching with 85% confidence threshold.

Closes #123
```

```
fix(servicenow): handle null priority values

ServiceNow sometimes returns null for priority field. Added defensive
check and default to priority 3 (Medium) when null.

Fixes #456
```

### Pull Request Guidelines

**PR Title Format:**
```
[SNOW-123] Add workload balancing to assignment engine
```

**PR Description Template:**
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manually tested

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests pass locally

## Related Issues
Closes #123
Related to #456
```

**Before Creating PR:**
1. Rebase on latest `develop`
2. Run all tests locally
3. Run linter and fix all issues
4. Update relevant documentation
5. Add meaningful PR description

---

## Testing Requirements

### Test Coverage Requirements

- **Minimum 70% overall coverage**
- **80% coverage for critical paths** (assignment logic, ServiceNow integration)
- **100% coverage for utility functions**

### Testing Pyramid

**Unit Tests (60%)** - Fast, isolated, test individual functions
```javascript
describe('AssignmentEngine', () => {
  describe('calculateConfidence', () => {
    it('should return high confidence for exact skill match', () => {
      const ticket = { category: 'Email', skills: ['Outlook'] };
      const agent = { skills: ['Outlook', 'Exchange'] };
      const confidence = calculateConfidence(ticket, agent);
      expect(confidence).toBeGreaterThan(0.9);
    });
  });
});
```

**Integration Tests (30%)** - Test component interactions
```javascript
describe('ServiceNow Integration', () => {
  it('should successfully assign ticket and update ServiceNow', async () => {
    const mockTicket = createMockTicket();
    const result = await assignmentService.processTicket(mockTicket);
    expect(result.success).toBe(true);
    expect(mockServiceNowAPI.updateTicket).toHaveBeenCalled();
  });
});
```

**E2E Tests (10%)** - Test complete workflows
```javascript
describe('Complete Assignment Flow', () => {
  it('should assign ticket from webhook to ServiceNow update', async () => {
    // Simulate webhook from ServiceNow
    const webhookPayload = {...};
    const response = await request(app)
      .post('/api/assign')
      .send(webhookPayload);
    
    expect(response.status).toBe(200);
    // Verify ticket was assigned in database
    // Verify ServiceNow was updated
  });
});
```

### Test Organization

- One test file per source file: `assignmentEngine.ts` → `assignmentEngine.test.ts`
- Use descriptive test names that explain the scenario
- Follow Arrange-Act-Assert pattern
- Mock external dependencies (ServiceNow API, database)
- Clean up test data after each test

### Running Tests

```bash
# Run all tests
npm test

# Run with coverage
npm run test:coverage

# Run specific test file
npm test -- assignmentEngine.test.ts

# Run in watch mode
npm run test:watch
```

---

## Documentation Standards

### Code Comments

**When to Comment:**
- Complex algorithms or business logic
- Non-obvious technical decisions
- Workarounds or hacks (with explanation)
- Public API functions

**When NOT to Comment:**
- Self-explanatory code
- Obvious operations
- Repeating what the code says

```javascript
// Bad - Obvious comment
// Get the user's name
const userName = user.name;

// Good - Explains WHY
// ServiceNow returns priority as string "1" instead of number
// We normalize it here to prevent downstream type errors
const priority = parseInt(ticket.priority, 10);

// Good - Explains complex logic
/**
 * Calculate assignment confidence using weighted scoring:
 * - Skills match: 40% weight
 * - Workload: 30% weight  
 * - Historical success rate: 20% weight
 * - Response time: 10% weight
 */
function calculateConfidence(ticket, agent) {
  // Implementation
}
```

### JSDoc / Docstrings

**All public functions must have documentation:**

```javascript
/**
 * Assigns a ServiceNow ticket to the most appropriate agent.
 *
 * Uses a multi-factor algorithm considering skills, workload,
 * and historical performance to select the optimal agent.
 *
 * @param {Object} ticket - ServiceNow ticket object
 * @param {string} ticket.sys_id - Unique ticket identifier
 * @param {string} ticket.category - Ticket category
 * @param {number} ticket.priority - Priority level (1-4)
 * @returns {Promise<AssignmentResult>} Assignment result with confidence score
 * @throws {AssignmentError} When no suitable agent is found
 *
 * @example
 * const result = await assignTicket({
 *   sys_id: 'abc123',
 *   category: 'Email',
 *   priority: 1
 * });
 */
async function assignTicket(ticket) {
  // Implementation
}
```

### README Files

Every major module should have a README:

```markdown
# Assignment Engine

## Overview
Core service that determines the best agent for each ticket.

## Architecture
[Brief diagram or description]

## Usage
```javascript
const engine = new AssignmentEngine(config);
const result = await engine.assign(ticket);
```

## Configuration
- `confidence_threshold`: Minimum confidence required (default: 0.85)
- `max_workload`: Maximum tickets per agent (default: 10)

## Testing
See `/tests/unit/assignmentEngine.test.ts`
```

---

## Security Best Practices

### Secrets Management

**NEVER commit secrets to Git:**
- API keys
- Passwords
- OAuth tokens
- Database credentials
- Private keys

**Use environment variables:**
```javascript
// Good
const apiKey = process.env.SERVICENOW_API_KEY;

// Bad - NEVER DO THIS
const apiKey = "sk_live_12345abcde";
```

**For local development:** Use `.env` files (add to `.gitignore`)
**For production:** Use secret management service (AWS Secrets Manager, HashiCorp Vault)

### Input Validation

**Always validate and sanitize inputs:**

```javascript
// Good - Validate before use
function processTicket(ticket) {
  if (!ticket || typeof ticket.sys_id !== 'string') {
    throw new ValidationError('Invalid ticket format');
  }
  
  const sanitized = {
    sys_id: sanitize(ticket.sys_id),
    description: sanitize(ticket.description),
    priority: parseInt(ticket.priority, 10)
  };
  
  // Process sanitized data
}

// Bad - Using raw input
function processTicket(ticket) {
  db.query(`SELECT * FROM tickets WHERE id = ${ticket.id}`); // SQL injection!
}
```

### Authentication & Authorization

**API Endpoints:**
- All endpoints require authentication
- Use JWT tokens or API keys
- Implement rate limiting (100 requests/minute)
- Log all authentication attempts

```javascript
// Example middleware
function requireAuth(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token || !verifyToken(token)) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  next();
}
```

### Data Protection

- **Encrypt sensitive data** at rest and in transit
- **Use HTTPS** for all API communications
- **Sanitize logs** - never log passwords or tokens
- **Implement audit trails** for all assignment actions

---

## Code Review Process

### Reviewer Responsibilities

**As a Reviewer:**
1. Review within 24 hours of PR creation
2. Check code quality, not just functionality
3. Verify tests are adequate
4. Ensure documentation is updated
5. Be constructive and respectful

**What to Look For:**
- Code correctness and logic errors
- Performance issues or inefficiencies
- Security vulnerabilities
- Code style violations
- Missing tests or documentation
- Edge cases not handled

### Review Checklist

```markdown
## Code Review Checklist

### Functionality
- [ ] Code solves the stated problem
- [ ] Edge cases are handled
- [ ] Error handling is appropriate

### Code Quality
- [ ] Code is readable and maintainable
- [ ] No code duplication
- [ ] Functions are small and focused
- [ ] Naming is clear and consistent

### Testing
- [ ] Unit tests cover new code
- [ ] Tests are meaningful
- [ ] All tests pass

### Documentation
- [ ] Code comments where needed
- [ ] README updated if applicable
- [ ] API docs updated

### Security
- [ ] No hardcoded secrets
- [ ] Inputs are validated
- [ ] No security vulnerabilities
```

### Giving Feedback

**Good Feedback:**
```
❌ Consider using async/await here instead of promises for better readability:

```javascript
// Instead of this
getAgent(id).then(agent => {
  return assignTicket(agent);
}).catch(err => {...});

// Try this
try {
  const agent = await getAgent(id);
  return await assignTicket(agent);
} catch (err) {...}
```

**Poor Feedback:**
```
This code is bad. Rewrite it.
```

### Approval Process

- **2 approvals required** for merging to `develop`
- **3 approvals required** for merging to `main`
- Author **cannot approve** their own PR
- All CI checks must pass
- No unresolved conversations

---

## Performance Guidelines

### General Principles

1. **Optimize for readability first**, then performance
2. **Profile before optimizing** - don't guess
3. **Benchmark changes** to verify improvements
4. **Consider time complexity** for algorithms

### Database Queries

**Use indexes:**
```sql
-- Good - Indexed query
CREATE INDEX idx_tickets_sys_id ON assignments(ticket_sys_id);
SELECT * FROM assignments WHERE ticket_sys_id = 'abc123';

-- Bad - Full table scan
SELECT * FROM assignments WHERE description LIKE '%email%';
```

**Avoid N+1 queries:**
```javascript
// Bad - N+1 problem
const tickets = await getTickets();
for (const ticket of tickets) {
  const agent = await getAgent(ticket.assigned_to); // N queries
}

// Good - Single query with join
const ticketsWithAgents = await getTicketsWithAgents();
```

### API Calls

**Batch requests when possible:**
```javascript
// Bad - Multiple calls
for (const ticket of tickets) {
  await updateServiceNow(ticket);
}

// Good - Batch update
await batchUpdateServiceNow(tickets);
```

**Implement caching:**
```javascript
const cache = new Map();

async function getAgentSkills(agentId) {
  if (cache.has(agentId)) {
    return cache.get(agentId);
  }
  
  const skills = await fetchSkillsFromDB(agentId);
  cache.set(agentId, skills);
  return skills;
}
```

### Response Time Targets

- **Webhook endpoint**: < 500ms
- **Assignment calculation**: < 2 seconds
- **ServiceNow API calls**: < 1 second
- **Database queries**: < 100ms

---

## Questions?

If you have questions about these guidelines:
1. Check this document first
2. Ask in #dev-general Slack channel
3. Bring up in daily standup
4. Suggest improvements via PR to this document

---

**Last Updated:** [Your Date]
**Maintained By:** [Team Lead Name]
