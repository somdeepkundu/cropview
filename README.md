# 🌿 CropView
### Geo-Tagged Crop Growth Stage Field Capture App

> **Layer 1 of 3** · Mobile-first · 100% Open Source · GitHub Pages ready

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Layer](https://img.shields.io/badge/Build-Layer%201%20%E2%9C%85-brightgreen)]()
[![GitHub Pages](https://img.shields.io/badge/Deploy-GitHub%20Pages-blue)]()

---

## 🧭 What is CropView?

CropView is an open-source, mobile-first progressive web app that enables farmers, agronomists, and researchers to:

1. **Point a phone camera at a crop field**
2. **Capture GPS position + compass azimuth + camera FOV** in real time
3. **Project the camera's ground footprint** as a polygon on a map
4. **Tag the observation with a FAO-56 Kc growth stage**
5. **Export structured GeoJSON / CSV** for downstream satellite correlation

The end goal (across 3 layers) is to correlate your phone's point-of-view with **Sentinel-2 satellite imagery** of the same area and time — enabling spectral + geometric validation of crop growth stage using NDVI and Kc values.

---

## 📐 System Architecture (All 3 Layers)

```
┌─────────────────────────────────────────────────────────┐
│  LAYER 3 · Growth Intelligence                          │
│  • NDVI → Kc mapping per crop type                      │
│  • Phenological stage classifier                        │
│  • ET₀ × Kc evapotranspiration estimate                 │
│  • Alert: stage mismatch detection                      │
└──────────────────────────┬──────────────────────────────┘
                           │ needs ↑
┌──────────────────────────▼──────────────────────────────┐
│  LAYER 2 · Satellite ↔ Phone Correlation                │
│  • Sentinel-2 tile fetch (B4, B8 → NDVI)               │
│  • Temporal matching (satellite pass vs. photo date)    │
│  • Spectral sampling at FOV polygon centroid            │
│  • Proj4 coordinate reprojection (WGS84 ↔ UTM)         │
└──────────────────────────┬──────────────────────────────┘
                           │ needs ↑
┌──────────────────────────▼──────────────────────────────┐
│  LAYER 1 · Geo-Camera Capture ✅ (this release)         │
│  • Live camera feed (environment-facing)                │
│  • GPS coordinates + accuracy                           │
│  • Compass azimuth (absolute heading)                   │
│  • FOV frustum → ground polygon (Turf.js)               │
│  • FAO-56 Kc stage selector (4 stages)                  │
│  • GeoJSON + CSV export                                 │
│  • Leaflet map with FOV overlay                         │
│  • Local session persistence                            │
└─────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start (GitHub Pages)

### 1. Fork / Clone

```bash
git clone https://github.com/YOUR_USERNAME/cropview.git
cd cropview
```

### 2. Files

```
cropview/
├── index.html          ← entire Layer 1 app (single file)
├── README.md           ← this file
├── ARCHITECTURE.md     ← deep technical design
└── LICENSE
```

### 3. Enable GitHub Pages

1. Go to your repo → **Settings** → **Pages**
2. Source: `Deploy from a branch`
3. Branch: `main` / `root`
4. Your app is live at: `https://YOUR_USERNAME.github.io/cropview`

### 4. Open on Mobile

- Open the URL on your phone browser (Chrome Android / Safari iOS)
- Tap **Grant Permissions & Start**
- Allow **Camera**, **Location**, **Motion** when prompted
- Walk outside, point at a field, tap **◉ CAPTURE**

---

## 📱 How to Use

### Capture Tab
| Element | Function |
|---|---|
| Live video | Real-time camera feed (rear-facing default) |
| Corner brackets | FOV visual boundary indicator |
| Compass rose | Rotates with your device heading |
| HUD badges | Live GPS + azimuth + accuracy readout |
| ⟳ FLIP | Switch front/rear camera |
| ◉ CAPTURE | Snap photo + record all sensor data |

### Map Tab
| Element | Function |
|---|---|
| Green dot | Your current GPS position |
| Green polygon | Real-time camera FOV ground projection |
| Amber dots | Previous capture locations |
| Amber polygons | FOV footprints of past captures |
| 🛰 SATELLITE | Toggle satellite basemap (ESRI) |
| ⊕ MY POS | Re-center map on you |

### Data Tab
| Section | Function |
|---|---|
| Live Sensor Data | Full sensor readout (lat/lon/alt/az/tilt/FOV/footprint) |
| Kc Stage Selector | Tag current observation with FAO-56 growth stage |
| Captures List | Review, map-jump, download or delete each capture |
| Export | Download all captures as GeoJSON or CSV |

---

## 🌱 FAO-56 Kc Growth Stages

CropView embeds the **FAO Irrigation and Drainage Paper 56** four-stage crop coefficient model:

| Stage | Name | Kc Range | Visual Indicators |
|---|---|---|---|
| **Kc¹** | Initial | 0.3 – 0.4 | Bare soil, seedling emergence |
| **Kc²** | Development | 0.4 – 0.9 | Rapid canopy growth, leaf area increasing |
| **Kc³** | Mid-Season | 0.9 – 1.2 | Full canopy cover, peak transpiration |
| **Kc⁴** | Late-Season | 0.6 – 0.9 | Senescence, maturation, harvest approach |

The selected Kc stage is embedded in every exported GeoJSON feature's `properties.kc_stage` field, ready for Layer 2 satellite cross-validation.

---

## 📦 GeoJSON Export Format

Each capture exports as a GeoJSON `FeatureCollection`. Example feature:

```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [73.856255, 18.516726, 560.0]
  },
  "properties": {
    "id": "CV-1715812345678",
    "timestamp": "2026-05-15T09:22:45.123Z",
    "azimuth_deg": 142,
    "tilt_deg": 38,
    "hfov_deg": 65,
    "gps_accuracy_m": 4.2,
    "kc_stage": 3,
    "footprint_width_m": 8.4,
    "footprint_depth_m": 5.1
  },
  "fov_polygon": {
    "type": "Polygon",
    "coordinates": [[ [73.856200, 18.516690], ... ]]
  }
}
```

This format is designed to be directly ingested by Layer 2's satellite correlation engine.

---

## 🔭 Sensor Stack

| Sensor | Browser API | Notes |
|---|---|---|
| GPS | `navigator.geolocation.watchPosition` | High accuracy mode, 2s refresh |
| Compass | `DeviceOrientationEvent` (absolute) | iOS uses `webkitCompassHeading` |
| Camera | `navigator.mediaDevices.getUserMedia` | Rear-facing, 1080p preferred |
| Tilt | `DeviceOrientationEvent.beta` | Used in footprint projection |

---

## 🧮 FOV Ground Footprint Math

Given:
- **H** = device height from ground (assumed 1.5m, or GPS altitude)
- **β** = device tilt angle (from `DeviceOrientationEvent.beta`)
- **θ_h** = horizontal FOV (~65° for most phones)
- **θ_v** = vertical FOV (θ_h × 9/16 for 16:9 sensor)

```
ground_distance = H × tan(β)
footprint_width = 2 × ground_distance × tan(θ_h / 2)
footprint_depth = ground_distance × tan(θ_v)
```

The four polygon corners are projected from the device position using **Turf.js `destination()`** along the device's compass bearing, forming a trapezoid on the ground.

> **Layer 2 improvement**: Will use actual device altitude (GPS), camera EXIF focal length, and sensor size for precision footprint calculation.

---

## 🛰 Roadmap

### ✅ Layer 1 (This Release)
- [x] Camera capture with GPS + compass
- [x] FOV frustum ground projection
- [x] Leaflet map with live FOV overlay
- [x] FAO-56 Kc stage tagging
- [x] GeoJSON + CSV export
- [x] Session persistence
- [x] Satellite basemap toggle

### 🔜 Layer 2 — Satellite Correlation
- [ ] Sentinel-2 tile fetch via Copernicus STAC API
- [ ] NDVI calculation from B4 (Red) + B8 (NIR)
- [ ] Temporal match: find satellite pass closest to capture timestamp
- [ ] Spectral sampling at FOV polygon centroid + corners
- [ ] Coordinate reprojection: WGS84 → UTM (proj4js)
- [ ] Side-by-side view: phone photo ↔ NDVI patch

### 🔜 Layer 3 — Growth Intelligence
- [ ] NDVI → Kc stage classifier (per crop type)
- [ ] Stage mismatch alert (observed Kc vs. NDVI-derived Kc)
- [ ] ET₀ × Kc daily evapotranspiration estimate
- [ ] Time-series chart of NDVI over season
- [ ] Multi-field comparison dashboard
- [ ] Offline PWA mode (service worker)

---

## 🔧 Tech Stack

| Component | Library/API | License |
|---|---|---|
| Map rendering | [Leaflet.js 1.9](https://leafletjs.com) | BSD-2 |
| Geospatial math | [Turf.js 6](https://turfjs.org) | MIT |
| Basemap | OpenStreetMap / ESRI World Imagery | ODbL / Free |
| Fonts | Google Fonts (Space Mono, DM Sans) | OFL |
| Hosting | GitHub Pages | Free |
| Satellite data (L2) | Copernicus Sentinel-2 STAC | Free / CC-BY |

**Zero backend. Zero dependencies to install. Zero cost.**

---

## 🤝 Contributing

Contributions are very welcome. Areas that need help:

- **iOS compass fix**: `webkitCompassHeading` drift calibration
- **FOV calibration**: Device-specific focal length database
- **Crop type library**: Kc values per crop (wheat, rice, maize, cotton…)
- **Layer 2 prototype**: Sentinel-2 STAC API integration
- **Offline PWA**: Service worker + tile caching

Please open an issue before submitting a large PR.

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

## 🙏 References

- Allen, R.G. et al. (1998). *FAO Irrigation and Drainage Paper 56: Crop Evapotranspiration*. FAO, Rome.
- ESA Copernicus Sentinel-2 Mission. https://sentinel.esa.int
- Turf.js Geospatial Analysis. https://turfjs.org
- Leaflet.js Interactive Maps. https://leafletjs.com

---

*Built with ❤️‍🩹 for farmers, agronomists, and open science.*
