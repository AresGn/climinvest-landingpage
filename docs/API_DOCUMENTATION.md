# API Documentation - ClimInvest

## Overview

ClimInvest integrates several APIs to provide a complete climate micro-insurance service. The architecture prioritizes OpenEPI APIs as the main data source, with fallback services to ensure availability.

## OpenEPI APIs - Main Services

### 1. WeatherClient API

**Main Endpoint:** OpenEPI WeatherClient
**Usage:** Real-time meteorological data and forecasts

```typescript
// Client configuration
import { WeatherClient } from 'openepi-client';

const weatherClient = new WeatherClient({
  apiKey: process.env.OPENEPI_API_KEY,
  baseURL: 'https://api.openepi.io/v1'
});

// Available methods
interface WeatherClientMethods {
  getLocationForecast(lat: number, lon: number): Promise<LocationForecast>;
  getSummaryForecast(lat: number, lon: number): Promise<SummaryForecast>;
  getWeatherAlerts(lat: number, lon: number): Promise<WeatherAlert[]>;
}
```

**Usage example:**
```typescript
// Weather forecast retrieval
const forecast = await weatherClient.getLocationForecast(6.3703, 2.3912);

// Response structure
interface LocationForecast {
  current: {
    temperature: number;
    humidity: number;
    precipitation: number;
    windSpeed: number;
    pressure: number;
  };
  daily: Array<{
    date: string;
    temperatureMax: number;
    temperatureMin: number;
    precipitationSum: number;
    windSpeedMax: number;
  }>;
  alerts: WeatherAlert[];
}
```

**Déclencheurs automatiques:**
```typescript
// Évaluation des seuils de déclenchement
const evaluateDroughtTrigger = (forecast: LocationForecast) => {
  const droughtIndicators = {
    consecutiveDryDays: forecast.daily.filter(d => d.precipitationSum < 1).length,
    highTemperature: forecast.daily.some(d => d.temperatureMax > 38),
    lowHumidity: forecast.current.humidity < 30
  };

  return droughtIndicators.consecutiveDryDays > 21 && 
         droughtIndicators.highTemperature && 
         droughtIndicators.lowHumidity;
};
```

### 2. SoilClient API

**Endpoint Principal:** OpenEPI SoilClient
**Utilisation:** Analyse de la qualité des sols et recommandations

```typescript
import { SoilClient } from 'openepi-client';

const soilClient = new SoilClient({
  apiKey: process.env.OPENEPI_API_KEY
});

// Méthodes disponibles
interface SoilClientMethods {
  getSoilType(lat: number, lon: number): Promise<SoilType>;
  getSoilProperties(lat: number, lon: number): Promise<SoilProperties>;
  getSoilHealth(lat: number, lon: number): Promise<SoilHealth>;
}
```

**Exemple d'utilisation:**
```typescript
// Analyse complète du sol
const soilAnalysis = async (lat: number, lon: number) => {
  const [soilType, properties, health] = await Promise.all([
    soilClient.getSoilType(lat, lon),
    soilClient.getSoilProperties(lat, lon),
    soilClient.getSoilHealth(lat, lon)
  ]);

  return {
    type: soilType,
    properties: {
      ph: properties.ph,
      organicMatter: properties.organicMatter,
      waterRetention: properties.waterRetention,
      fertility: properties.fertilityIndex
    },
    health: {
      score: health.overallScore,
      erosionRisk: health.erosionRisk,
      contamination: health.contaminationLevel
    },
    recommendations: generateSoilRecommendations(soilType, properties, health)
  };
};
```

**Scoring de crédit basé sur le sol:**
```typescript
const calculateSoilCreditScore = (soilData: SoilAnalysis) => {
  const scores = {
    ph: scorePH(soilData.properties.ph), // 0-100
    organic: scoreOrganicMatter(soilData.properties.organicMatter), // 0-100
    water: scoreWaterRetention(soilData.properties.waterRetention), // 0-100
    fertility: soilData.properties.fertility // 0-100
  };

  const weightedScore = (
    scores.ph * 0.25 +
    scores.organic * 0.30 +
    scores.water * 0.25 +
    scores.fertility * 0.20
  );

  return {
    soilScore: Math.round(weightedScore),
    creditImpact: weightedScore > 70 ? 'Positive' : weightedScore > 50 ? 'Neutral' : 'Negative',
    recommendations: generateImprovementPlan(scores)
  };
};
```

### 3. FloodClient API

**Endpoint Principal:** OpenEPI FloodClient
**Utilisation:** Détection et prévision d'inondations

```typescript
import { FloodClient } from 'openepi-client';

const floodClient = new FloodClient({
  apiKey: process.env.OPENEPI_API_KEY
});

// Méthodes disponibles
interface FloodClientMethods {
  getFloodRisk(lat: number, lon: number): Promise<FloodRisk>;
  getHistoricalFloods(lat: number, lon: number): Promise<HistoricalFlood[]>;
  getFloodAlerts(lat: number, lon: number): Promise<FloodAlert[]>;
}
```

**Exemple d'utilisation:**
```typescript
// Surveillance des inondations
const monitorFloodRisk = async (lat: number, lon: number) => {
  const floodRisk = await floodClient.getFloodRisk(lat, lon);
  
  return {
    currentRisk: floodRisk.riskLevel, // LOW, MEDIUM, HIGH, CRITICAL
    probability7Days: floodRisk.probability,
    waterLevel: floodRisk.currentWaterLevel,
    trend: floodRisk.waterLevelTrend, // RISING, STABLE, FALLING
    evacuationZones: floodRisk.evacuationZones,
    estimatedDamage: calculatePotentialDamage(floodRisk),
    triggerThreshold: floodRisk.riskLevel === 'HIGH' && floodRisk.probability > 0.7
  };
};

// Déclenchement automatique inondation
const evaluateFloodTrigger = (floodData: FloodRisk) => {
  return {
    triggered: floodData.riskLevel === 'CRITICAL' || 
               (floodData.riskLevel === 'HIGH' && floodData.probability > 0.8),
    payoutPercentage: calculateFloodPayout(floodData.riskLevel),
    evidence: {
      waterLevel: floodData.currentWaterLevel,
      satelliteImagery: floodData.satelliteEvidence,
      weatherData: floodData.associatedWeather
    }
  };
};
```

### 4. CropHealthClient API

**Endpoint Principal:** OpenEPI CropHealthClient
**Utilisation:** Surveillance satellite des cultures

```typescript
import { CropHealthClient } from 'openepi-client';

const cropHealthClient = new CropHealthClient({
  apiKey: process.env.OPENEPI_API_KEY
});

// Méthodes disponibles
interface CropHealthClientMethods {
  getCropHealth(lat: number, lon: number, cropType: string): Promise<CropHealth>;
  getNDVITimeSeries(lat: number, lon: number): Promise<NDVITimeSeries>;
  getCropStressIndicators(lat: number, lon: number): Promise<StressIndicators>;
}
```

**Exemple d'utilisation:**
```typescript
// Surveillance santé des cultures
const monitorCropHealth = async (lat: number, lon: number, cropType: string) => {
  const [health, ndviSeries, stressIndicators] = await Promise.all([
    cropHealthClient.getCropHealth(lat, lon, cropType),
    cropHealthClient.getNDVITimeSeries(lat, lon),
    cropHealthClient.getCropStressIndicators(lat, lon)
  ]);

  return {
    currentNDVI: health.ndvi,
    healthStatus: interpretNDVI(health.ndvi),
    trend: analyzeNDVITrend(ndviSeries),
    stressFactors: {
      waterStress: stressIndicators.waterStress,
      nutrientDeficiency: stressIndicators.nutrientDeficiency,
      pestPressure: stressIndicators.pestPressure,
      diseaseRisk: stressIndicators.diseaseRisk
    },
    growthStage: health.growthStage,
    yieldPrediction: predictYield(health, cropType),
    recommendations: generateCropAdvice(health, stressIndicators)
  };
};

// Déclenchement stress des cultures
const evaluateCropStressTrigger = (cropData: CropHealth, ndviHistory: NDVITimeSeries) => {
  const criticalNDVI = cropData.ndvi < 0.25;
  const persistentStress = ndviHistory.values.slice(-14).every(v => v < 0.3);
  const severeStress = cropData.stressIndicators.waterStress > 0.8;

  return {
    triggered: criticalNDVI && persistentStress && severeStress,
    severity: calculateStressSeverity(cropData),
    payoutAmount: calculateCropStressPayout(cropData.ndvi, ndviHistory),
    evidence: {
      ndviCurrent: cropData.ndvi,
      ndviTrend: ndviHistory,
      stressLevel: cropData.stressIndicators,
      satelliteImages: cropData.satelliteEvidence
    }
  };
};
```

### 5. GeocoderClient API

**Endpoint Principal:** OpenEPI GeocoderClient
**Utilisation:** Services de géolocalisation et zonage

```typescript
import { GeocoderClient } from 'openepi-client';

const geocoderClient = new GeocoderClient({
  apiKey: process.env.OPENEPI_API_KEY
});

// Méthodes disponibles
interface GeocoderClientMethods {
  reverseGeocode(lat: number, lon: number): Promise<LocationInfo>;
  getClimateZone(lat: number, lon: number): Promise<ClimateZone>;
  getAgriculturalZone(lat: number, lon: number): Promise<AgriculturalZone>;
}
```

**Exemple d'utilisation:**
```typescript
// Géolocalisation et zonage
const getLocationProfile = async (lat: number, lon: number) => {
  const [locationInfo, climateZone, agriZone] = await Promise.all([
    geocoderClient.reverseGeocode(lat, lon),
    geocoderClient.getClimateZone(lat, lon),
    geocoderClient.getAgriculturalZone(lat, lon)
  ]);

  return {
    address: {
      country: locationInfo.country,
      region: locationInfo.administrativeArea,
      district: locationInfo.locality,
      village: locationInfo.subLocality
    },
    climate: {
      zone: climateZone.type,
      rainfall: climateZone.averageRainfall,
      temperature: climateZone.averageTemperature,
      season: climateZone.currentSeason
    },
    agriculture: {
      zone: agriZone.type,
      suitableCrops: agriZone.recommendedCrops,
      soilType: agriZone.predominantSoil,
      riskLevel: agriZone.climateRisk
    },
    premiumMultiplier: calculateLocationRisk(climateZone, agriZone)
  };
};
```

## APIs de Fallback

### Open-Meteo API (Fallback Météo)

```typescript
// Service de fallback météorologique
class OpenMeteoFallbackService {
  private baseURL = 'https://api.open-meteo.com/v1';

  async getCurrentWeather(lat: number, lon: number) {
    const params = new URLSearchParams({
      latitude: lat.toString(),
      longitude: lon.toString(),
      current: 'temperature_2m,precipitation,wind_speed_10m,relative_humidity_2m',
      daily: 'temperature_2m_max,temperature_2m_min,precipitation_sum',
      timezone: 'auto',
      forecast_days: '7'
    });

    const response = await fetch(`${this.baseURL}/forecast?${params}`);
    const data = await response.json();

    // Conversion au format OpenEPI pour compatibilité
    return this.convertToOpenEPIFormat(data);
  }

  private convertToOpenEPIFormat(openMeteoData: any) {
    return {
      current: {
        temperature: openMeteoData.current.temperature_2m,
        humidity: openMeteoData.current.relative_humidity_2m,
        precipitation: openMeteoData.current.precipitation,
        windSpeed: openMeteoData.current.wind_speed_10m
      },
      daily: openMeteoData.daily.time.map((date: string, index: number) => ({
        date,
        temperatureMax: openMeteoData.daily.temperature_2m_max[index],
        temperatureMin: openMeteoData.daily.temperature_2m_min[index],
        precipitationSum: openMeteoData.daily.precipitation_sum[index]
      })),
      source: 'Open-Meteo'
    };
  }
}
```

### NASA POWER API (Données Historiques)

```typescript
// Service pour données historiques
class NASAPowerService {
  private baseURL = 'https://power.larc.nasa.gov/api/temporal/daily/point';

  async getHistoricalWeather(lat: number, lon: number, startDate: string, endDate: string) {
    const params = new URLSearchParams({
      parameters: 'T2M,PRECTOTCORR,RH2M,WS10M',
      community: 'AG',
      longitude: lon.toString(),
      latitude: lat.toString(),
      start: startDate,
      end: endDate,
      format: 'JSON'
    });

    const response = await fetch(`${this.baseURL}?${params}`);
    const data = await response.json();

    return this.processNASAData(data);
  }

  private processNASAData(nasaData: any) {
    const parameters = nasaData.properties.parameter;
    const dates = Object.keys(parameters.T2M);

    return dates.map(date => ({
      date,
      temperature: parameters.T2M[date],
      precipitation: parameters.PRECTOTCORR[date],
      humidity: parameters.RH2M[date],
      windSpeed: parameters.WS10M[date]
    }));
  }
}
```

## Services d'Accès Multi-Canaux

### Service Téléphonique (980)

```typescript
// API pour gestion des appels téléphoniques
class PhoneServiceAPI {
  async handleIncomingCall(phoneNumber: string, language: string) {
    const session = await this.createCallSession(phoneNumber, language);
    
    return {
      sessionId: session.id,
      welcomeMessage: this.getWelcomeMessage(language),
      availableOptions: [
        { key: '1', action: 'subscribe', label: 'Souscrire assurance' },
        { key: '2', action: 'claim', label: 'Déclarer sinistre' },
        { key: '3', action: 'status', label: 'Statut assurance' },
        { key: '4', action: 'payment', label: 'Paiement prime' }
      ],
      nextStep: 'waitForUserInput'
    };
  }

  async processUserInput(sessionId: string, input: string) {
    const session = await this.getCallSession(sessionId);
    
    switch (session.currentStep) {
      case 'language_selection':
        return this.handleLanguageSelection(session, input);
      case 'service_selection':
        return this.handleServiceSelection(session, input);
      case 'subscription_flow':
        return this.handleSubscriptionFlow(session, input);
      case 'claim_flow':
        return this.handleClaimFlow(session, input);
      default:
        return this.handleUnknownInput(session, input);
    }
  }
}
```

### Service SMS (980)

```typescript
// API pour gestion des SMS
class SMSServiceAPI {
  async handleIncomingSMS(phoneNumber: string, message: string) {
    const normalizedMessage = message.toUpperCase().trim();
    
    if (normalizedMessage.includes('MON ASSURANCE AGRICOLE')) {
      return this.initiateSMSSubscription(phoneNumber);
    }
    
    if (normalizedMessage.includes('SINISTRE')) {
      return this.initiateSMSClaim(phoneNumber);
    }
    
    if (normalizedMessage.includes('STATUT')) {
      return this.sendInsuranceStatus(phoneNumber);
    }
    
    return this.sendHelpMessage(phoneNumber);
  }

  private async initiateSMSSubscription(phoneNumber: string) {
    const steps = [
      'Bienvenue! Répondez 1=Français, 2=Fon, 3=Yoruba, 4=Bambara',
      'Votre région: 1=Cotonou, 2=Porto-Novo, 3=Parakou, 4=Natitingou, 5=Autre',
      'Culture: 1=Maïs, 2=Coton, 3=Arachide, 4=Igname, 5=Maraîchage',
      'Superficie (hectares): Tapez le nombre',
      'Analyse en cours via satellite...',
      'Prime: {amount} FCFA/mois. Répondez OUI pour confirmer',
      'Mobile Money: 1=MTN, 2=Orange, 3=Flooz',
      'Assurance activée! Confirmation envoyée.'
    ];

    await this.startSMSFlow(phoneNumber, steps);
    return { status: 'SMS_FLOW_STARTED', nextStep: 'LANGUAGE_SELECTION' };
  }
}
```

## Intégration Mobile Money

### MTN Mobile Money API

```typescript
// Simulation de l'API MTN MoMo
class MTNMoMoAPI {
  async requestToPay(amount: number, phoneNumber: string, externalId: string) {
    // Simulation de l'API MTN MoMo
    const transactionId = `mtn_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    
    return {
      transactionId,
      status: 'PENDING',
      amount,
      currency: 'XOF',
      phoneNumber,
      externalId,
      createdAt: new Date().toISOString()
    };
  }

  async getTransactionStatus(transactionId: string) {
    // Simulation du statut de transaction
    const statuses = ['PENDING', 'SUCCESSFUL', 'FAILED'];
    const randomStatus = statuses[Math.floor(Math.random() * statuses.length)];
    
    return {
      transactionId,
      status: randomStatus,
      updatedAt: new Date().toISOString()
    };
  }

  async payOut(amount: number, phoneNumber: string, reason: string) {
    // Simulation de paiement sortant (indemnisation)
    const payoutId = `payout_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    
    return {
      payoutId,
      status: 'PROCESSING',
      amount,
      phoneNumber,
      reason,
      estimatedCompletion: new Date(Date.now() + 2 * 60 * 60 * 1000).toISOString() // 2h
    };
  }
}
```

## Gestion des Erreurs et Fallback

```typescript
// Service de gestion des erreurs avec fallback intelligent
class APIErrorHandler {
  async executeWithFallback<T>(
    primaryAPI: () => Promise<T>,
    fallbackAPI: () => Promise<T>,
    defaultData: () => T
  ): Promise<{ data: T; source: string }> {
    
    try {
      const data = await primaryAPI();
      return { data, source: 'OpenEPI' };
    } catch (primaryError) {
      console.warn('Primary API failed, trying fallback:', primaryError.message);
      
      try {
        const data = await fallbackAPI();
        return { data, source: 'Fallback' };
      } catch (fallbackError) {
        console.error('Fallback API also failed:', fallbackError.message);
        
        const data = defaultData();
        return { data, source: 'Default' };
      }
    }
  }

  handleAPIError(error: any, apiName: string) {
    const errorTypes = {
      NETWORK_ERROR: 'Problème de connexion internet',
      RATE_LIMIT: 'Limite de requêtes atteinte',
      AUTH_ERROR: 'Erreur d\'authentification API',
      DATA_ERROR: 'Données invalides reçues',
      TIMEOUT: 'Délai d\'attente dépassé'
    };

    return {
      type: this.classifyError(error),
      message: errorTypes[this.classifyError(error)] || 'Erreur inconnue',
      apiName,
      timestamp: new Date().toISOString(),
      shouldRetry: this.shouldRetry(error),
      retryAfter: this.calculateRetryDelay(error)
    };
  }
}
```

Cette documentation API complète montre l'intégration prioritaire des services OpenEPI avec des fallbacks robustes, garantissant la disponibilité du service ClimInvest même en cas de problèmes avec les APIs principales.
