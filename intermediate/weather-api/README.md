# Weather API Integration

<p align="center">
  <img src="https://images.unsplash.com/photo-1592210454359-9043f067919b?w=600&q=80" alt="Weather API" width="600">
</p>

**Level:** Intermediate | **Time:** 1-2 weeks | **Prerequisites:** JavaScript/Node.js, REST APIs, Async programming

## Description

Build a comprehensive weather application that integrates with multiple weather APIs, displays current conditions, forecasts, weather maps, and alerts. Learn API integration, data transformation, caching strategies, and creating responsive weather interfaces.

## Core Features

### Must Have
- Current weather conditions by location
- 5-day weather forecast
- Location search (city, ZIP code, coordinates)
- Temperature unit conversion (C/F)
- Weather icons and conditions display
- Geolocation support
- Error handling for API failures
- Loading states

### Nice to Have
- Hourly forecast (24-48 hours)
- Weather alerts and warnings
- Air quality index (AQI)
- UV index and sun times
- Weather maps (radar, temperature, precipitation)
- Multiple location favorites
- Historical weather data
- Weather comparison between cities
- Severe weather notifications
- Animated weather backgrounds

## Learning Objectives

- **API Integration**: Work with RESTful APIs and handle responses
- **Async JavaScript**: Master Promises, async/await, error handling
- **Data Caching**: Implement caching to reduce API calls
- **Geolocation API**: Access user location
- **Data Visualization**: Display weather data effectively
- **Rate Limiting**: Handle API request limits

## Implementation Guide

### Step 1: Choose a Weather API

**Recommended APIs:**
- **OpenWeatherMap** (Free tier: 1000 calls/day)
- **WeatherAPI** (Free tier: 1M calls/month)
- **AccuWeather** (Limited free tier)
- **Tomorrow.io** (Weather data + forecasts)

Register at [OpenWeatherMap](https://openweathermap.org/api) and get your API key.

### Step 2: HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather App</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>🌤️ Weather App</h1>
        </header>

        <div class="search-section">
            <input 
                type="text" 
                id="searchInput" 
                placeholder="Search city..."
                autocomplete="off"
            >
            <button id="searchBtn">Search</button>
            <button id="locationBtn" title="Use my location">📍</button>
        </div>

        <div id="loading" class="loading hidden">
            <div class="spinner"></div>
            <p>Loading weather data...</p>
        </div>

        <div id="error" class="error hidden"></div>

        <div id="weatherDisplay" class="weather-display hidden">
            <!-- Current Weather -->
            <div class="current-weather">
                <div class="location">
                    <h2 id="cityName">--</h2>
                    <p id="dateTime">--</p>
                </div>
                
                <div class="weather-main">
                    <img id="weatherIcon" src="" alt="Weather icon">
                    <div class="temperature">
                        <span id="temp" class="temp-value">--</span>
                        <span class="temp-unit">°C</span>
                    </div>
                    <p id="description" class="description">--</p>
                </div>

                <div class="weather-details">
                    <div class="detail-item">
                        <span class="label">Feels Like</span>
                        <span id="feelsLike">--°C</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Humidity</span>
                        <span id="humidity">--%</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Wind Speed</span>
                        <span id="windSpeed">-- km/h</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Pressure</span>
                        <span id="pressure">-- hPa</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Visibility</span>
                        <span id="visibility">-- km</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">UV Index</span>
                        <span id="uvIndex">--</span>
                    </div>
                </div>
            </div>

            <!-- 5-Day Forecast -->
            <div class="forecast-section">
                <h3>5-Day Forecast</h3>
                <div id="forecastContainer" class="forecast-container"></div>
            </div>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
```

### Step 3: CSS Styling

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
}

.container {
    max-width: 900px;
    margin: 0 auto;
}

header {
    text-align: center;
    color: white;
    margin-bottom: 30px;
}

header h1 {
    font-size: 2.5rem;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.search-section {
    display: flex;
    gap: 10px;
    margin-bottom: 30px;
}

#searchInput {
    flex: 1;
    padding: 15px;
    border: none;
    border-radius: 10px;
    font-size: 16px;
}

#searchBtn,
#locationBtn {
    padding: 15px 25px;
    border: none;
    border-radius: 10px;
    background: white;
    color: #667eea;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

#searchBtn:hover,
#locationBtn:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0,0,0,0.2);
}

.loading,
.error {
    background: white;
    padding: 30px;
    border-radius: 15px;
    text-align: center;
}

.hidden {
    display: none;
}

.spinner {
    border: 4px solid #f3f3f3;
    border-top: 4px solid #667eea;
    border-radius: 50%;
    width: 50px;
    height: 50px;
    animation: spin 1s linear infinite;
    margin: 0 auto 20px;
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

.weather-display {
    background: white;
    border-radius: 20px;
    padding: 30px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}

.current-weather {
    margin-bottom: 30px;
}

.location h2 {
    font-size: 2rem;
    color: #333;
}

.location p {
    color: #666;
    margin-top: 5px;
}

.weather-main {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 20px;
    margin: 30px 0;
}

#weatherIcon {
    width: 120px;
    height: 120px;
}

.temperature {
    font-size: 4rem;
    font-weight: bold;
    color: #667eea;
}

.description {
    font-size: 1.5rem;
    color: #666;
    text-transform: capitalize;
}

.weather-details {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 20px;
    margin-top: 30px;
}

.detail-item {
    background: #f8f9fa;
    padding: 15px;
    border-radius: 10px;
    text-align: center;
}

.detail-item .label {
    display: block;
    color: #666;
    font-size: 0.9rem;
    margin-bottom: 5px;
}

.detail-item span:last-child {
    font-size: 1.3rem;
    font-weight: 600;
    color: #333;
}

.forecast-section h3 {
    color: #333;
    margin-bottom: 20px;
}

.forecast-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 15px;
}

.forecast-item {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    padding: 20px;
    border-radius: 15px;
    text-align: center;
}

.forecast-item .day {
    font-weight: 600;
    margin-bottom: 10px;
}

.forecast-item img {
    width: 60px;
    height: 60px;
}

.forecast-item .temp-range {
    margin-top: 10px;
    font-size: 1.1rem;
}

@media (max-width: 768px) {
    .search-section {
        flex-direction: column;
    }
    
    .weather-main {
        flex-direction: column;
    }
    
    .temperature {
        font-size: 3rem;
    }
}
```

### Step 4: JavaScript Implementation

```javascript
class WeatherApp {
    constructor() {
        this.API_KEY = 'YOUR_OPENWEATHERMAP_API_KEY'; // Replace with your API key
        this.BASE_URL = 'https://api.openweathermap.org/data/2.5';
        this.cache = {};
        this.CACHE_DURATION = 10 * 60 * 1000; // 10 minutes
        
        this.initializeElements();
        this.attachEventListeners();
        this.loadLastSearch();
    }

    initializeElements() {
        this.searchInput = document.getElementById('searchInput');
        this.searchBtn = document.getElementById('searchBtn');
        this.locationBtn = document.getElementById('locationBtn');
        this.loading = document.getElementById('loading');
        this.error = document.getElementById('error');
        this.weatherDisplay = document.getElementById('weatherDisplay');
    }

    attachEventListeners() {
        this.searchBtn.addEventListener('click', () => this.searchByCity());
        this.searchInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') this.searchByCity();
        });
        this.locationBtn.addEventListener('click', () => this.getCurrentLocation());
    }

    async searchByCity() {
        const city = this.searchInput.value.trim();
        if (!city) {
            this.showError('Please enter a city name');
            return;
        }

        try {
            this.showLoading();
            const data = await this.fetchWeatherByCity(city);
            this.displayWeather(data);
            localStorage.setItem('lastSearch', city);
        } catch (error) {
            this.showError(error.message);
        }
    }

    getCurrentLocation() {
        if (!navigator.geolocation) {
            this.showError('Geolocation is not supported by your browser');
            return;
        }

        this.showLoading();
        navigator.geolocation.getCurrentPosition(
            async (position) => {
                try {
                    const { latitude, longitude } = position.coords;
                    const data = await this.fetchWeatherByCoords(latitude, longitude);
                    this.displayWeather(data);
                } catch (error) {
                    this.showError(error.message);
                }
            },
            (error) => {
                this.showError('Unable to retrieve your location');
            }
        );
    }

    async fetchWeatherByCity(city) {
        const cacheKey = `city_${city}`;
        if (this.isCacheValid(cacheKey)) {
            return this.cache[cacheKey].data;
        }

        const url = `${this.BASE_URL}/weather?q=${city}&appid=${this.API_KEY}&units=metric`;
        const response = await fetch(url);
        
        if (!response.ok) {
            throw new Error('City not found');
        }

        const data = await response.json();
        
        // Fetch forecast
        const forecastUrl = `${this.BASE_URL}/forecast?q=${city}&appid=${this.API_KEY}&units=metric`;
        const forecastResponse = await fetch(forecastUrl);
        const forecastData = await forecastResponse.json();
        
        const result = { ...data, forecast: forecastData };
        this.updateCache(cacheKey, result);
        
        return result;
    }

    async fetchWeatherByCoords(lat, lon) {
        const cacheKey = `coords_${lat}_${lon}`;
        if (this.isCacheValid(cacheKey)) {
            return this.cache[cacheKey].data;
        }

        const url = `${this.BASE_URL}/weather?lat=${lat}&lon=${lon}&appid=${this.API_KEY}&units=metric`;
        const response = await fetch(url);
        
        if (!response.ok) {
            throw new Error('Unable to fetch weather data');
        }

        const data = await response.json();
        
        // Fetch forecast
        const forecastUrl = `${this.BASE_URL}/forecast?lat=${lat}&lon=${lon}&appid=${this.API_KEY}&units=metric`;
        const forecastResponse = await fetch(forecastUrl);
        const forecastData = await forecastResponse.json();
        
        const result = { ...data, forecast: forecastData };
        this.updateCache(cacheKey, result);
        
        return result;
    }

    displayWeather(data) {
        // Update current weather
        document.getElementById('cityName').textContent = 
            `${data.name}, ${data.sys.country}`;
        document.getElementById('dateTime').textContent = 
            new Date().toLocaleDateString('en-US', { 
                weekday: 'long', 
                year: 'numeric', 
                month: 'long', 
                day: 'numeric' 
            });
        
        const iconCode = data.weather[0].icon;
        document.getElementById('weatherIcon').src = 
            `https://openweathermap.org/img/wn/${iconCode}@4x.png`;
        document.getElementById('temp').textContent = 
            Math.round(data.main.temp);
        document.getElementById('description').textContent = 
            data.weather[0].description;
        
        // Update details
        document.getElementById('feelsLike').textContent = 
            `${Math.round(data.main.feels_like)}°C`;
        document.getElementById('humidity').textContent = 
            `${data.main.humidity}%`;
        document.getElementById('windSpeed').textContent = 
            `${Math.round(data.wind.speed * 3.6)} km/h`;
        document.getElementById('pressure').textContent = 
            `${data.main.pressure} hPa`;
        document.getElementById('visibility').textContent = 
            `${(data.visibility / 1000).toFixed(1)} km`;
        document.getElementById('uvIndex').textContent = 
            data.uvi || 'N/A';
        
        // Update forecast
        this.displayForecast(data.forecast);
        
        this.hideLoading();
        this.weatherDisplay.classList.remove('hidden');
    }

    displayForecast(forecastData) {
        const container = document.getElementById('forecastContainer');
        container.innerHTML = '';
        
        // Get one forecast per day (at noon)
        const dailyForecasts = forecastData.list.filter(item => 
            item.dt_txt.includes('12:00:00')
        ).slice(0, 5);
        
        dailyForecasts.forEach(day => {
            const date = new Date(day.dt * 1000);
            const dayName = date.toLocaleDateString('en-US', { weekday: 'short' });
            
            const forecastItem = document.createElement('div');
            forecastItem.className = 'forecast-item';
            forecastItem.innerHTML = `
                <div class="day">${dayName}</div>
                <img src="https://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png" 
                     alt="${day.weather[0].description}">
                <div class="temp-range">
                    ${Math.round(day.main.temp_max)}° / ${Math.round(day.main.temp_min)}°
                </div>
            `;
            container.appendChild(forecastItem);
        });
    }

    isCacheValid(key) {
        if (!this.cache[key]) return false;
        const age = Date.now() - this.cache[key].timestamp;
        return age < this.CACHE_DURATION;
    }

    updateCache(key, data) {
        this.cache[key] = {
            data,
            timestamp: Date.now()
        };
    }

    showLoading() {
        this.loading.classList.remove('hidden');
        this.error.classList.add('hidden');
        this.weatherDisplay.classList.add('hidden');
    }

    hideLoading() {
        this.loading.classList.add('hidden');
    }

    showError(message) {
        this.hideLoading();
        this.weatherDisplay.classList.add('hidden');
        this.error.textContent = message;
        this.error.classList.remove('hidden');
    }

    loadLastSearch() {
        const lastSearch = localStorage.getItem('lastSearch');
        if (lastSearch) {
            this.searchInput.value = lastSearch;
            this.searchByCity();
        }
    }
}

// Initialize the app
const app = new WeatherApp();
```

## Technology Stack
- **Frontend:** HTML5, CSS3, Vanilla JavaScript
- **API:** OpenWeatherMap API (or WeatherAPI, AccuWeather)
- **Features:** Fetch API, Geolocation API, localStorage
- **Optional:** Node.js/Express for backend proxy

## Testing Checklist
- [ ] Search by city name works
- [ ] Current location button functions
- [ ] Weather data displays correctly
- [ ] 5-day forecast shows
- [ ] Icons load properly
- [ ] Error handling for invalid cities
- [ ] Loading states display
- [ ] Cache reduces API calls
- [ ] Responsive on mobile devices
- [ ] Last search persists

## Additional Challenges

### Easy
- Add temperature unit toggle (C/F)
- Include weather alerts
- Add multiple city comparison

### Medium
- Implement weather maps (radar, satellite)
- Add hourly forecast (24 hours)
- Include air quality index (AQI)
- Create favorite locations list

### Hard
- Build weather trends/history charts
- Implement severe weather notifications
- Add weather-based recommendations
- Create animated weather backgrounds
- Build PWA with offline support

## Resources
- [OpenWeatherMap API Docs](https://openweathermap.org/api)
- [Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API)
- [Fetch API Guide](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
