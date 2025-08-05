# ClimInvest - Climate Micro-Insurance via SMS

<div align="center">
  <img src="assets/icon.png" alt="ClimInvest Logo" width="120" height="120">

  ## Revolutionizing Agricultural Insurance in West Africa
</div>

[![OpenEPI Hackathon 2025](https://img.shields.io/badge/OpenEPI-Hackathon%202025-blue)](https://developer.openepi.io/hackathon-2025)
[![SDG](https://img.shields.io/badge/SDG-2%20Zero%20Hunger-green)](https://sdgs.un.org/goals/goal2)
[![React Native](https://img.shields.io/badge/React%20Native-0.72-blue)](https://reactnative.dev/)
[![OpenEPI](https://img.shields.io/badge/OpenEPI-Integrated-green)](https://openepi.io/)

## Project Overview

ClimInvest is a revolutionary mobile application that democratizes access to agricultural insurance in West Africa. Using a mobile-first approach with SMS integration and phone calls, the application offers climate micro-insurance accessible to all farmers, even those without smartphones.

## 🎥 Video Demonstration

### Video 1 - Application Overview

[Voir Video 1](https://res.cloudinary.com/dtwx8br7o/video/upload/v1754236059/Video1_gpawi3.mp4)

---

### Video 2 - Detailed Features

[Voir Video 2](https://res.cloudinary.com/dtwx8br7o/video/upload/v1754236085/Video2_k8nf5s.mp4)

> **Discover ClimInvest in action**: SMS subscription, automatic payout triggers, Analytics tab with credit scoring, and all accessibility features.

*[Click on the video links to see the complete application demonstration]*

### Problem Addressed

- **97% of African farmers uninsured** vs 50% in Asia
- **0.7% of global insurance premiums** for 17% of arable land
- **Compensation delays: 3-6 months** with traditional insurance
- **Access barriers**: prohibitive costs, administrative complexity, low banking penetration

### Innovative Solution

**100% mobile** micro-insurance using:
- **Call to 980** - Advisors guiding the complete process
- **SMS "MON ASSURANCE AGRICOLE" to 980** - Simple SMS subscription
- **Advanced OpenEPI APIs** - Real-time climate, soil, yield data
- **Mobile Money** - MTN MoMo, Orange Money, Flooz
- **Voice interface** - Local languages (fon, yoruba, bambara)
- **Community** - One user can help their entire farming community

## Accessibility Features

### Multiple Access Options

1. **Mobile Application** (Android/iOS)
   - Complete interface with all features
   - Screen reader support
   - Voice navigation in local languages
   - Full OpenEPI APIs integration

2. **Phone Access (980)**
   - **For users without Android smartphones**
   - Advisors guiding the subscription process
   - Same process for loan applications
   - Multilingual support
   - Access to same OpenEPI data via advisors

3. **Simple SMS to 980**
   - Send **"MON ASSURANCE AGRICOLE"** to 980
   - SMS-guided process
   - SMS confirmation and follow-up
   - Accessible on all phones

4. **Community System**
   - A farmer with the app can help their community
   - Facilitated collective registration
   - Sharing of OpenEPI climate information

### Data Transparency

The **Analytics** tab provides advanced services based on user feedback:
- **Agricultural credit score** based on OpenEPI data (soil, climate, yields)
- **Access to financing** with eligible amounts and personalized rates
- **Agricultural analysis**: soil quality, historical yields, market prices
- **Personalized advice** to optimize production and sales

## OpenEPI Integration - System Core

### OpenEPI APIs Used

1. **Climate Data API**
   - Historical and real-time meteorological data
   - Drought and precipitation indices
   - Automatic payout triggers

2. **Soil Quality API**
   - Soil quality analysis
   - Personalized agricultural recommendations
   - Credit scoring based on soil quality

3. **Crop Health Monitoring**
   - Satellite crop surveillance
   - Early detection of water stress
   - Proactive alerts to farmers

4. **Flood Detection API**
   - Real-time flood detection
   - Mapping of affected areas
   - Automatic compensation triggers

### Benefits of OpenEPI Integration

- **Reliable and updated data** in real-time
- **Complete geographical coverage** of West Africa
- **Advanced detection algorithms** for climate events
- **Seamless integration** with mobile payment systems
- **Cost reduction** through automation

## Competitive Advantages vs Traditional Insurance

| **Criteria** | **ClimInvest** | **Traditional Insurance** |
|-------------|-----------------|-------------------------|
| **Monthly cost** | 200-1,000 FCFA | 5,000-20,000 FCFA |
| **Payout delay** | 24-72h automatic | 3-6 months (expertise) |
| **Accessibility** | Mobile money + illiterate | Office + bank account |
| **Geographic coverage** | Remote areas via satellite | Urban centers only |
| **Prerequisites** | Basic phone | Smartphone + internet |

## Proven Technological Foundations

Satellite data is accessible via Digital Earth Africa, which provides historical images since 1984 covering the entire African continent. The NDVI system can detect crop stress up to **2 weeks before visual detection**, while Sentinel-1 offers **>95% accuracy for flood detection**.

AGRHYMET infrastructures guarantee access to real-time meteorological data for the 15 ECOWAS countries, with automated weather stations and Next Generation seasonal forecasting models.

## Sustainable Economic Model

- **80%** of premiums → Compensation fund
- **15%** → Technical costs (data, SMS, platform)
- **5%** → Growth and reserves

**Break-even point**: 50,000 insured with a government reserve fund of 10% of premiums for major disasters.

## International Lessons

**Kenya (Kilimo Salama)**: Launched in 2009 with 200 farmers, the program reached 51,000 insured in Kenya and 14,000 in Rwanda. Premium revenues increased from 19 million KSh in 2011 to 33 million KSh in 6 months in 2012.

**India (PMFBY)**: The government program covers 194 million farmers with 50% premium subsidies. Premiums range from 1.5% to 5% of insured amounts depending on crops.

These successes demonstrate the viability of index insurance at scale with appropriate public support.

## Projected Impact

- **500,000 farmers** protected by 2027
- **2.5 million people** secured (families included)
- **45x faster compensation** (24-72h vs 3-6 months)
- **10x lower cost** than traditional insurance
- **Extended geographic coverage** thanks to OpenEPI data

## SDG Contribution

- **SDG 2**: Zero Hunger - Food security
- **SDG 1**: Rural poverty reduction
- **SDG 13**: Climate action - Shock resilience
- **SDG 10**: Reduced inequalities - Democratized access

## Installation and Setup

### Prerequisites

```bash
Node.js >= 18
npm or yarn
Expo CLI
Android Studio (for Android emulation)
Xcode (for iOS emulation, macOS only)
OpenEPI API keys (required)
```

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/clim-invest
cd clim-invest

# Install dependencies
npm install

# Configure OpenEPI keys in .env
cp .env.example .env
# Add your OpenEPI keys

# Start development server
npm start

# Run on Android
npm run android

# Run on iOS
npm run ios
```

### OpenEPI Configuration

```bash
# Required environment variables
OPENEPI_API_KEY=your_api_key_here
OPENEPI_BASE_URL=https://api.openepi.io/v1
OPENEPI_CLIENT_ID=your_client_id
OPENEPI_CLIENT_SECRET=your_client_secret
```

## Project Structure

```
ClimInvest/
├── src/
│   ├── components/          # Reusable components
│   │   ├── common/         # Accessible components
│   │   ├── cards/          # Information cards
│   │   └── dashboard/      # Dashboard components
│   ├── screens/            # Main screens
│   ├── navigation/         # Navigation configuration
│   ├── services/           # OpenEPI APIs and other services
│   │   ├── openEpiService.ts        # Main OpenEPI service
│   │   ├── hybridOpenEpiService.ts  # Hybrid service with fallback
│   │   └── creditScoringService.ts  # OpenEPI-based scoring
│   ├── store/              # Redux store and slices
│   ├── types/              # TypeScript types
│   ├── utils/              # Utilities
│   └── hooks/              # Custom hooks
├── docs/                   # Technical documentation
└── assets/                 # Static resources
```

## Complete Documentation

- [Technical Architecture](docs/TECHNICAL_ARCHITECTURE.md)
- [OpenEPI Integration](docs/DATA_SOURCES.md)
- [API Documentation](docs/API_DOCUMENTATION.md)
- [Business Model](docs/BUSINESS_MODEL.md)
- [Impact Metrics](docs/IMPACT_METRICS.md)
- [Deployment Guide](docs/DEPLOYMENT_GUIDE.md)
- [User Feedback](docs/USER_FEEDBACK.md)

## Testing and Quality

```bash
# Run tests
npm test

# Tests with coverage
npm run test:coverage

# OpenEPI integration tests
npm run test:openepi

# Linting
npm run lint
```

## Contact and Support

### For Farmers
- **Phone**: 980 (free)
- **SMS**: Send "MON ASSURANCE AGRICOLE" to 980
- **Application**: Download from Google Play Store

### For Developers
- **ClimInvest Team** - contact@climinvest.org
- **OpenEPI Documentation** - [https://developer.openepi.io](https://developer.openepi.io)
- **Repository** - [https://github.com/climinvest/mobile-app](https://github.com/climinvest/mobile-app)

## License

This project is under MIT license. See the `LICENSE` file for more details.

---

*Developed for OpenEPI Hackathon 2025 - Revolutionizing African agriculture through climate data*
