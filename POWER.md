---
name: "weather"
version: "1.0.0"
displayName: "Weather"
description: "Get weather forecasts and current conditions for any location"
keywords: ["weather", "forecast", "temperature", "conditions", "alerts", "air quality"]
---

# Weather Power

## Overview
The Weather power provides comprehensive weather data for any location worldwide using free APIs from NOAA (US) and Open-Meteo (global). This power gives you access to 12 different weather-related tools covering forecasts, current conditions, historical data, alerts, air quality, marine conditions, and more.

**No API keys required** - all weather data is freely available.

## Available Steering Files

This power uses a single steering file:
- **steering.md** - Complete guide to all 12 weather tools with examples

## Available MCP Tools

### Core Weather Tools

1. **get_forecast** - Weather forecasts for any location (up to 16 days, worldwide)
   - Required: `latitude`, `longitude`
   - Optional: `days` (1-16, default: 7)

2. **get_current_conditions** - Real-time weather observations (US only)
   - Required: `latitude`, `longitude`

3. **search_location** - Find coordinates for location names
   - Required: `query` (location name)

4. **get_alerts** - Weather watches, warnings, advisories (US only)
   - Required: `latitude`, `longitude`

### Advanced Weather Tools

5. **get_historical_weather** - Historical weather data (1940-present)
   - Required: `latitude`, `longitude`, `start_date` (YYYY-MM-DD), `end_date` (YYYY-MM-DD)

6. **get_air_quality** - Air quality monitoring with AQI and pollutant data
   - Required: `latitude`, `longitude`

7. **get_marine_conditions** - Wave, swell, and ocean current data
   - Required: `latitude`, `longitude`

8. **get_weather_imagery** - Radar images and precipitation maps
   - Required: `latitude`, `longitude`

9. **get_lightning_activity** - Real-time lightning strike detection
   - Required: `latitude`, `longitude`

10. **get_river_conditions** - River levels and flood monitoring
    - Required: `latitude`, `longitude`

11. **get_wildfire_info** - Active wildfire tracking
    - Required: `latitude`, `longitude`

12. **check_service_status** - API health checks and cache statistics
    - No parameters

## Tool Usage Examples

### Using Individual Tools

**Get a weather forecast:**
```javascript
// First get coordinates
usePower("weather", "search_location", {
  "query": "Seattle, WA"
})
// Returns: { latitude: 47.6062, longitude: -122.3321 }

// Then get forecast
usePower("weather", "get_forecast", {
  "latitude": 47.6062,
  "longitude": -122.3321,
  "days": 7
})
```

**Check air quality:**
```javascript
usePower("weather", "get_air_quality", {
  "latitude": 34.0522,
  "longitude": -118.2437  // Los Angeles
})
```

**View historical weather:**
```javascript
usePower("weather", "get_historical_weather", {
  "latitude": 40.7128,
  "longitude": -74.0060,  // New York
  "start_date": "2024-01-01",
  "end_date": "2024-01-31"
})
```

## Combining Tools (Workflows)

### Complete Weather Report Workflow
```javascript
// 1. Find location coordinates
const location = usePower("weather", "search_location", {
  "query": "Portland, OR"
});

// 2. Get multiple weather aspects
const forecast = usePower("weather", "get_forecast", {
  "latitude": location.latitude,
  "longitude": location.longitude,
  "days": 7
});

const alerts = usePower("weather", "get_alerts", {
  "latitude": location.latitude,
  "longitude": location.longitude
});

const airQuality = usePower("weather", "get_air_quality", {
  "latitude": location.latitude,
  "longitude": location.longitude
});

// Present comprehensive report
```

### Trip Planning Workflow
```javascript
// 1. Check forecast for destination
usePower("weather", "get_forecast", {
  "latitude": 36.1699,
  "longitude": -115.1398,  // Las Vegas
  "days": 14
})

// 2. Check for any weather alerts
usePower("weather", "get_alerts", {
  "latitude": 36.1699,
  "longitude": -115.1398
})
```

### Historical Analysis Workflow
```javascript
// Compare weather patterns across years
usePower("weather", "get_historical_weather", {
  "latitude": 41.8781,
  "longitude": -87.6298,  // Chicago
  "start_date": "2023-12-01",
  "end_date": "2023-12-31"
})

usePower("weather", "get_historical_weather", {
  "latitude": 41.8781,
  "longitude": -87.6298,
  "start_date": "2024-12-01",
  "end_date": "2024-12-31"
})
```

## Best Practices

### ✅ Do:
- **Always get coordinates first** using `search_location` before calling other tools
- Use specific location names (city + state/country) for accurate searches
- Check for weather alerts when severe weather is possible
- Use appropriate day ranges (1-3 days for detailed, 7-14 for planning)
- Combine forecast + alerts for comprehensive weather reports
- Check air quality when discussing outdoor activities

### ❌ Don't:
- Don't pass city names directly to tools (except search_location) - use coordinates
- Don't request more than 16 days of forecast (API limit)
- Don't expect real-time data for historical queries (1940-yesterday only)
- Don't use US-only tools (current_conditions, alerts) for international locations

## Common Errors & Solutions

**Error: "Invalid latitude: must be a finite number"**
- Solution: Use `search_location` first to get coordinates, then pass them to other tools

**Error: "Tool 'get-forecast' is not enabled"**
- Solution: Tool name uses underscore `get_forecast` not hyphen `get-forecast`

**Error: "No data available for this location"**
- Solution: Check if using US-only tool (alerts, current_conditions) for international location

## Configuration

**Environment Variables:**
- `ENABLED_TOOLS` - Controls which tools are available (default: "basic")
  - `basic` - Essential 5 tools
  - `standard` - Basic + historical
  - `full` - Standard + air quality
  - `all` - All 12 tools (recommended)

**Current Config:** All tools enabled (`ENABLED_TOOLS=all`)

## Integration with Other Powers

### Weather + Calendar Planning
Combine with calendar power to suggest outdoor activity timing based on weather

### Weather + Travel Planning
Check destination weather when planning trips

### Weather + Image Generation
Generate weather-appropriate scene images

---

**Data Sources:** NOAA National Weather Service (US), Open-Meteo (Worldwide)
**API Keys:** None required
**Rate Limits:** Free tier limits apply
