# Weather App

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a small app where a user types a city, and you fetch and display its current weather from a public API. This is the beginner's first real taste of the async web: a request goes out, time passes, and something — data, an error, or nothing — comes back. The interesting work is not the layout but handling the three states every network UI must show: loading, success, and failure. You will read JSON, map it onto a clean view, and make sure the user is never staring at a frozen screen wondering what happened.

## Prerequisites

- JavaScript basics including functions and objects
- Promises and `async`/`await`, or `.then()` chains
- How to read API documentation and inspect a JSON response
- A free API key from a weather provider (e.g. OpenWeather or Open-Meteo, which needs none)

## Learning Objectives

By the end, you should be able to:

- Make HTTP requests with `fetch` and parse a JSON response
- Model and render explicit loading, success, and error states
- Handle failures gracefully: bad city, network error, non-200 responses
- Keep an API key out of source control and understand its limits
- Transform raw API data into a small, UI-friendly shape

## Functional Requirements

1. The user can search for a city by name and trigger a weather lookup.
2. While the request is in flight, a loading indicator is shown.
3. On success, the UI shows temperature, a condition description, and humidity.
4. An unknown city or failed request shows a clear, human-readable error message.
5. Rapid repeated submissions must not leave a stale result on screen.
6. The search input is keyboard-accessible and submittable via Enter.
7. Temperature units are labelled explicitly (°C or °F).

## Suggested Milestones

1. **Milestone 1 — Fetch & display:** Call the API for a hardcoded city and render the result.
2. **Milestone 2 — Search & states:** Wire up the input, add loading and error handling.
3. **Milestone 3 — Robustness:** Handle empty input, bad cities, and out-of-order responses.

## Data & Interface Sketch

```text
View state (one of)
  { status: "idle" }
  { status: "loading" }
  { status: "success", data: Weather }
  { status: "error", message: string }

Weather (mapped from API JSON)
  city:        string
  tempC:       number
  condition:   string   ("Clouds", "Rain", ...)
  humidity:    number   (percent)
  icon:        string   (code -> your own icon)

Layout
+--------------------------------------+
| [ search city......... ] [ Search ]  |
+--------------------------------------+
|  London           [ loading... ]     |
|  18°C  Cloudy   Humidity 72%         |
+--------------------------------------+
```

## Stretch Goals

- Add a multi-day forecast below the current conditions.
- Toggle between Celsius and Fahrenheit without re-fetching.
- Use the Geolocation API to load weather for the user's current location.
- Cache the last successful result so a reload shows something instantly.

## Definition of Done

- [ ] A valid city shows real weather data with a labelled unit.
- [ ] Loading, success, and error states each render distinctly.
- [ ] A nonexistent city produces a friendly error, not a blank screen or crash.
- [ ] The API key is not committed to the repository.
- [ ] Submitting with Enter and clicking Search behave identically.

## Common Pitfalls

- Assuming `fetch` rejects on HTTP errors — it only rejects on network failure; check `response.ok`.
- Forgetting to handle the loading state, so the UI looks broken during the request.
- Race conditions where a slow earlier request overwrites a newer result.
- Hardcoding and committing your API key, then hitting rate limits or leaking it.
- Rendering the raw API JSON shape directly, coupling your UI to the provider.

## Resources

- [MDN: Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — requests, responses, and error checking.
- [MDN: Response.ok](https://developer.mozilla.org/en-US/docs/Web/API/Response/ok) — why you must check status yourself.
- [Open-Meteo API](https://open-meteo.com/en/docs) — a free weather API with no key required.
- [web.dev: Loading states and skeletons](https://web.dev/articles/optimize-cls) — communicating progress to users.
