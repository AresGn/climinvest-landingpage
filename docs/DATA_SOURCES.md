# Data Sources - OpenEPI Integration

## Overview

ClimInvest primarily relies on OpenEPI APIs to provide reliable climate, soil, and agricultural data to farmers in West Africa. This integration enables automatic payout triggering and precise credit scoring.

## OpenEPI APIs - System Core

### 1. WeatherClient - Meteorological Data

```typescript
// OpenEPI WeatherClient integration
import { WeatherClient } from 'openepi-client';

const weatherClient = new WeatherClient({
  apiKey: process.env.OPENEPI_API_KEY,
  baseURL: 'https://api.openepi.io/v1'
});

// Usage for automatic triggering
export class WeatherTriggerService {
  async evaluateWeatherTriggers(lat: number, lon: number) {
    try {
      // Detailed OpenEPI forecasts
      const forecast = await weatherClient.getLocationForecast(lat, lon);
      const summary = await weatherClient.getSummaryForecast(lat, lon);

      return {
        current_conditions: forecast.current,
        daily_forecast: forecast.daily,
        weather_alerts: summary.alerts,
        trigger_evaluation: this.evaluateTriggers(forecast, summary)
      };
    } catch (error) {
      console.error('❌ OpenEPI WeatherClient Error:', error);
      throw error;
    }
  }

  private evaluateTriggers(forecast: any, summary: any) {
    return {
      drought_risk: this.calculateDroughtRisk(forecast),
      storm_risk: this.calculateStormRisk(summary.alerts),
      temperature_extreme: this.evaluateTemperatureExtremes(forecast.daily)
    };
  }
}
```

**Data Provided:**
- Current temperature and 7-day forecasts
- Historical and forecast precipitation
- Relative humidity and wind speed
- Automatic weather alerts
- Drought and water stress indices

**Usage in ClimInvest:**
- Automatic drought compensation triggering
- Proactive alerts to farmers
- Climate risk-based premium calculation
- Personalized agricultural recommendations

### 2. SoilClient - Soil Analysis

```typescript
// OpenEPI SoilClient integration
import { SoilClient } from 'openepi-client';

const soilClient = new SoilClient({
  apiKey: process.env.OPENEPI_API_KEY
});

export class SoilAnalysisService {
  async getComprehensiveSoilData(lat: number, lon: number) {
    try {
      // Complete soil data via OpenEPI
      const soilType = await soilClient.getSoilType(lat, lon);
      const soilProperties = await soilClient.getSoilProperties(lat, lon);

      return {
        soil_type: soilType,
        properties: {
          ph_level: soilProperties.ph,
          organic_matter: soilProperties.organicMatter,
          water_retention: soilProperties.waterRetention,
          fertility_index: soilProperties.fertilityIndex,
          erosion_risk: soilProperties.erosionRisk
        },
        quality_score: this.calculateSoilQualityScore(soilProperties),
        crop_suitability: this.assessCropSuitability(soilType, soilProperties),
        recommendations: this.generateSoilRecommendations(soilProperties)
      };
    } catch (error) {
      console.error('❌ OpenEPI SoilClient Error:', error);
      throw error;
    }
  }

  private calculateSoilQualityScore(properties: any): number {
    // Scoring algorithm based on OpenEPI properties
    const scores = {
      ph: this.scorePH(properties.ph),
      organic: this.scoreOrganicMatter(properties.organicMatter),
      water: this.scoreWaterRetention(properties.waterRetention),
      fertility: properties.fertilityIndex
    };

    return (scores.ph + scores.organic + scores.water + scores.fertility) / 4;
  }
}
```

**Data Provided:**
- Soil type and classification
- pH, organic matter, water retention
- Fertility index and erosion risk
- Crop suitability by crop type
- Soil improvement recommendations

**Usage in ClimInvest:**
- Soil quality-based credit scoring
- Personalized agricultural recommendations
- Adjusted insurance premium calculation
- Yield potential assessment

### 3. FloodClient - Flood Detection

```typescript
// OpenEPI FloodClient integration
import { FloodClient } from 'openepi-client';

const floodClient = new FloodClient({
  apiKey: process.env.OPENEPI_API_KEY
});

export class FloodMonitoringService {
  async monitorFloodRisk(lat: number, lon: number) {
    try {
      // Continuous flood monitoring
      const floodRisk = await floodClient.getFloodRisk(lat, lon);
      const historicalFloods = await floodClient.getHistoricalFloods(lat, lon);

      return {
        current_risk_level: floodRisk.riskLevel,
        probability_7days: floodRisk.probability,
        water_level_trend: floodRisk.waterLevelTrend,
        historical_events: historicalFloods,
        trigger_threshold: this.calculateFloodTrigger(floodRisk),
        evacuation_zones: floodRisk.evacuationZones
      };
    } catch (error) {
      console.error('❌ OpenEPI FloodClient Error:', error);
      throw error;
    }
  }

  async checkAutomaticTriggers(policies: InsurancePolicy[]) {
    for (const policy of policies) {
      const floodData = await this.monitorFloodRisk(
        policy.location.latitude,
        policy.location.longitude
      );

      if (floodData.current_risk_level === 'HIGH' &&
          floodData.probability_7days > 0.7) {

        await this.triggerFloodPayout(policy, floodData);
      }
    }
  }
}
```

**Data Provided:**
- Real-time flood risk level
- 7-day flood probability
- Water level trend
- Historical floods by area
- Recommended evacuation zones

**Usage in ClimInvest:**
- Automatic flood compensation triggering
- Preventive alerts to farmers
- Risk zone mapping
- Geographic premium adjustment

### 4. CropHealthClient - Surveillance des Cultures

```typescript
// Intégration OpenEPI CropHealthClient
import { CropHealthClient } from 'openepi-client';

const cropHealthClient = new CropHealthClient({
  apiKey: process.env.OPENEPI_API_KEY
});

export class CropMonitoringService {
  async monitorCropHealth(lat: number, lon: number, cropType: string) {
    try {
      // Surveillance satellite des cultures
      const healthData = await cropHealthClient.getCropHealth(lat, lon, cropType);
      const ndviData = await cropHealthClient.getNDVITimeSeries(lat, lon);
      
      return {
        current_ndvi: healthData.ndvi,
        health_status: this.interpretHealthStatus(healthData.ndvi),
        stress_indicators: {
          water_stress: healthData.waterStress,
          nutrient_deficiency: healthData.nutrientDeficiency,
          pest_disease_risk: healthData.pestDiseaseRisk
        },
        ndvi_trend: this.analyzeNDVITrend(ndviData),
        growth_stage: healthData.growthStage,
        yield_prediction: this.predictYield(healthData, cropType),
        recommendations: this.generateCropRecommendations(healthData)
      };
    } catch (error) {
      console.error('❌ OpenEPI CropHealthClient Error:', error);
      throw error;
    }
  }

  private interpretHealthStatus(ndvi: number): string {
    if (ndvi > 0.7) return 'Excellent';
    if (ndvi > 0.5) return 'Bon';
    if (ndvi > 0.3) return 'Moyen';
    if (ndvi > 0.2) return 'Faible';
    return 'Critique';
  }

  async evaluateCropStressTriggers(policies: InsurancePolicy[]) {
    for (const policy of policies) {
      const cropData = await this.monitorCropHealth(
        policy.location.latitude,
        policy.location.longitude,
        policy.cropType
      );

      // Déclenchement si NDVI critique pendant 14 jours
      if (cropData.current_ndvi < 0.25 && 
          this.isStressPersistent(cropData.ndvi_trend, 14)) {
        
        await this.triggerCropStressPayout(policy, cropData);
      }
    }
  }
}
```

**Données Fournies:**
- Indice NDVI actuel et historique
- Statut de santé des cultures
- Indicateurs de stress (eau, nutriments, maladies)
- Stade de croissance des cultures
- Prédictions de rendement
- Recommandations agricoles spécifiques

**Utilisation dans ClimInvest:**
- Déclenchement automatique pour stress des cultures
- Alertes précoces de problèmes agricoles
- Conseils agricoles personnalisés
- Évaluation du potentiel de rendement

### 5. GeocoderClient - Services de Géolocalisation

```typescript
// Intégration OpenEPI GeocoderClient
import { GeocoderClient } from 'openepi-client';

const geocoderClient = new GeocoderClient({
  apiKey: process.env.OPENEPI_API_KEY
});

export class LocationService {
  async reverseGeocode(lat: number, lon: number) {
    try {
      const locationData = await geocoderClient.reverseGeocode(lat, lon);
      
      return {
        country: locationData.country,
        region: locationData.administrativeArea,
        district: locationData.locality,
        village: locationData.subLocality,
        climate_zone: locationData.climateZone,
        agricultural_zone: locationData.agriculturalZone,
        risk_profile: await this.getLocationRiskProfile(lat, lon)
      };
    } catch (error) {
      console.error('❌ OpenEPI GeocoderClient Error:', error);
      throw error;
    }
  }

  async getLocationRiskProfile(lat: number, lon: number) {
    // Profil de risque basé sur la localisation OpenEPI
    const [weather, soil, flood] = await Promise.all([
      weatherClient.getLocationForecast(lat, lon),
      soilClient.getSoilProperties(lat, lon),
      floodClient.getFloodRisk(lat, lon)
    ]);

    return {
      drought_risk: this.calculateDroughtRisk(weather),
      flood_risk: flood.riskLevel,
      soil_quality: this.calculateSoilQuality(soil),
      overall_risk: this.calculateOverallRisk(weather, soil, flood)
    };
  }
}
```

## Services de Fallback

### Open-Meteo (Fallback Météo)

```typescript
// Fallback météorologique si OpenEPI indisponible
export class OpenMeteoFallbackService {
  async getCurrentWeather(lat: number, lon: number) {
    try {
      const response = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,precipitation,wind_speed_10m&daily=temperature_2m_max,temperature_2m_min,precipitation_sum&timezone=auto`
      );
      
      const data = await response.json();
      
      // Conversion au format OpenEPI pour compatibilité
      return this.convertToOpenEPIFormat(data);
    } catch (error) {
      console.error('❌ Open-Meteo Fallback Error:', error);
      throw error;
    }
  }
}
```

### NASA POWER (Données Historiques)

```typescript
// Service complémentaire pour données historiques
export class NASAPowerService {
  async getHistoricalWeather(lat: number, lon: number, years: number) {
    const endDate = new Date();
    const startDate = new Date();
    startDate.setFullYear(endDate.getFullYear() - years);

    try {
      const response = await fetch(
        `https://power.larc.nasa.gov/api/temporal/daily/point?parameters=T2M,PRECTOTCORR&community=AG&longitude=${lon}&latitude=${lat}&start=${this.formatDate(startDate)}&end=${this.formatDate(endDate)}&format=JSON`
      );
      
      const data = await response.json();
      return this.processNASAData(data);
    } catch (error) {
      console.error('❌ NASA POWER Error:', error);
      throw error;
    }
  }
}
```

## Architecture de Données Hybride

```typescript
// Service principal avec fallback intelligent
export class HybridDataService {
  private openEpiService: OpenEpiService;
  private fallbackServices: FallbackServices;
  private cacheService: CacheService;

  async getWeatherData(lat: number, lon: number) {
    // 1. Vérifier le cache
    const cachedData = await this.cacheService.get(`weather_${lat}_${lon}`);
    if (cachedData && this.isCacheValid(cachedData)) {
      return cachedData;
    }

    // 2. Essayer OpenEPI (priorité)
    try {
      const openEpiData = await this.openEpiService.getCurrentWeather(lat, lon);
      await this.cacheService.set(`weather_${lat}_${lon}`, openEpiData, 3600); // 1h cache
      return { source: 'OpenEPI', data: openEpiData };
    } catch (openEpiError) {
      console.warn('⚠️ OpenEPI indisponible, utilisation fallback');
    }

    // 3. Fallback Open-Meteo
    try {
      const fallbackData = await this.fallbackServices.openMeteo.getCurrentWeather(lat, lon);
      await this.cacheService.set(`weather_${lat}_${lon}`, fallbackData, 1800); // 30min cache
      return { source: 'Open-Meteo', data: fallbackData };
    } catch (fallbackError) {
      console.error('❌ Tous les services météo échoués');
      
      // 4. Données simulées en dernier recours
      return { 
        source: 'Simulated', 
        data: this.generateDefaultWeatherData(lat, lon) 
      };
    }
  }
}
```

## Métriques de Qualité des Données

```typescript
// Surveillance de la qualité des données OpenEPI
export class DataQualityMonitor {
  private metrics = {
    openEpiAvailability: 0,
    dataFreshness: 0,
    accuracyScore: 0,
    completeness: 0
  };

  async monitorDataQuality() {
    const testLocations = [
      { lat: 6.3703, lon: 2.3912 }, // Cotonou
      { lat: 9.3077, lon: 2.6056 }, // Parakou
      { lat: 6.4969, lon: 2.6283 }  // Porto-Novo
    ];

    for (const location of testLocations) {
      await this.testOpenEPIServices(location);
    }

    return this.metrics;
  }

  private async testOpenEPIServices(location: {lat: number, lon: number}) {
    const services = ['weather', 'soil', 'flood', 'cropHealth'];
    let successCount = 0;

    for (const service of services) {
      try {
        await this.testService(service, location);
        successCount++;
      } catch (error) {
        console.error(`❌ Service ${service} failed for location ${location.lat}, ${location.lon}`);
      }
    }

    this.metrics.openEpiAvailability = (successCount / services.length) * 100;
  }
}
```

Cette architecture de données garantit une intégration robuste avec OpenEPI tout en maintenant la résilience grâce aux services de fallback, permettant à ClimInvest de fonctionner de manière fiable même en cas d'indisponibilité temporaire des services principaux.
