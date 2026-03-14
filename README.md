<div align="center">

<!-- Header Banner -->
<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=180&section=header&text=AQI%20Map&fontSize=72&fontColor=fff&animation=twinkling&fontAlignY=35&desc=Real-time%20Air%20Quality%20Visualization%20%E2%80%94%20Central%20Oregon&descAlignY=58&descSize=18" />

<br/>

<!-- Badges -->
<img src="https://img.shields.io/badge/Status-Offline%20(Credits%20Exhausted)-critical?style=for-the-badge" alt="Status"/>
<img src="https://img.shields.io/badge/Region-Central%20Oregon-2ea44f?style=for-the-badge&logo=googlemaps&logoColor=white" alt="Region"/>
<img src="https://img.shields.io/badge/Update%20Cycle-Every%202%20Hours-0075ca?style=for-the-badge" alt="Update Frequency"/>
<img src="https://img.shields.io/badge/Interpolation-Ordinary%20Kriging-8a2be2?style=for-the-badge" alt="Method"/>

<br/><br/>

<a href="https://nbpub.github.io/AQI_Map/">
  <img src="https://img.shields.io/badge/🗺️ View Live Map-Click Here-ff6b35?style=for-the-badge" alt="Live Map"/>
</a>
&nbsp;
<a href="/docs#aqi-map-documentation">
  <img src="https://img.shields.io/badge/📖 Documentation-Read Docs-4a90d9?style=for-the-badge" alt="Docs"/>
</a>

<br/><br/>

> 🚨 **API credits fully consumed** — The map stopped live updates on **January 2, 2025**.
> The page and all historical data remain accessible. See the [API Limitation](#-api-limitation) section for full details.

</div>

---

## 📖 Table of Contents

- [Motivation](#-motivation)
- [Live Demo](#-live-demo)
- [How It Works](#-how-it-works)
- [Tech Stack](#-tech-stack)
- [API Limitation](#-api-limitation)
- [Deployment](#-deployment)
- [Roadmap](#-roadmap)

---

## 🎯 Motivation

Central Oregon's high desert summers are stunning — but late summer also brings a familiar hazard: **wildfire smoke**. Fine particulate matter (PM2.5) from regional and out-of-state fires can dramatically degrade air quality, sometimes within hours, often with little warning.

Existing air quality tools tend to show **raw sensor readings** or **sparse station maps** that are hard to interpret at a glance. This project was built to answer a simple question:

> *"Is the air outside safe to breathe right now — and where exactly is it worse?"*

The AQI Map addresses this by:
- Pulling live sensor data from the **PurpleAir network**, which is denser and more localized than government monitoring stations
- **Spatially interpolating** that data into a smooth, continuous surface using Ordinary Kriging
- Displaying the result as an intuitive **color-coded overlay** on an interactive map

The result is a visualization that's immediately readable — no data science background required.

---

## 🖥️ Live Demo

The hosted demo is scoped to **[Central Oregon](https://en.wikipedia.org/wiki/Central_Oregon)**, but the underlying code supports any geographic region by specifying coordinate bounds.

While API credits are exhausted, the page still loads with the **last 10 captured timepoints**, allowing you to explore how the map looked and how the interface works.

**→ [https://nbpub.github.io/AQI_Map/](https://nbpub.github.io/AQI_Map/)**

---

## ⚙️ How It Works

The map is produced through a fully automated pipeline that runs on a schedule via GitHub Actions. Here's the end-to-end flow:

```
┌─────────────────────────────────────────────────────────────────────┐
│                        GitHub Actions (cron)                        │
└──────────────────────────────┬──────────────────────────────────────┘
                               │  every 2 hours
                               ▼
                   ┌───────────────────────┐
                   │  PurpleAir API call   │
                   │  (get_sensors)        │
                   └───────────┬───────────┘
                               │  ~180–200 sensor readings (PM2.5)
                               ▼
                   ┌───────────────────────┐
                   │  Data Filtering       │
                   │  confidence · recency │
                   │  location quality     │
                   └───────────┬───────────┘
                               │  clean point data
                               ▼
                   ┌───────────────────────┐
                   │  AQI Calculation      │
                   │  60-min average PM2.5 │
                   └───────────┬───────────┘
                               │  AQI values at sensor coords
                               ▼
                   ┌───────────────────────┐
                   │  Ordinary Kriging     │
                   │  spatial interpolation│
                   └───────────┬───────────┘
                               │  continuous AQI raster + variance map
                               ▼
                   ┌───────────────────────┐
                   │  Image Overlay        │
                   │  pushed to repo       │
                   └───────────┬───────────┘
                               │
                               ▼
                   ┌───────────────────────┐
                   │  GitHub Pages         │
                   │  Leaflet + OSM map    │
                   │  updated live         │
                   └───────────────────────┘
```

### 1. 📡 Sensor Data — PurpleAir API

PurpleAir operates a large network of low-cost, consumer-grade air quality sensors. The [`get_sensors`](https://api.purpleair.com/#api-sensors-get-sensors-data) endpoint is queried with a bounding box covering Central Oregon.

To ensure data quality, strict filtering is applied before any sensor reading is used:

| Filter | Description |
|--------|-------------|
| **Confidence** | Both internal channels of the sensor must agree |
| **Recency** | Only sensors with a recent timestamp are included |
| **Location Rating** | Sensors with poor placement (e.g., indoors near sources) are excluded |

After filtering, typically **180 to 200 sensors** remain per call.

### 2. 🧮 AQI Calculation

Raw PM2.5 concentrations are converted to AQI values using the **60-minute average** from each sensor. This smooths out transient spikes while remaining responsive to changing conditions. AQI is updated with every 2-hour cycle.

### 3. 🗺️ Spatial Interpolation — Ordinary Kriging

Point sensor data is converted into a spatially continuous surface using **[Ordinary Kriging](https://en.wikipedia.org/wiki/Kriging)** — a geostatistical interpolation technique that estimates values at unsampled locations based on spatial autocorrelation.

Two outputs are generated per cycle:
- **AQI raster** — color-mapped and used as the map overlay
- **[Variance plot](/data/kriging_variance.png)** — visualizes interpolation uncertainty; areas far from sensors have higher variance

### 4. 🌐 Map Rendering — Leaflet + OpenStreetMap

The interpolated raster is overlaid on an interactive map powered by [Leaflet.js](https://leafletjs.com/), using [OpenStreetMap](https://www.openstreetmap.org/) tiles as the base layer. The interface supports:

- Scrolling through the **current + 9 previous results** for temporal comparison
- Toggling the overlay opacity
- Standard pan and zoom navigation

### 5. 🤖 Automation — GitHub Actions

A scheduled GitHub Actions workflow ties everything together. It runs every 2 hours, executes the data pipeline, and commits the updated overlay images directly to the repository, triggering a GitHub Pages rebuild.

---

## 🛠️ Tech Stack

<div align="center">

| Layer | Technology |
|-------|-----------|
| **Sensor Data** | [PurpleAir API](https://api.purpleair.com/) |
| **AQI Standard** | [US EPA PM2.5 / AQI](https://www.airnow.gov/aqi/aqi-basics/) |
| **Interpolation** | [Ordinary Kriging](https://en.wikipedia.org/wiki/Kriging) (Python) |
| **Frontend Map** | [Leaflet.js](https://leafletjs.com/) |
| **Map Tiles** | [OpenStreetMap](https://www.openstreetmap.org/) |
| **Hosting** | [GitHub Pages](https://pages.github.com/) |
| **Automation** | [GitHub Actions](https://github.com/features/actions) |

</div>

<div align="center">

<br/>

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![Leaflet](https://img.shields.io/badge/Leaflet.js-199900?style=flat-square&logo=leaflet&logoColor=white)
![OpenStreetMap](https://img.shields.io/badge/OpenStreetMap-7EBC6F?style=flat-square&logo=openstreetmap&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-222222?style=flat-square&logo=github&logoColor=white)

</div>

---

## ⚡ API Limitation

PurpleAir uses a **one-time credit system** — credits are consumed with each API call and **do not reset or refresh**. This makes long-term continuous operation a planning challenge.

### Current Key (Nov 2024)

A fresh API key was obtained on **November 2, 2024**, loaded with **1,000,000 points**.

| Parameter | Value |
|-----------|-------|
| Key issued | November 2, 2024 |
| Starting credits | 1,000,000 points |
| Calls per day | 12 (every 2 hours) |
| Points per call | ~1,378 avg |
| **Points per day** | **~16,528 ± 46.3** (n=7 measured) |
| Estimated duration | ~60.5 days |
| Operation start | November 3, 2024 |
| **Credits exhausted** | **January 2, 2025** |

<details>
<summary><strong>📊 Per-call cost breakdown</strong></summary>

<br/>

Point cost varies slightly with the number of active sensors and the volume of data returned per sensor. For Central Oregon:

| Sensors Queried | Data Transferred | Approximate Points |
|:-:|:-:|:-:|
| 180 – 200 sensors | 12.0 – 12.25 KB | 1,370 – 1,380 pts |

The `get_sensors` endpoint charges based on the number of sensor-field combinations returned. Sensor count fluctuates as devices go offline or fail quality checks.

**Measured daily usage:** `16,528.4 ± 46.3 points/day` (sample size: n=7)

</details>

<details>
<summary><strong>🔄 Previous API Keys</strong></summary>

<br/>

This is not the first key used for this project. Prior keys were similarly exhausted during earlier operation periods. Each new key extends the project's lifespan by approximately 60 days at the current call rate.

</details>

---

## 🚀 Deployment

Full deployment documentation — including environment setup, configuration for custom regions, GitHub Actions workflow details, and GitHub Pages setup — is available on the **[documentation page](/docs#aqi-map-documentation)**.

At a high level, deploying your own instance requires:

1. **A PurpleAir API key** with sufficient credits for your intended run duration
2. **Defining a bounding box** for your region of interest (lat/lon bounds)
3. **Configuring GitHub Actions secrets** with your API credentials
4. **Enabling GitHub Pages** on the repository

The code is designed to be region-agnostic — Central Oregon is simply the default configuration for this demo.

---

## 🗓️ Roadmap

Planned improvements for future development:

- [ ] **Clean map markers** — Remove stale sensor markers when scrolling through historical Kriging timepoints, so only the relevant overlay is shown
- [ ] **Auto-scroll playback** — Add a button to animate through the 10 stored results as a time-lapse, giving an intuitive sense of how air quality evolved
- [ ] **Configurable region UI** — Allow users to adjust the bounding box from the web interface without needing to redeploy
- [ ] **AQI color legend** — Add an always-visible legend explaining the AQI color scale directly on the map

---

## 📄 License & Attribution

- **Map tiles** © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors
- **Sensor data** sourced from the [PurpleAir](https://www2.purpleair.com/) network
- **AQI methodology** based on [US EPA standards](https://www.epa.gov/pm-pollution/particulate-matter-pm-basics)

---

<div align="center">

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=100&section=footer" />

<sub>Built for Central Oregon · Powered by PurpleAir & OpenStreetMap · Automated with GitHub Actions</sub>

</div>
