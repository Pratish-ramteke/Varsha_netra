# VarshaNetra — Backend API Contract

This is the contract the React frontend is built against. The real FastAPI
backend must match these shapes exactly so the frontend requires **zero**
changes when `VITE_USE_MOCK_API` is switched from `true` to `false`.

Base URL: configured via `VITE_API_BASE_URL` (default `http://localhost:8000/api`).

---

## Auth

### `POST /api/auth/send-otp`
Request:
```json
{ "phoneNumber": "+919876543210" }
```
Response:
```json
{ "requestId": "abc123", "expiresInSeconds": 300 }
```

### `POST /api/auth/verify-otp`
Request:
```json
{ "requestId": "abc123", "otp": "123456" }
```
Response:
```json
{ "token": "jwt...", "user": { "id": "u1", "phoneNumber": "+919876543210" } }
```

---

## Dashboard

### `GET /api/dashboard`
```json
{
  "city": "Nagpur",
  "state": "Maharashtra",
  "updatedAt": "2026-08-17T10:00:00Z",
  "kpis": {
    "overallCityRisk": "HIGH",
    "currentRainfall": 78,
    "rainProbability": 82,
    "criticalZoneCount": 4
  }
}
```

---

## Zones

### `GET /api/zones`
```json
[
  {
    "id": "khamla",
    "name": "Khamla",
    "latitude": 21.1245,
    "longitude": 79.0378,
    "riskLevel": "HIGH",
    "floodProbability": 78,
    "rainfall": 78,
    "predictedRainfall": 92,
    "waterLevel": 65,
    "estimatedImpact": "40-60 cm waterlogging expected on 3 roads"
  }
]
```
`boundary` (GeoJSON Polygon/MultiPolygon) may be added later for polygon rendering.

### `GET /api/zones/{zoneId}`
```json
{
  "areaId": "khamla",
  "areaName": "Khamla",
  "floodProbability": 78,
  "riskLevel": "HIGH",
  "rainfall": 78,
  "predictedRainfall": 92,
  "waterLevel": 65,
  "riskWindow": { "start": "19:30", "end": "21:15" },
  "factors": [
    { "name": "Heavy Rainfall", "status": "HIGH", "explanation": "..." }
  ],
  "expectedImpact": {
    "waterDepth": "40-60 cm",
    "affectedRoads": 3,
    "estimatedDisruption": "2-3 hours"
  }
}
```

### `GET /api/zones/{zoneId}/traffic`
```json
{
  "zoneId": "khamla",
  "affectedRoads": [
    { "name": "Khamla Square Main Road", "waterDepth": "40-60 cm", "estimatedClosureMinutes": 150 }
  ],
  "bypassRoutes": [
    { "name": "Via Hingna Road", "description": "...", "addedTravelMinutes": 15 }
  ],
  "estimatedClosureMinutes": 150,
  "estimatedRecoveryTime": "22:30"
}
```

---

## Forecast

### `GET /api/forecast`
```json
{
  "currentRainfall": 78,
  "rainProbability": 82,
  "peakRainfall": 100,
  "floodRisk": "HIGH",
  "hourly": [{ "time": "18:00", "rainfall": 60, "riskLevel": "MODERATE" }],
  "riskTimeline": [{ "time": "18:00", "riskLevel": "MODERATE" }],
  "peakRiskWindow": {
    "start": "7:30 PM", "end": "9:00 PM", "peakTime": "8:15 PM", "expectedAffectedZones": 4
  },
  "recommendedPreparation": ["Prepare response teams"]
}
```

---

## Alerts

### `GET /api/alerts`
```json
[
  {
    "id": "alert-001",
    "zoneId": "khamla",
    "zoneName": "Khamla",
    "severity": "CRITICAL",
    "message": "...",
    "createdAt": "2026-08-17T09:48:00Z",
    "recommendedActions": ["Divert traffic", "Alert residents"]
  }
]
```

---

## Scenario Simulation

### `POST /api/scenarios/simulate`
Request:
```json
{ "rainfall": 130 }
```
Response:
```json
{
  "rainfallSimulated": 130,
  "overallRisk": "CRITICAL",
  "criticalZones": 8,
  "highRiskZones": 11,
  "changedZones": [
    { "zoneId": "khamla", "zoneName": "Khamla", "previousRisk": "HIGH", "newRisk": "CRITICAL" }
  ]
}
```

---

## AI Recommendation

### `POST /api/ai/recommendation`
Request:
```json
{
  "areaId": "khamla",
  "riskLevel": "HIGH",
  "floodProbability": 78,
  "rainfall": 92,
  "waterLevel": 65,
  "riskFactors": ["heavy rainfall", "low elevation", "limited drainage"]
}
```
Response:
```json
{
  "summary": "...",
  "why": ["..."],
  "recommendations": ["Monitor water level", "Divert traffic if threshold is exceeded"],
  "priority": "HIGH"
}
```
The LLM must never receive raw sensor/DB access — only the ML engine's
already-computed risk output shown above. No LLM/weather API keys are ever
present in frontend code.

---

## ML Risk Engine Contract (internal, backend ⇄ ML service)

Input:
```json
{
  "rainfall": 100,
  "forecastRainfall": 130,
  "elevation": 312,
  "drainageCapacity": 0.45,
  "historicalFloodCount": 8,
  "waterLevel": 65
}
```
Output:
```json
{
  "floodProbability": 0.82,
  "riskLevel": "CRITICAL",
  "riskFactors": ["high rainfall", "low elevation", "limited drainage"]
}
```

---

## Health

### `GET /api/health`
```json
{ "status": "ok" }
```
