# Deployment Guide - ClimInvest

## Deployment Overview

This guide details the complete deployment of ClimInvest, including the mobile application, backend services, OpenEPI integration, and multi-channel support infrastructure (application, 980 call, SMS).

## Technical Prerequisites

### Development Environment

```bash
# Required versions
Node.js >= 18.0.0
npm >= 8.0.0
React Native CLI >= 2.0.1
Expo CLI >= 6.0.0

# Development tools
Android Studio (for Android)
Xcode (for iOS, macOS only)
VS Code with React Native extensions
```

### Required API Keys

```bash
# Essential environment variables
OPENEPI_API_KEY=your_openepi_key_here
OPENEPI_CLIENT_ID=your_client_id
OPENEPI_CLIENT_SECRET=your_client_secret
OPENEPI_BASE_URL=https://api.openepi.io/v1

# Fallback APIs
OPENWEATHER_API_KEY=your_openweather_key
NASA_POWER_API_KEY=optional_nasa_key

# Mobile Money services (simulation)
MTN_MOMO_API_KEY=simulation_key
ORANGE_MONEY_API_KEY=simulation_key
FLOOZ_API_KEY=simulation_key

# Communication services
SMS_SERVICE_API_KEY=your_sms_provider_key
VOICE_SERVICE_API_KEY=your_voice_provider_key
```

## Installation and Configuration

### 1. Cloning and Installation

```bash
# Cloner le repository
git clone https://github.com/climinvest/mobile-app.git
cd climinvest-mobile

# Installer les dépendances
npm install

# Configuration iOS (macOS uniquement)
cd ios && pod install && cd ..

# Vérifier l'installation
npx react-native doctor
```

### 2. Configuration des Variables d'Environnement

```bash
# Copier le fichier d'exemple
cp .env.example .env

# Éditer le fichier .env avec vos clés
nano .env
```

**Contenu du fichier .env:**
```bash
# OpenEPI Configuration (PRIORITAIRE)
OPENEPI_API_KEY=your_actual_openepi_key
OPENEPI_CLIENT_ID=your_client_id
OPENEPI_CLIENT_SECRET=your_client_secret
OPENEPI_BASE_URL=https://api.openepi.io/v1

# Fallback APIs
OPENWEATHER_API_KEY=your_openweather_key
OPEN_METEO_BASE_URL=https://api.open-meteo.com/v1

# Mobile Money (Simulation pour développement)
MTN_MOMO_BASE_URL=https://sandbox.momodeveloper.mtn.com
ORANGE_MONEY_BASE_URL=https://api.orange.com/orange-money-webpay
FLOOZ_BASE_URL=https://api.flooz.app

# Communication Services
SMS_PROVIDER_URL=https://api.africas-talking.com/version1
VOICE_PROVIDER_URL=https://api.twilio.com/2010-04-01
CALL_CENTER_NUMBER=980

# Application Configuration
APP_ENV=development
DEBUG_MODE=true
CACHE_DURATION=3600
```

### 3. Configuration OpenEPI

```typescript
// src/config/openEpiConfig.ts
export const OPENEPI_CONFIG = {
  apiKey: process.env.OPENEPI_API_KEY,
  clientId: process.env.OPENEPI_CLIENT_ID,
  clientSecret: process.env.OPENEPI_CLIENT_SECRET,
  baseURL: process.env.OPENEPI_BASE_URL,
  
  // Configuration des clients OpenEPI
  clients: {
    weather: {
      timeout: 30000,
      retryAttempts: 3,
      cacheDuration: 3600 // 1 heure
    },
    soil: {
      timeout: 45000,
      retryAttempts: 2,
      cacheDuration: 86400 // 24 heures
    },
    flood: {
      timeout: 20000,
      retryAttempts: 3,
      cacheDuration: 1800 // 30 minutes
    },
    cropHealth: {
      timeout: 60000,
      retryAttempts: 2,
      cacheDuration: 7200 // 2 heures
    }
  },
  
  // Seuils de déclenchement automatique
  triggers: {
    drought: {
      ndviThreshold: 0.25,
      consecutiveDryDays: 21,
      temperatureThreshold: 38
    },
    flood: {
      riskLevelThreshold: 'HIGH',
      probabilityThreshold: 0.7
    },
    cropStress: {
      ndviCritical: 0.2,
      persistenceDays: 14
    }
  }
};
```

## Déploiement Mobile

### 1. Build Android

```bash
# Build de développement
npx expo run:android

# Build de production
npx expo build:android --type apk

# Build pour Google Play Store
npx expo build:android --type app-bundle
```

### 2. Build iOS

```bash
# Build de développement (macOS uniquement)
npx expo run:ios

# Build de production
npx expo build:ios --type archive

# Configuration pour App Store
npx expo build:ios --type app-store
```

### 3. Configuration des Stores

**Google Play Store:**
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  <uses-permission android:name="android.permission.SEND_SMS" />
  <uses-permission android:name="android.permission.CALL_PHONE" />
  
  <application
    android:name=".MainApplication"
    android:label="@string/app_name"
    android:icon="@mipmap/ic_launcher"
    android:allowBackup="false"
    android:theme="@style/AppTheme">
    
    <!-- Configuration pour accessibilité -->
    <meta-data
      android:name="android.app.accessibility"
      android:resource="@xml/accessibility_service_config" />
  </application>
</manifest>
```

**App Store (iOS):**
```xml
<!-- ios/ClimInvest/Info.plist -->
<dict>
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>ClimInvest utilise votre localisation pour fournir des données climatiques précises et calculer vos primes d'assurance.</string>
  
  <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
  <string>L'accès à votre localisation permet de surveiller vos cultures et déclencher automatiquement les indemnisations.</string>
  
  <key>NSMicrophoneUsageDescription</key>
  <string>Le microphone est utilisé pour l'interface vocale en langues locales.</string>
  
  <key>NSSpeechRecognitionUsageDescription</key>
  <string>La reconnaissance vocale permet l'utilisation en langues locales (fon, yoruba, bambara).</string>
</dict>
```

## Infrastructure Backend

### 1. Services de Communication

```typescript
// services/communicationInfrastructure.ts
export class CommunicationInfrastructure {
  private smsService: SMSService;
  private voiceService: VoiceService;
  private callCenterService: CallCenterService;

  constructor() {
    this.initializeServices();
  }

  // Service SMS pour numéro 980
  async setupSMSService() {
    return {
      provider: 'Africa\'s Talking',
      shortCode: '980',
      keywords: ['MON ASSURANCE AGRICOLE', 'SINISTRE', 'STATUT'],
      autoResponses: {
        'MON ASSURANCE AGRICOLE': this.initiateSMSSubscription,
        'SINISTRE': this.initiateSMSClaim,
        'STATUT': this.sendInsuranceStatus
      },
      languages: ['fr', 'fon', 'yoruba', 'bambara'],
      dailyLimit: 10000 // SMS par jour
    };
  }

  // Service d'appels pour numéro 980
  async setupCallCenterService() {
    return {
      provider: 'Twilio',
      phoneNumber: '+229980',
      ivr: {
        welcomeMessage: 'Bienvenue chez ClimInvest. Appuyez 1 pour français, 2 pour fon, 3 pour yoruba, 4 pour bambara',
        menuOptions: {
          '1': 'Souscrire une assurance',
          '2': 'Déclarer un sinistre',
          '3': 'Statut de mon assurance',
          '4': 'Parler à un conseiller'
        }
      },
      agents: {
        capacity: 50, // Conseillers simultanés
        languages: ['fr', 'fon', 'yoruba', 'bambara'],
        workingHours: '06:00-22:00 GMT',
        escalationRules: {
          waitTime: 120, // secondes avant escalade
          complexCases: 'supervisor',
          technicalIssues: 'tech_support'
        }
      }
    };
  }
}
```

### 2. Intégration Mobile Money

```typescript
// services/mobileMoneyIntegration.ts
export class MobileMoneyIntegration {
  private providers: Map<string, MobileMoneyProvider>;

  constructor() {
    this.initializeProviders();
  }

  private initializeProviders() {
    // MTN Mobile Money
    this.providers.set('mtn', {
      name: 'MTN Mobile Money',
      apiUrl: process.env.MTN_MOMO_BASE_URL,
      apiKey: process.env.MTN_MOMO_API_KEY,
      supportedCountries: ['BJ', 'BF', 'CI', 'GH', 'GN', 'ML', 'NE', 'SN', 'TG'],
      transactionLimits: {
        daily: 500000, // FCFA
        monthly: 2000000 // FCFA
      }
    });

    // Orange Money
    this.providers.set('orange', {
      name: 'Orange Money',
      apiUrl: process.env.ORANGE_MONEY_BASE_URL,
      apiKey: process.env.ORANGE_MONEY_API_KEY,
      supportedCountries: ['BF', 'CI', 'ML', 'NE', 'SN'],
      transactionLimits: {
        daily: 300000, // FCFA
        monthly: 1500000 // FCFA
      }
    });

    // Flooz
    this.providers.set('flooz', {
      name: 'Flooz',
      apiUrl: process.env.FLOOZ_BASE_URL,
      apiKey: process.env.FLOOZ_API_KEY,
      supportedCountries: ['BJ', 'TG'],
      transactionLimits: {
        daily: 200000, // FCFA
        monthly: 1000000 // FCFA
      }
    });
  }

  async processPayment(
    provider: string,
    phoneNumber: string,
    amount: number,
    reference: string
  ) {
    const providerConfig = this.providers.get(provider);
    if (!providerConfig) {
      throw new Error(`Provider ${provider} not supported`);
    }

    // Simulation de paiement pour développement
    if (process.env.APP_ENV === 'development') {
      return this.simulatePayment(provider, phoneNumber, amount, reference);
    }

    // Intégration réelle en production
    return this.executeRealPayment(providerConfig, phoneNumber, amount, reference);
  }
}
```

## Monitoring et Observabilité

### 1. Monitoring OpenEPI

```typescript
// services/openEpiMonitoring.ts
export class OpenEpiMonitoring {
  private metrics: Map<string, any> = new Map();

  async monitorAPIHealth() {
    const services = ['weather', 'soil', 'flood', 'cropHealth', 'geocoder'];
    const healthStatus = new Map();

    for (const service of services) {
      try {
        const startTime = Date.now();
        await this.testServiceEndpoint(service);
        const responseTime = Date.now() - startTime;

        healthStatus.set(service, {
          status: 'healthy',
          responseTime,
          lastChecked: new Date().toISOString()
        });
      } catch (error) {
        healthStatus.set(service, {
          status: 'unhealthy',
          error: error.message,
          lastChecked: new Date().toISOString()
        });
      }
    }

    return {
      overall: this.calculateOverallHealth(healthStatus),
      services: Object.fromEntries(healthStatus),
      recommendations: this.generateHealthRecommendations(healthStatus)
    };
  }

  async trackUsageMetrics() {
    return {
      dailyRequests: this.metrics.get('daily_requests') || 0,
      successRate: this.calculateSuccessRate(),
      averageResponseTime: this.calculateAverageResponseTime(),
      errorRate: this.calculateErrorRate(),
      quotaUsage: await this.checkQuotaUsage()
    };
  }
}
```

### 2. Alertes et Notifications

```typescript
// services/alertingSystem.ts
export class AlertingSystem {
  async setupAlerts() {
    return {
      openEpiDowntime: {
        condition: 'OpenEPI service unavailable > 5 minutes',
        action: 'Switch to fallback APIs + notify admin',
        severity: 'critical'
      },
      
      highErrorRate: {
        condition: 'Error rate > 10% over 15 minutes',
        action: 'Investigate + scale resources',
        severity: 'warning'
      },
      
      quotaExceeded: {
        condition: 'OpenEPI quota > 90%',
        action: 'Throttle requests + notify admin',
        severity: 'warning'
      },
      
      paymentFailures: {
        condition: 'Mobile Money failures > 20%',
        action: 'Check provider status + manual intervention',
        severity: 'critical'
      }
    };
  }
}
```

## Tests et Validation

### 1. Tests d'Intégration OpenEPI

```bash
# Tests des services OpenEPI
npm run test:openepi

# Tests de fallback
npm run test:fallback

# Tests end-to-end
npm run test:e2e

# Tests de performance
npm run test:performance
```

### 2. Tests des Canaux d'Accès

```typescript
// tests/accessChannels.test.ts
describe('Access Channels Integration', () => {
  test('SMS subscription flow', async () => {
    const smsService = new SMSService();
    const response = await smsService.handleIncomingSMS(
      '+22961234567',
      'MON ASSURANCE AGRICOLE'
    );
    
    expect(response.status).toBe('SMS_FLOW_STARTED');
    expect(response.nextStep).toBe('LANGUAGE_SELECTION');
  });

  test('Phone call flow', async () => {
    const callService = new CallCenterService();
    const session = await callService.handleIncomingCall(
      '+22961234567',
      'fr'
    );
    
    expect(session.welcomeMessage).toContain('Bienvenue chez ClimInvest');
    expect(session.availableOptions).toHaveLength(4);
  });

  test('Mobile app registration', async () => {
    const authService = new AuthService();
    const user = await authService.registerUser({
      name: 'Test Farmer',
      phone: '+22961234567',
      location: { latitude: 6.3703, longitude: 2.3912 },
      cropType: 'maize'
    });
    
    expect(user.id).toBeDefined();
    expect(user.isActive).toBe(true);
  });
});
```

## Déploiement en Production

### 1. Checklist de Déploiement

```bash
# Vérifications pré-déploiement
□ Clés OpenEPI configurées et testées
□ Services de fallback opérationnels
□ Mobile Money intégrations testées
□ Centre d'appels 980 configuré
□ SMS service 980 opérationnel
□ Tests d'accessibilité validés
□ Performance tests passés
□ Monitoring configuré
□ Alertes configurées
□ Documentation à jour

# Commandes de déploiement
npm run build:production
npm run deploy:android
npm run deploy:ios
npm run deploy:backend
```

### 2. Configuration Production

```typescript
// config/production.ts
export const PRODUCTION_CONFIG = {
  openEpi: {
    baseURL: 'https://api.openepi.io/v1',
    timeout: 30000,
    retryAttempts: 3,
    rateLimiting: true
  },
  
  communication: {
    smsProvider: 'Africa\'s Talking',
    voiceProvider: 'Twilio',
    shortCode: '980',
    maxConcurrentCalls: 100
  },
  
  mobileMoneyProviders: ['mtn', 'orange', 'flooz'],
  
  monitoring: {
    enabled: true,
    logLevel: 'info',
    metricsCollection: true,
    alerting: true
  },
  
  security: {
    encryption: 'AES-256-GCM',
    tokenExpiry: 3600,
    rateLimiting: {
      windowMs: 900000, // 15 minutes
      max: 100 // requests per window
    }
  }
};
```

Ce guide de déploiement garantit une mise en production robuste de ClimInvest avec tous les canaux d'accès (application, 980, SMS) et l'intégration complète des APIs OpenEPI.
