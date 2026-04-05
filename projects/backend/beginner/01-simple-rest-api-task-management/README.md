# Simple REST API for Task Management (no database)

## Idea
Build a basic REST API that manages tasks using only in-memory storage. This project teaches fundamental HTTP concepts, routing, and request/response handling without the complexity of a database layer.

## Learning Objectives
- Understand REST principles and HTTP methods (GET, POST, PUT, DELETE)
- Implement basic CRUD operations
- Handle request/response serialization
- Work with status codes and error handling
- Use in-memory data structures (arrays/objects)

## Implementation Tips
- Choose a lightweight framework (Express.js, Flask, FastAPI, etc.)
- Start with a simple data structure to store tasks
- Implement endpoints: GET all tasks, GET task by ID, POST new task, PUT update task, DELETE task
- Use appropriate HTTP status codes (200, 201, 400, 404, 500)
- Add basic request validation
- Consider how data persists (or doesn't) during runtime
- Test with tools like Postman or curl
- Add error handling for edge cases (duplicate IDs, invalid input)
- Document API endpoints with descriptions

## Key Challenges
- Handling concurrent requests
- Validating input data formats
- Generating unique task IDs
- Managing state in memory
- Understanding when data is lost
