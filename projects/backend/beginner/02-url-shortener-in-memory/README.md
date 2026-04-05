# URL Shortener (in-memory storage)

## Idea
Create a service that converts long URLs into short, unique identifiers and redirects users to the original URL. Data is stored in memory, so it resets on restart. This teaches URL encoding, hash generation, and basic routing.

## Learning Objectives
- Understand URL structure and encoding
- Implement hash/ID generation algorithms
- Handle URL validation and parsing
- Build redirect functionality
- Work with in-memory key-value stores

## Implementation Tips
- Use a simple hash function or counter-based ID generation
- Validate input URLs before shortening
- Store original URL with short code in a dictionary/map
- Implement redirect endpoint (301/302 HTTP status)
- Add analytics: track how many times each URL was accessed
- Consider edge cases: duplicate URLs, malformed URLs
- Add expiration logic (optional) for shortened URLs
- Implement a retrieve endpoint to get original URL
- Add optional custom short codes
- Prevent collision handling for generated codes

## Key Challenges
- Generating collision-free short codes
- URL validation and sanitization
- Redirect logic and status codes
- Tracking usage statistics
- Handling URL encoding edge cases
