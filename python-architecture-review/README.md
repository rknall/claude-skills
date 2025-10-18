# Python Backend Architecture Review Skill

A comprehensive Claude Code skill for reviewing and analyzing Python backend application architectures.

## Overview

This skill provides expert-level architecture review capabilities covering all aspects of Python backend systems, from high-level design patterns to specific implementation details.

## What This Skill Reviews

- **System Architecture**: Monolithic vs microservices, service boundaries, communication patterns
- **Database Design**: Schema design, scaling strategies, ORM choices, query optimization
- **API Design**: REST/GraphQL/gRPC design, versioning, documentation
- **Security**: Authentication, authorization, encryption, vulnerability prevention
- **Scalability**: Horizontal/vertical scaling, caching, load balancing, async processing
- **Observability**: Logging, metrics, tracing, error tracking
- **Deployment**: Containerization, orchestration, CI/CD, infrastructure as code
- **Code Organization**: Project structure, dependency management, testing strategies
- **Resilience**: Retry patterns, circuit breakers, graceful degradation
- **Performance**: Optimization strategies, profiling, Python-specific considerations

## When to Use

Activate this skill when you need to:
- Review a backend architecture design document
- Get feedback on system design choices
- Analyze scalability and performance patterns
- Evaluate security architecture
- Assess database design decisions
- Review API design
- Get recommendations for technology stack
- Analyze code organization and structure

## Resources Included

### SKILL.md
The main skill definition with comprehensive review framework and instructions.

### architecture-checklist.md
A detailed checklist covering all aspects of backend architecture review. Use this for quick assessments or to ensure nothing is missed during reviews.

### common-patterns.md
Code examples and reference implementations for common architectural patterns:
- Repository Pattern
- Service Layer Pattern
- Dependency Injection
- Event-Driven Architecture
- Circuit Breaker Pattern
- CQRS Pattern
- Retry Patterns
- Background Task Processing
- API Versioning
- Middleware Patterns
- And more...

### technology-recommendations.md
Curated recommendations for technology choices including:
- Web frameworks (FastAPI, Django, Flask)
- Database solutions (PostgreSQL, MongoDB, Redis)
- ORMs (SQLAlchemy, Tortoise ORM)
- Authentication libraries
- Task queues (Celery, RQ, Dramatiq)
- Testing frameworks
- Observability tools
- Recommended stack combinations

## Installation

### From Marketplace

1. Add this marketplace to Claude Code:
```bash
/plugin marketplace add rknall/Skills
```

2. Install the skill:
```bash
/plugin install python-architecture-review
```

### Manual Installation

1. Clone or download this repository
2. Copy the `python-architecture-review` directory to `~/.claude/skills/`
3. Restart Claude Code or reload skills

## Usage Examples

### Example 1: Full Architecture Review

```
I have a design document for a new SaaS application backend. It will handle:
- 100K users initially, scaling to 1M
- Real-time notifications
- File uploads and processing
- Multi-tenant architecture
- PostgreSQL for main database
- Redis for caching
- FastAPI as the web framework

Can you review this architecture?
```

### Example 2: Specific Area Review

```
I'm designing the authentication system for my Python backend.
I'm planning to use JWT tokens with refresh tokens, stored in Redis.
Can you review this approach and suggest improvements?
```

### Example 3: Technology Stack Assessment

```
I'm choosing between Django and FastAPI for a new project that needs:
- Admin interface
- Complex business logic
- High read/write throughput
- Microservices architecture in the future

Which would you recommend?
```

### Example 4: Security Review

```
Here's my API security setup:
- JWT authentication
- CORS configured for frontend domain
- Rate limiting on all endpoints
- HTTPS only
- Environment variables for secrets

What am I missing from a security perspective?
```

## Review Output

The skill provides structured reviews including:

1. **Executive Summary**: High-level assessment and critical concerns
2. **Detailed Findings**: Analysis across all review areas with prioritized concerns
3. **Recommendations**: Specific, actionable improvements
4. **Technology Assessment**: Evaluation of chosen technologies
5. **Security Checklist**: Security-specific items to address
6. **Scalability Roadmap**: Scaling strategy and timeline
7. **Next Steps**: Prioritized action items

## Review Areas Covered

The skill evaluates architectures across these dimensions:

- System Architecture & Design Patterns
- Database Architecture
- API Design & Communication
- Security Architecture
- Scalability & Performance
- Observability & Monitoring
- Deployment & Infrastructure
- Code Organization & Project Structure
- Data Flow & State Management
- Resilience & Error Handling

Each area includes both general best practices and Python-specific considerations.

## Contributing

To improve this skill:

1. Update the SKILL.md file with new review criteria
2. Add new patterns to common-patterns.md
3. Update technology recommendations as new libraries emerge
4. Expand the checklist with additional items

## Version History

- **1.0.0** (2025-10-18): Initial release
  - Comprehensive review framework
  - 10+ architectural patterns
  - Technology recommendations
  - Complete checklist

## License

This skill is provided as-is for use with Claude Code.

## Support

For issues, questions, or suggestions, please contact the skill maintainer.
