 # 🛰️ STRATOWATCH — Global Conflict Intelligence Platform

> **“Built at the intersection of cybersecurity, cloud intelligence, and AI — to show what’s possible when domain knowledge meets modern tooling.”**

![React](https://img.shields.io/badge/React-18-61DAFB?style=flat&logo=react&logoColor=white)
![APIs](https://img.shields.io/badge/Live_APIs-5-00ff88?style=flat)
![AI Assisted](https://img.shields.io/badge/AI_Assisted-Claude_by_Anthropic-cc785c?style=flat)
![Status](https://img.shields.io/badge/Status-Active-ff2233?style=flat)
![Purpose](https://img.shields.io/badge/Purpose-Portfolio_%26_Research-0066cc?style=flat)

-----

## 🌐 What is STRATOWATCH?

STRATOWATCH is a **real-time geopolitical and military intelligence dashboard** built to monitor
the 2026 Iran-Israel-CENTCOM conflict scenario. It aggregates live data from five public APIs,
renders animated threat trajectories on a custom Canvas radar map, tracks Gulf airspace via
ADS-B, monitors shipping lane disruptions, and displays live cyber threat intelligence — all
inside a single React application with no external UI frameworks.

The name STRATOWATCH reflects the strategic, high-altitude view this platform provides:
watching conflict unfold across military, cyber, economic, maritime, and airspace domains
simultaneously — the way an intelligence analyst would.

-----

## 💡 Why I Built This — The Story Behind the Project

I’m a cybersecurity professional currently stacking certifications across security, cloud, and AI.
At the time of building this, I held the **ISC2 Certified in Cybersecurity (CC)** credential,
was completing a cybersecurity bootcamp, and was deep into my **Oracle Cloud Infrastructure (OCI)**
learning path — covering Cloud Foundations, Cloud Security Professional, and AI Foundations.

I had been doing **TryHackMe rooms** consistently and posting progress on LinkedIn to stay visible
and accountable. But I wanted to build something that went beyond room completions — something that
demonstrated **domain knowledge in action**, not just theoretical understanding.

The idea came from a question I kept asking myself:

> *“If I were a security analyst monitoring a real conflict — what would my screen look like?”*

That question became STRATOWATCH.

-----

## 🤖 First Use of Anthropic Claude AI — Honest Disclosure

**This project was designed and built with the assistance of Claude (by Anthropic) — and I want
to be fully transparent about that.**

This was my **first time using Anthropic’s Claude AI for a project of this scale and complexity.**
Prior to this, I had used AI tools casually. This was different — it was a deliberate, structured
collaboration where I brought the domain knowledge and vision, and Claude helped me execute it in
React and Canvas API at a speed and quality I could not have achieved alone at this stage of my
journey.

Here is what I contributed:

- The **concept, architecture, and intelligence design** — what tabs, what data, what threat types,
  what map layers, what APIs
- The **cybersecurity and geopolitical domain knowledge** — APT actor names, weapon systems,
  intercept rates, shipping lane data, OSINT sourcing
- The **project direction at every iteration** — I reviewed, questioned, and steered every
  decision across multiple conversations
- The **API selection logic** — knowing which APIs were publicly available, what they return,
  and how to handle fallback gracefully
- The **decision to make it portfolio-worthy** — structuring it as a professional, documented,
  deployable project rather than a one-time experiment

Here is what Claude helped me execute:

- Translating my vision into production-grade React component architecture
- Building the Canvas radar animation, trajectory simulation, and shipping lane rendering
- Structuring the multi-API fetch logic with timeout handling and fallback modes
- Typography, color system, and the overall visual design language
- Writing this README

**Why disclose this?** Because the industry is moving fast and professionals who understand how
to use AI tools effectively are exactly what the market needs right now. Hiding AI assistance
is becoming more naive than owning it. What matters is whether you understand what was built —
and I do, completely.

-----

## ⚙️ Features

### 🗺️ Geo-Intel Master Map (Custom Canvas — No Map Libraries)

- Animated radar sweep with real geographic coordinate mapping
- Live ballistic missile, drone, and coalition strike trajectories
- Intercept explosions with multi-ring shockwave animations
- **4 switchable map layers:** All Layers / Missiles+Radar / Air Traffic / Shipping Lanes
- Animated shipping lane routes with open/closed/suspended status and moving vessel icons
- Moving aircraft icons showing active, diverted, rerouted, and halted flights
- Animated pulse rings on high-threat city nodes
- Hormuz closure zone with real-time animated warning rings
- Coordinate grid with lat/lon labels

### 📊 Intelligence Tabs

|Tab              |Description                                                 |
|-----------------|------------------------------------------------------------|
|**Overview**     |Command summary with live military and cyber event feeds    |
|**Geo-Intel Map**|Full interactive map with layered threat visualization      |
|**Military**     |Strike stats, interception rates by system, countries struck|
|**Airspace**     |Live OpenSky ADS-B flight tracking over Gulf region         |
|**Maritime**     |Hormuz closure status, shipping lane disruption analysis    |
|**Cyber Ops**    |AlienVault OTX live threat pulses + Iranian APT actor table |
|**Economic**     |Infrastructure damage cost estimates, live market data      |
|**Viewers**      |CountAPI live viewer counter with regional breakdown        |

### 💹 Live Market Data Integration

- WTI Crude, Brent, S&P 500, Dow Jones, Gold, VIX — Yahoo Finance API
- Bitcoin live price and 24h change — CoinGecko API
- EU Natural Gas and US Gas per Gallon — OSINT estimates
- All with live/OSINT status indicators and graceful fallback

-----

## 🔌 APIs Integrated

|API            |Data Source                                      |Fallback           |
|---------------|-------------------------------------------------|-------------------|
|Yahoo Finance  |Live market prices (WTI, Brent, S&P, VIX, Gold)  |OSINT estimates    |
|CoinGecko      |Bitcoin live price + 24h change                  |Disabled gracefully|
|OpenSky Network|ADS-B Gulf flight tracking (lat 20-41, lon 35-65)|Representative data|
|AlienVault OTX |Live cyber threat intelligence pulses            |OSINT APT table    |
|CountAPI       |Real-time viewer counter                         |Simulated fallback |

All API calls use `allorigins.win` as a CORS proxy with `AbortSignal.timeout()` for
production-grade timeout handling. Every API has a fallback mode so the dashboard
remains fully functional even when live data is unavailable.

-----

## 🛠️ Tech Stack

|Layer        |Technology                                          |
|-------------|----------------------------------------------------|
|Framework    |React 18 — Functional components + Hooks            |
|Map Rendering|Vanilla Canvas API — zero map libraries             |
|Styling      |CSS-in-JS — zero UI frameworks                      |
|Fonts        |Orbitron + Share Tech Mono + Rajdhani (Google Fonts)|
|API Access   |Fetch API + allorigins CORS proxy                   |
|Animation    |requestAnimationFrame — 60fps radar loop            |
|State        |useState + useRef + useCallback                     |

-----

## 🚀 How to Run

### Option 1 — Claude.ai (Instant Preview)

Paste `stratowatch.jsx` directly into [claude.ai](https://claude.ai) as a React artifact.
No setup required. Renders immediately in the browser.

### Option 2 — Local Development

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/stratowatch-intel
cd stratowatch-intel

# Install dependencies
npm install

# Start dev server
npm run dev
```

### Option 3 — Deploy to Vercel (Public URL)

```bash
npm install -g vercel
vercel deploy
```

-----

## 🎯 Skills Demonstrated

This project was deliberately designed to showcase the following technical and domain skills:

**Technical**

- Multi-API integration with timeout handling, CORS proxying, and graceful fallback
- Real-time Canvas animation — coordinate geometry, Bezier curves, radar sweep math
- Complex React state management across multiple components and data streams
- Production-grade error handling — no crashes on API failure
- Geospatial coordinate-to-pixel mapping without external libraries

**Domain Knowledge**

- Cybersecurity threat intelligence — APT actors, TTPs, malware families (WezRat, ZeroCleare,
  Sicarii), IRGC cyber structure
- OSINT methodology — sourcing from CENTCOM, IDF, AlienVault OTX, Critical Threats Project
- Military systems knowledge — Patriot PAC-3, THAAD, Arrow-3, Iron Dome, Shahed-136
- Maritime intelligence — Hormuz chokepoint, LNG shipping lanes, Force Majeure implications
- Economic impact analysis — oil market disruption, infrastructure damage assessment

-----

## 📁 Repository Structure

```
stratowatch-intel/
│
├── stratowatch.jsx       # Main React application (single file)
├── README.md             # This file
└── LICENSE               # MIT License
```

-----

## 🗺️ Roadmap — Future Improvements

These are planned upgrades for future iterations:

- [ ] Replace Gemini API integration with **Claude API** for AI-powered threat analysis panel
- [ ] Connect live **tshark/network traffic analyzer** as a backend data feed
- [ ] Add **WebSocket support** for true real-time event pushing
- [ ] Deploy with a live public URL via Vercel
- [ ] Add **OCI cloud backend** — aligning with OCI Cloud Security Professional certification path
- [ ] Mobile-responsive layout improvements
- [ ] Dark/light mode toggle

-----

## ⚠️ Disclaimer

This dashboard is built entirely for **research, educational, and portfolio demonstration
purposes only.** All military and conflict data is sourced from publicly available OSINT
and open reporting. No classified, sensitive, or private information is used or displayed.
The conflict scenario depicted is based on publicly reported events and is presented as a
technical demonstration, not as a news or intelligence service.

-----

## 👤 Author

kingdavid Christopher
Cybersecurity Professional | Oracle Cloud | AI

Certifications (current/in-progress):

- ✅ ISC2 Certified in Cybersecurity (CC)
- ✅ Cisco Networking Basics
- ✅ Cisco Networking Support & Security
- ✅ Cisco Cyber Threat Management
- 🔄 OCI Cloud Foundations Associate
- 🔄 OCI Cloud Security Professional
- 🔄 OCI AI Foundations Associate
- 🔄 Python Programming Diploma
- 🔄 Anthropic AI — MCP & Claude API

**Connect:**
[LinkedIn](https://linkedin.com/in/yourprofile) ·
[GitHub](https://github.com/yourusername) ·
[TryHackMe](https://tryhackme.com/p/yourprofile)

-----

## 🤝 Acknowledgements

- **Anthropic Claude** — AI assistance for React architecture, Canvas implementation,
  and project documentation. First major project built in collaboration with Claude AI.
- **OpenSky Network** — Free ADS-B flight tracking API
- **AlienVault OTX** — Open Threat Exchange cyber intelligence
- **CoinGecko** — Free cryptocurrency market data
- **Yahoo Finance** — Market data via public API endpoint
- **CountAPI** — Free lightweight counter API

-----

*STRATOWATCH · stratowatch.intel · For Research and Educational Purposes Only*────────────────────────────────────
