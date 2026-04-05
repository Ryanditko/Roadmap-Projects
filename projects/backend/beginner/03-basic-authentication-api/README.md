# Basic Authentication API (hardcoded users)

## Idea
Implement a simple authentication system with hardcoded credentials. Learn the fundamentals of user validation, session management, and protected endpoints without complex database or OAuth integration.

## Learning Objectives
- Understand authentication vs authorization
- Implement login/logout flows
- Work with credentials and password validation
- Manage user sessions/tokens
- Protect API endpoints with middleware
- Understand basic security concepts

## Implementation Tips
- Start with hardcoded user credentials
- Implement login endpoint that validates credentials
- Generate a simple token or session ID on successful login
- Use middleware to check token presence on protected routes
- Store active sessions in memory
- Implement logout to invalidate sessions
- Add refresh token concept (optional)
- Consider token expiration times
- Use environment variables for secrets (even if hardcoded)
- Add role-based access control (admin/user)
- Implement password hashing (even for hardcoded users)

## Key Challenges
- Token generation and validation
- Session management
- Middleware implementation
- Security of credentials in code
- Token expiration logic
- Handling concurrent sessions
