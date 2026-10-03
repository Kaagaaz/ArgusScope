## Geospatial Intelligence & OSINT Mapping Platform

ArgusScope is a web-based geospatial intelligence platform for visualizing publicly available network and geolocation data through an interactive map interface.

It combines IP geolocation, Wi-Fi BSSID lookup, multi-BSSID location fusion, browser GPS, and multiple map layers.

---

<img width="3200" height="1732" alt="1000067698" src="https://github.com/user-attachments/assets/c5830b74-3127-4aa1-8791-c679e9dc0e07" />






 ---
## Architecture

```text
User
 │
 ▼
GitHub Pages
 │
 │ HTTPS / JSON
 ▼
Render
 │
 └── FastAPI Backend
      ├── IP Geolocation
      │    ├── ipapi.co
      │    └── ip-api.com
      │
      ├── BSSID Lookup
      │    └── WiGLE
      │
      └── BSSID Cluster Fusion
             │
             ▼
        Geographic data
             │
             ▼
        Leaflet Frontend
```

The frontend and backend are deployed separately:

- **Frontend:** GitHub Pages
- **Backend:** Render
- **Backend framework:** FastAPI
- **Map engine:** Leaflet
- **Street data:** OpenStreetMap
- **Wireless database:** WiGLE

---

# Features

## IP Geolocation

ArgusScope accepts a public IPv4 or IPv6 address and requests approximate geographic information from external IP geolocation providers.

Possible fields include:

- IP address
- Latitude
- Longitude
- City
- Region
- Country
- Country code
- Postal code
- Timezone
- ISP
- Organization
- ASN
- Estimated accuracy
- Provider source

### Flow

```text
IP
 │
 ▼
FastAPI
 │
 ├── ipapi.co
 │
 └── ip-api.com
 │
 ▼
Coordinates
 │
 ▼
Leaflet map
```

**Important:** IP geolocation is not GPS tracking. An IP normally represents network or ISP infrastructure and can resolve only approximately.

---

# BSSID / Wi-Fi Location

ArgusScope can query a Wi-Fi BSSID through the backend.

Example format:

```text
AA:BB:CC:DD:EE:FF
```

Flow:

```text
Frontend
 │
 ▼
FastAPI
 │
 ▼
WiGLE
 │
 ▼
Coordinates
 │
 ▼
Map
```

Possible results include:

- BSSID
- SSID when available
- Latitude
- Longitude
- Observation count
- Estimated radius
- Confidence
- Source

WiGLE credentials remain server-side and are loaded through environment variables.

---

# Multi-BSSID Cluster Fusion

Multiple BSSID observations can be submitted together.

Example:

```json
{
  "scans": [
    {
      "netid": "AA:BB:CC:DD:EE:01",
      "rssi": -45
    },
    {
      "netid": "AA:BB:CC:DD:EE:02",
      "rssi": -60
    }
  ]
}
```

The current backend uses RSSI as a weighting factor:

```python
weight = max(1, 100 - abs(rssi))
```

The resulting latitude and longitude are calculated using a weighted centroid.

This is an estimation technique, not true radio-frequency triangulation.

---

# Device GPS

The GPS button uses the browser Geolocation API.

```text
Device
 │
 ▼
Browser Geolocation API
 │
 ▼
Latitude / Longitude
 │
 ▼
Leaflet
```

GPS is separate from IP and BSSID lookup and does not need to pass through the backend.

The browser may request location permission.

---

# Mapping System

ArgusScope uses **Leaflet** for:

- Map rendering
- Zooming
- Panning
- Markers
- Accuracy circles
- Popups
- Touch interaction
- Layer switching

## Map Layers

### Street Map

OpenStreetMap is used for detailed geographic mapping.

Depending on coverage, it can show:

- Roads
- Street names
- Localities
- Neighborhoods
- Buildings
- Businesses
- Schools
- Hospitals
- Parks
- Other mapped points of interest

The available detail depends on OpenStreetMap coverage.

### Satellite Layers

ArgusScope can provide satellite imagery through configured third-party imagery layers such as Google or Esri.

Their imagery and usage are subject to the respective providers' terms.

### Cyber Dark

The Cyber Dark map is a darkened street-map presentation. It does not require a separate dark-map API.

---

# Map Target Visualization

Successful lookups can display:

- A target marker
- An estimated accuracy circle
- Location information
- Source information
- Confidence information

The accuracy circle is an application-level estimate and should not be interpreted as a guaranteed physical boundary.

---

# Frontend Architecture

The frontend is a static HTML application using:

```text
HTML5
CSS3
JavaScript
Leaflet
```

Logical structure:

```text
index.html
│
├── Interface
│   ├── Header
│   ├── Search controls
│   ├── Maps menu
│   ├── GPS button
│   └── Result/terminal panel
│
├── Map
│   ├── Street
│   ├── Satellite
│   ├── Esri imagery
│   └── Cyber Dark
│
└── API communication
    ├── locate-ip
    ├── locate-bssid
    └── triangulate-cluster
```

The frontend does not directly expose WiGLE credentials.

---

# Backend Architecture

The backend is written in Python using FastAPI.

```text
FastAPI
│
├── CORS
├── Environment configuration
├── Input validation
├── IP geolocation
│   ├── ipapi.co
│   └── ip-api.com
├── BSSID lookup
│   └── WiGLE
├── BSSID cache
└── Cluster fusion
```

Current backend version:

```text
2.2.0
```

---

# API Endpoints

```text
GET  /
GET  /health
GET  /api/v1/locate-ip
GET  /api/v1/locate-bssid
POST /api/v1/triangulate-cluster
```

## `GET /`

Returns basic service information.

Example:

```json
{
  "status": "online",
  "service": "ArgusScope Intelligence Engine",
  "version": "2.2.0"
}
```

## `GET /health`

Health-check endpoint.

Example:

```json
{
  "status": "healthy",
  "service": "ArgusScope",
  "version": "2.2.0"
}
```

## `GET /api/v1/locate-ip`

Example:

```text
/api/v1/locate-ip?ip=8.8.8.8
```

The backend validates the IP before querying providers.

Private, loopback, link-local, reserved, multicast, and unspecified addresses are rejected.

## `GET /api/v1/locate-bssid`

Example:

```text
/api/v1/locate-bssid?netid=AA:BB:CC:DD:EE:FF
```

The backend normalizes and validates the BSSID before querying WiGLE.

## `POST /api/v1/triangulate-cluster`

Accepts a list of BSSID observations and returns an estimated weighted center.

---

# Provider Fallback

IP lookup uses a fallback architecture:

```text
              IP
               │
               ▼
          Validation
               │
               ▼
           ipapi.co
            /     \
          OK       Fail
          │          │
          ▼          ▼
       Result     ip-api.com
                     /    \
                   OK      Fail
                   │         │
                   ▼         ▼
                Result     Error
```

This allows the backend to continue when the first provider is unavailable.

---

# BSSID Cache

The backend contains an in-memory BSSID cache:

```python
BSSID_CACHE = {}
```

The current TTL is:

```text
600 seconds
```

or 10 minutes.

Flow:

```text
Request
 │
 ▼
Cache?
 ├── Yes → Return cached result
 │
 └── No → Query WiGLE
              │
              ▼
          Store result
              │
              ▼
          Return result
```

Because this cache is in memory, it can disappear when the Render service restarts.

---

# Environment Variables

WiGLE credentials are loaded server-side:

```python
WIGLE_USER = os.getenv("WIGLE_API_USER", "")
WIGLE_TOKEN = os.getenv("WIGLE_API_KEY", "")
```

Example Render environment variables:

```text
WIGLE_API_USER=your_username
WIGLE_API_KEY=your_api_key
```

Never commit API credentials to GitHub or place them in frontend JavaScript.

---

# CORS

The current backend allows cross-origin requests:

```python
allow_origins=["*"]
```

This is convenient during development.

For production, restrict it to the actual frontend origin.

Example:

```python
allow_origins=[
    "https://kaagaaz.github.io"
]
```

---

# Deployment

```text
                   INTERNET
                       │
                       ▼
             ┌─────────────────┐
             │   GitHub Pages  │
             │ ArgusScope UI   │
             └────────┬────────┘
                      │
                      │ HTTPS
                      ▼
             ┌─────────────────┐
             │     Render      │
             │ FastAPI Backend │
             └────────┬────────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       ipapi.co   ip-api.com    WiGLE
```

A typical Render start command is:

```text
uvicorn main:app --host 0.0.0.0 --port $PORT
```

The exact command depends on the backend file location.

---

# Dependencies

Typical backend dependencies:

```text
fastapi
uvicorn
pydantic
requests
```

Example `requirements.txt`:

```text
fastapi
uvicorn
pydantic
requests
```

The frontend does not require Node.js, npm, React, Vite, or another build system if it is maintained as a static HTML application.

---

# Mobile Support

The frontend is designed for:

- Desktop
- Tablet
- Mobile portrait
- Mobile landscape
- Touchscreen devices

On smaller screens, controls can stack while the map remains the primary workspace.

---

# Data Sources

| Source | Purpose |
|---|---|
| ipapi.co | IP geolocation |
| ip-api.com | IP geolocation fallback |
| WiGLE | BSSID/network observations |
| OpenStreetMap | Detailed street map |
| Google imagery | Satellite layer |
| Esri | Satellite imagery |
| Browser Geolocation API | Device GPS |

Each external service has its own availability, accuracy, terms, and limitations.

---

# Accuracy

ArgusScope is a **geospatial intelligence visualization tool**, not a precision tracking system.

### IP

Results can be affected by:

- ISP infrastructure
- VPNs
- Proxies
- Mobile networks
- CGNAT
- Corporate networks
- Provider databases

### BSSID

Results depend on:

- Database coverage
- Observation quality
- Observation age
- Access-point movement
- Number of observations

### GPS

Browser GPS can be considerably more precise, but accuracy depends on the device, permissions, environment, and available location services.

### Maps

Map detail depends on the underlying map provider and the amount of geographic data available for an area.

---

# Security

ArgusScope separates public frontend code from sensitive backend configuration.

```text
PUBLIC
├── Frontend
├── Map interface
└── API requests

PRIVATE
├── WiGLE credentials
├── Provider configuration
└── Backend secrets
```

Future production hardening can include:

- API authentication
- Rate limiting
- Request logging
- Abuse prevention
- Security headers
- Strict CORS
- Better input validation
- Provider health monitoring

---

# Ethical Use

ArgusScope is intended for:

- Cybersecurity education
- OSINT research
- Defensive security
- Network research
- Geospatial analysis
- Authorized investigations
- Testing systems and data you are authorized to analyze

Do not use the platform to harass, stalk, intimidate, or target individuals.

IP geolocation does not establish a person's exact physical location, and BSSID information should be handled responsibly.

Always follow applicable laws, service terms, and organizational policies.

---

# Privacy

ArgusScope should not intentionally collect unnecessary personal information.

When a lookup is initiated, the requested data is sent to the ArgusScope backend, which may communicate with external providers.

External providers may process requests according to their own privacy policies.

Device GPS is particularly sensitive because it represents the physical location of the device. Browser permission controls access to that information.

---

# Limitations

Current limitations include:

- IP geolocation is approximate.
- WiGLE coverage is not universal.
- WiGLE observations can become outdated.
- External providers can experience downtime.
- External providers can enforce rate limits.
- The in-memory cache disappears after backend restart.
- OpenStreetMap only displays information that has been mapped.
- Satellite imagery providers have different coverage and update schedules.
- Browser GPS requires permission.
- The backend currently has limited application-level abuse protection.
- Production CORS should be tightened.
- RSSI-weighted BSSID fusion is an estimation method, not true RF triangulation.

---

# Development Workflow

## Frontend

```text
1. Modify frontend
       │
       ▼
2. Test locally
       │
       ▼
3. Commit to GitHub
       │
       ▼
4. GitHub Pages deployment
       │
       ▼
5. Test frontend ↔ backend
```

## Backend

```text
1. Modify FastAPI
       │
       ▼
2. Test locally
       │
       ▼
3. Commit backend
       │
       ▼
4. Render deployment
       │
       ▼
5. Test /health
       │
       ▼
6. Test API endpoints
```

---

# Recommended Repository Structure

```text
ArgusScope/
│
├── frontend/
│   └── index.html
│
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   └── ...
│
├── README.md
└── .gitignore
```

The actual repository structure can differ.

---

# Future Development

## Backend

- PostgreSQL/PostGIS
- Persistent caching
- Redis
- API rate limiting
- Authentication
- Structured logging
- Provider health monitoring
- Async HTTP requests
- Provider abstraction
- Improved API versioning

## Geospatial

- Reverse geocoding
- OpenStreetMap POI search
- Address lookup
- Nearby-place analysis
- Geographic bounding boxes
- Route visualization
- Historical location comparison
- Configurable accuracy circles

## BSSID

- Better observation handling
- Multiple observations per BSSID
- Timestamp-aware observations
- Improved confidence scoring
- Visualization of individual observations

## Frontend

- Search history
- Saved investigations
- Target cards
- Coordinate copy
- JSON export
- CSV export
- Investigation timeline
- Map overlays
- Measurement tools
- Improved mobile navigation

---

# Complete Architecture

```text
                    ┌─────────────────────┐
                    │       USER          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   GitHub Pages      │
                    │                     │
                    │  ArgusScope Web UI  │
                    └──────────┬──────────┘
                               │
                         HTTPS / JSON
                               │
                               ▼
                    ┌─────────────────────┐
                    │       Render        │
                    │                     │
                    │ FastAPI Intelligence│
                    │       Engine        │
                    └──────────┬──────────┘
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
             ▼                 ▼                 ▼
        ┌─────────┐       ┌─────────┐      ┌─────────┐
        │ ipapi   │       │ ip-api  │      │ WiGLE   │
        └─────────┘       └─────────┘      └─────────┘
             │                 │                 │
             └─────────────────┼─────────────────┘
                               │
                               ▼
                         Coordinates
                               │
                               ▼
                    ┌─────────────────────┐
                    │       Leaflet       │
                    │                     │
                    │  Street / Satellite │
                    │  / Cyber Dark Maps  │
                    └─────────────────────┘
```

---

# Technology Stack

| Component | Technology |
|---|---|
| Frontend | HTML5 |
| Styling | CSS3 |
| Frontend logic | JavaScript |
| Map engine | Leaflet |
| Street data | OpenStreetMap |
| Satellite imagery | Google / Esri |
| Backend | Python |
| API framework | FastAPI |
| HTTP client | Requests |
| Data validation | Pydantic |
| IP validation | Python `ipaddress` |
| Backend hosting | Render |
| Frontend hosting | GitHub Pages |
| Wireless database | WiGLE |

---

# Version

Current backend version:

```text
2.2.0
```

---


# Disclaimer

ArgusScope provides approximate geospatial information derived from external data sources.

Results should not be treated as guaranteed physical locations.

The developers are not responsible for misuse of the software or decisions made solely from its results.

Use ArgusScope responsibly, legally, and only against systems, networks, devices, and data for which you have appropriate authorization.

---

# Status

**ArgusScope is an active development project.**

Current architecture includes:

```text
✓ Web frontend
✓ Responsive UI
✓ FastAPI backend
✓ IP geolocation
✓ BSSID lookup
✓ BSSID cluster fusion
✓ Browser GPS
✓ Interactive maps
✓ Detailed street map
✓ Satellite layers
✓ Cyber Dark map
✓ Render deployment
✓ GitHub-hosted frontend
```

Future development can focus on accuracy modeling, backend resilience, geospatial analysis, security hardening, and investigation-management features.
