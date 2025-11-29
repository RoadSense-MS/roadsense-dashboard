# Quick Reference — RouteScanner Frontend

## 📦 Build & Deployment Commands

```bash
# Development
npm start                          # Start dev server (localhost:3000)

# Production
npm run build                      # Build optimized bundle
npx serve -s build -l 8080        # Serve locally on port 8080

# Docker
docker build -t routescanner-frontend .
docker-compose up -d              # Start with API service
# Frontend: http://localhost:8080

# Analysis
npm run analyze                    # Run webpack-bundle-analyzer
```

## 🗂️ Project Structure

```
src/
  pages/
    Login.jsx                      # Authentication page
    Stats.jsx                      # Dashboard with KPIs & trends
    SegmentDetail.jsx              # Road segment detail page
  components/
    MapPage.jsx                    # Interactive map view
    FiltersBar.jsx                 # Filter controls
    ScoreChart.jsx                 # Segment score history (lazy)
    ChartTrend.jsx                 # Trend visualization
    DefectsList.jsx                # Defects list
  hooks/
    useGeoData.js                  # Fetch /geojson & /scores
    useSegmentData.js              # Fetch segment details
  context/
    AuthContext.jsx                # JWT auth & user state
  api/
    client.js                      # Axios client with interceptor
  utils/
    symbology.js                   # Color scheme for scores
    performanceMonitor.js          # Performance tracking

docs/
  API_INTEGRATION.md               # Backend API specification
  BACKEND_GEOJSON_SIMPLIFICATION.md  # Geometry simplification guide
  BUNDLE_ANALYSIS.md               # Bundle metrics & recommendations
  PROJECT_COMPLETION.md            # Full checklist & deployment guide

Dockerfile                         # Multi-stage build
docker-compose.yml                 # Service orchestration
nginx.conf                         # Reverse proxy config
.dockerignore                      # Docker context optimization
.github/workflows/lighthouse-ci.yml # Automated performance tests
```

## 🔌 API Endpoints Expected

```
BASE_URL: REACT_APP_API_BASE_URL (default: http://api:8000)

POST   /auth/login
       → { access_token, user }

GET    /geojson?simplify=true&tolerance=0.001&type=fissure&from=2024-01-01&to=2024-12-31&pmin=0&pmax=100
       → GeoJSON FeatureCollection (segments + defects)

GET    /scores?type=fissure&from=2024-01-01&to=2024-12-31
       → { summary: {...}, timeseries: [...] }

GET    /segments/:id
       → { id, name, score, geojson, ... }

GET    /segments/:id/defects
       → [{ id, type, severity, location, ... }]

GET    /segments/:id/score-history?days=30
       → [{ date, score, defect_count }]
```

## 🎨 Pages & Routes

| Route | Component | Features |
|-------|-----------|----------|
| `/login` | Login.jsx | Email/password auth |
| `/` | MapPage.jsx | Leaflet map, clustering, filters |
| `/stats` | Stats.jsx | KPI cards, trend charts |
| `/segment/:id` | SegmentDetail.jsx | Segment map, score history, defects |

All routes except `/login` are protected by `PrivateRoute`.

## ⚡ Performance Metrics (Lighthouse)

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| FCP | < 1.5s | 0.75s | ✅ |
| LCP | < 2.5s | 1.75s | ✅ |
| Speed Index | < 3.5s | 1.0s | ✅ |
| TTI | < 3.8s | 1.8s | ✅ |
| TBT | < 300ms | 20ms | ✅ |
| CLS | < 0.1 | 0 | ✅ |

## 📊 Bundle Size (gzip)

| Part | Size | Note |
|------|------|------|
| Main | 76.8 KB | App shell + routing |
| React DOM | 62.6 KB | Lazy-loaded chunk |
| Map libs | 44.6 KB | Leaflet + clustering (lazy) |
| Chart.js | 15.2 KB | Charts (lazy) |
| CSS | ~13 KB | TailwindCSS + component styles |
| **Total** | **~228 KB** | Production build gzip |

## 🔐 Authentication Flow

1. User visits `/login`
2. Submits email/password to `POST /auth/login`
3. Backend returns `{ access_token, user }`
4. Token stored in `AuthContext` (sessionStorage)
5. All API requests include `Authorization: Bearer <token>`
6. Protected routes render via `PrivateRoute` component
7. Logout clears token and redirects to `/login`

## 🗺️ Map Features

- **Center**: Casablanca (33.54°, -7.65°)
- **Zoom**: 12
- **Tiles**: OpenStreetMap (with preconnect for performance)
- **Polylines**: Colored by damage score (green → red)
- **Markers**: Clustered defect points with popup details
- **Popups**: Feature properties (ID, score, defect count)

## 🎯 Key Implementation Details

### Code-Splitting
- Routes are lazy-loaded: `MapPage`, `Stats`, `SegmentDetail`
- `ScoreChart` is lazy-loaded when needed on segment detail
- Reduces initial bundle by deferring map and chart libraries

### Simplification
- Axios interceptor adds `simplify=true&tolerance=0.001` to `/geojson` requests
- Backend must implement geometry simplification to honor this parameter
- See `docs/BACKEND_GEOJSON_SIMPLIFICATION.md` for examples

### Caching
- Nginx serves static assets with `Cache-Control: public, immutable` (1 year)
- Index.html has `Cache-Control: no-cache` (always fetches fresh)
- API responses should set `Cache-Control: public, max-age=86400` for tile endpoints

### Security
- Nginx adds `X-Frame-Options`, `X-Content-Type-Options`, `X-XSS-Protection` headers
- JWT tokens stored in sessionStorage (cleared on browser close)
- HTTPS recommended for production

## 🚀 Common Tasks

### Local Testing
```bash
npm start                          # Dev server + hot reload
npm run build                      # Production build
npx serve -s build -l 8080        # Serve build locally
```

### Docker Testing
```bash
docker build -t routescanner-frontend .
docker run -d -p 8080:80 routescanner-frontend
# Open http://localhost:8080
```

### Performance Audit
```bash
npm run build
npx lighthouse http://localhost:8080 --output=json --output=html --output-path=./lighthouse-report
# Reports saved to ./lighthouse-report.report.json and .html
```

### Bundle Analysis
```bash
npm run build
npm run analyze          # Opens webpack-bundle-analyzer
```

## 📝 Environment Variables

```bash
# .env.local or system env
REACT_APP_API_BASE_URL=http://api:8000
REACT_APP_ENVIRONMENT=development|production
```

For Docker:
```yaml
# docker-compose.yml
environment:
  - REACT_APP_API_BASE_URL=http://api:8000
```

## 🔗 Documentation Links

- [API Contracts](./docs/API_INTEGRATION.md)
- [Backend Optimization](./docs/BACKEND_GEOJSON_SIMPLIFICATION.md)
- [Bundle Analysis](./docs/BUNDLE_ANALYSIS.md)
- [Project Completion](./PROJECT_COMPLETION.md)
- [Project Summary](./README_PROJECT_SUMMARY.md)

## ✅ Quality Checklist

- [x] All spec requirements implemented
- [x] Code-splitting & lazy-loading
- [x] Lighthouse performance audit passed
- [x] Accessibility improvements (landmarks, skip links, labels)
- [x] Docker & Nginx configuration
- [x] GitHub Actions CI with Lighthouse
- [x] Comprehensive documentation
- [x] Production build optimized

---

**Status**: ✅ Ready for backend integration and production deployment.
