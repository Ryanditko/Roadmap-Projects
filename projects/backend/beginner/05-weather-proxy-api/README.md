# Weather Proxy API

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 5–8 hours

## Overview

Build an API that sits in front of a public weather service, forwarding requests, simplifying the response, and caching results so you do not hammer the upstream provider. This is your first taste of being an HTTP *client* as well as a server — the pattern behind every backend-for-frontend and API gateway. A user asks your service for the weather in a city; your service asks the provider, trims the payload to what matters, and answers, ideally from cache the second time.

## Prerequisites

- Ability to build HTTP endpoints and return JSON ([Static JSON API Server](../09-static-json-api-server/) is a lighter precursor)
- Understanding of environment variables for secrets
- A free API key from a weather provider (OpenWeatherMap, WeatherAPI, or Open-Meteo which needs none)
- Familiarity with making outbound HTTP requests in your language

## Learning Objectives

By the end, you should be able to:

- Make outbound HTTP requests from a server and handle their responses
- Keep an API key out of source code using configuration/environment variables
- Transform a verbose upstream payload into a clean, minimal response
- Cache responses with a time-to-live to cut latency and upstream calls
- Handle upstream failures — timeouts, 404s, rate limits — without exposing them raw
- Translate upstream error conditions into sensible status codes for your clients

## Functional Requirements

1. The system must expose an endpoint that accepts a city name and returns current weather.
2. The system must read the upstream API key from configuration, never hardcoded.
3. The system must transform the upstream response into a simplified, documented shape.
4. The system must cache each city's result for a configurable TTL and serve cache hits without calling upstream.
5. An unknown city must return 404, distinct from an upstream outage.
6. The system must apply a timeout to the upstream call and degrade gracefully if it is exceeded.
7. The system must never leak the API key in responses, logs, or error messages.

## Suggested Milestones

1. **Milestone 1 — Passthrough:** Call the upstream provider for a city and return its raw response.
2. **Milestone 2 — Transform & harden:** Map the payload to your own shape and handle unknown-city and timeout cases.
3. **Milestone 3 — Cache:** Add in-memory caching with a TTL and confirm cache hits skip the upstream call.

## Data & Interface Sketch

```text
GET /weather?city=Lisbon
  -> 200 {
       "city": "Lisbon",
       "tempC": 21.4,
       "condition": "Clear",
       "cachedAt": "ISO-8601",
       "source": "cache" | "upstream"
     }
  -> 404  { "error": "city not found" }
  -> 502  { "error": "weather provider unavailable" }
  -> 504  { "error": "weather provider timed out" }

Cache entry: key = normalized city, value = { payload, expiresAt }
```

## Stretch Goals

- Add a multi-day forecast endpoint alongside current conditions.
- Add retry with exponential backoff for transient upstream errors.
- Support units via a `?units=metric|imperial` parameter.
- Expose cache metrics (hit/miss counts) on a debug endpoint.

## Definition of Done

- [ ] A valid city returns simplified weather data with the correct fields.
- [ ] The API key is loaded from configuration and never appears in any output.
- [ ] A repeated request within the TTL is served from cache and makes no upstream call (verifiable).
- [ ] Unknown cities return 404 and upstream failures return 502/504, not 500.
- [ ] An upstream timeout does not hang your endpoint indefinitely.

## Common Pitfalls

- Committing the API key or echoing it back in an error — treat it as a secret from day one.
- Passing the upstream's raw error body straight through, coupling your clients to a third party.
- Caching by the raw (unnormalized) city string, so "Lisbon" and "lisbon" miss each other.
- No timeout on the outbound call, letting a slow provider freeze your whole service.

## Resources

- [Open-Meteo API](https://open-meteo.com/en/docs) — a free weather API that needs no key, great for starting.
- [OpenWeatherMap API docs](https://openweathermap.org/api) — a widely used provider with a free tier.
- [MDN: Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) — HTTP caching concepts, useful even for in-memory TTLs.
- [The Twelve-Factor App: Config](https://12factor.net/config) — why secrets belong in the environment, not the code.
