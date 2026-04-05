# Weather Proxy API (consume external API)

## Idea
Create an API that acts as a proxy to an external weather service. Learn HTTP client patterns, API integration, error handling, and caching. This teaches how to consume third-party APIs and expose their data through your own interface.

## Learning Objectives
- Make HTTP requests to external APIs
- Handle API responses and error scenarios
- Parse and transform data
- Implement caching to reduce external calls
- Handle rate limiting and timeouts
- Work with authentication tokens/keys

## Implementation Tips
- Use free weather API (OpenWeatherMap, WeatherAPI, etc.)
- Implement weather endpoint: GET /weather?city=<city_name>
- Handle API key management securely
- Add simple caching to reduce API calls
- Parse external API response and return simplified JSON
- Handle missing cities and API errors gracefully
- Add timeout handling for slow external services
- Implement retry logic with exponential backoff
- Add logging for API calls and errors
- Consider rate limiting from external API
- Add forecast endpoint for multi-day predictions
- Validate and sanitize city name input

## Key Challenges
- External API dependency and reliability
- Error propagation and user-friendly messages
- Caching strategy and invalidation
- API key security
- Rate limiting
- Timeout and connection issues
