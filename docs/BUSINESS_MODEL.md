# Business Model - ClimInvest

## Model Overview

ClimInvest operates on a sustainable climate micro-insurance model, combining maximum accessibility and economic viability. The model relies on OpenEPI data to optimize costs and automate processes.

## Pricing Structure

### Monthly Premiums by Crop and Risk Zone

| Crop | Low Risk Zone | Medium Risk Zone | High Risk Zone | Max Coverage |
|---------|-------------------|-------------------|------------------|----------------|
| **Corn** | 400 FCFA/month | 600 FCFA/month | 800 FCFA/month | 30,000 FCFA |
| **Cotton** | 350 FCFA/month | 550 FCFA/month | 750 FCFA/month | 25,000 FCFA |
| **Peanut** | 300 FCFA/month | 500 FCFA/month | 700 FCFA/month | 20,000 FCFA |
| **Yam** | 450 FCFA/month | 650 FCFA/month | 850 FCFA/month | 35,000 FCFA |
| **Market Gardening** | 250 FCFA/month | 400 FCFA/month | 600 FCFA/month | 15,000 FCFA |

### OpenEPI-Based Premium Calculation

```typescript
// Algorithme de pricing basé sur les données OpenEPI
export class PremiumCalculator {
  async calculatePremium(
    location: { lat: number; lon: number },
    cropType: string,
    farmSize: number
  ) {
    // Collecte de données OpenEPI pour évaluation du risque
    const [weatherRisk, soilQuality, floodRisk, historicalData] = await Promise.all([
      this.openEpiService.getWeatherRisk(location.lat, location.lon),
      this.openEpiService.getSoilQuality(location.lat, location.lon),
      this.openEpiService.getFloodRisk(location.lat, location.lon),
      this.openEpiService.getHistoricalDisasters(location.lat, location.lon, 10)
    ]);

    // Calcul des facteurs de risque
    const riskFactors = {
      drought: this.calculateDroughtRisk(weatherRisk, historicalData),
      flood: this.calculateFloodRisk(floodRisk, historicalData),
      soil: this.calculateSoilRisk(soilQuality),
      climate: this.calculateClimateRisk(weatherRisk)
    };

    // Prime de base par culture
    const basePremium = this.getBasePremium(cropType);
    
    // Multiplicateur de risque composite
    const riskMultiplier = this.calculateRiskMultiplier(riskFactors);
    
    // Ajustement par taille d'exploitation
    const sizeAdjustment = this.calculateSizeAdjustment(farmSize);
    
    const monthlyPremium = Math.round(basePremium * riskMultiplier * sizeAdjustment);
    
    return {
      monthlyPremium,
      annualPremium: monthlyPremium * 12,
      coverageAmount: this.calculateCoverageAmount(cropType, farmSize),
      riskLevel: this.determineRiskLevel(riskMultiplier),
      factors: riskFactors,
      dataSource: 'OpenEPI'
    };
  }

  private calculateRiskMultiplier(factors: RiskFactors): number {
    // Pondération des facteurs de risque
    const weights = {
      drought: 0.35,
      flood: 0.25,
      soil: 0.25,
      climate: 0.15
    };

    const compositeRisk = (
      factors.drought * weights.drought +
      factors.flood * weights.flood +
      factors.soil * weights.soil +
      factors.climate * weights.climate
    );

    // Multiplicateur entre 0.8 (faible risque) et 2.0 (haut risque)
    return Math.max(0.8, Math.min(2.0, 0.8 + (compositeRisk * 1.2)));
  }
}
```

## Répartition des Revenus

### Allocation des Primes Collectées

```typescript
// Modèle de répartition des fonds
export const FUND_ALLOCATION = {
  // 80% - Fonds d'indemnisation et réserves
  claimsFund: 0.65,           // 65% - Paiements directs des sinistres
  catastropheReserve: 0.15,   // 15% - Réserve pour catastrophes majeures
  
  // 15% - Coûts techniques et opérationnels
  openEpiLicenses: 0.05,      // 5% - Licences APIs OpenEPI
  technologyInfra: 0.04,      // 4% - Infrastructure technique
  mobileMoneyFees: 0.03,      // 3% - Frais Mobile Money
  callCenterOps: 0.03,        // 3% - Opération centre d'appels 980
  
  // 5% - Croissance et développement
  productDevelopment: 0.02,   // 2% - Développement nouvelles fonctionnalités
  marketExpansion: 0.02,      // 2% - Expansion géographique
  partnerCommissions: 0.01    // 1% - Commissions partenaires
};

// Calcul de la viabilité financière
export class FinancialViabilityCalculator {
  calculateBreakEven(userBase: number, averagePremium: number) {
    const monthlyRevenue = userBase * averagePremium;
    const annualRevenue = monthlyRevenue * 12;
    
    const costs = {
      claims: annualRevenue * FUND_ALLOCATION.claimsFund,
      reserves: annualRevenue * FUND_ALLOCATION.catastropheReserve,
      technology: annualRevenue * (
        FUND_ALLOCATION.openEpiLicenses +
        FUND_ALLOCATION.technologyInfra +
        FUND_ALLOCATION.mobileMoneyFees +
        FUND_ALLOCATION.callCenterOps
      ),
      growth: annualRevenue * (
        FUND_ALLOCATION.productDevelopment +
        FUND_ALLOCATION.marketExpansion +
        FUND_ALLOCATION.partnerCommissions
      )
    };

    return {
      userBase,
      monthlyRevenue,
      annualRevenue,
      costs,
      isViable: annualRevenue > Object.values(costs).reduce((a, b) => a + b, 0),
      breakEvenUsers: this.calculateMinimumUsers(averagePremium)
    };
  }

  private calculateMinimumUsers(averagePremium: number): number {
    // Coûts fixes minimums
    const fixedCosts = {
      openEpiLicenses: 50000 * 12,      // 50k FCFA/mois
      infrastructure: 100000 * 12,      // 100k FCFA/mois
      callCenter: 200000 * 12,          // 200k FCFA/mois
      development: 150000 * 12          // 150k FCFA/mois
    };

    const totalFixedCosts = Object.values(fixedCosts).reduce((a, b) => a + b, 0);
    const annualPremiumPerUser = averagePremium * 12;
    const netRevenuePerUser = annualPremiumPerUser * 0.2; // 20% marge après sinistres

    return Math.ceil(totalFixedCosts / netRevenuePerUser);
  }
}
```

## Canaux de Distribution et Coûts

### Structure des Coûts par Canal d'Accès

```typescript
// Coûts par canal d'acquisition et service
export const CHANNEL_COSTS = {
  mobileApp: {
    acquisitionCost: 500,        // FCFA par utilisateur acquis
    serviceCost: 50,             // FCFA par utilisateur par mois
    conversionRate: 0.15,        // 15% des téléchargements deviennent clients
    retentionRate: 0.85          // 85% de rétention annuelle
  },
  
  phoneService980: {
    acquisitionCost: 1200,       // FCFA par utilisateur (temps conseiller)
    serviceCost: 200,            // FCFA par utilisateur par mois
    conversionRate: 0.45,        // 45% des appels deviennent clients
    retentionRate: 0.90          // 90% de rétention (service personnalisé)
  },
  
  smsService980: {
    acquisitionCost: 300,        // FCFA par utilisateur
    serviceCost: 100,            // FCFA par utilisateur par mois
    conversionRate: 0.25,        // 25% des SMS deviennent clients
    retentionRate: 0.80          // 80% de rétention
  },
  
  communityHelper: {
    acquisitionCost: 200,        // FCFA par utilisateur (commission helper)
    serviceCost: 75,             // FCFA par utilisateur par mois
    conversionRate: 0.60,        // 60% conversion (confiance communautaire)
    retentionRate: 0.95          // 95% rétention (support local)
  }
};

// Optimisation du mix de canaux
export class ChannelOptimizer {
  calculateOptimalChannelMix(targetUsers: number, budget: number) {
    const channels = Object.entries(CHANNEL_COSTS);
    
    // Calcul du coût total par utilisateur sur 2 ans
    const channelROI = channels.map(([name, costs]) => {
      const totalAcquisitionCost = costs.acquisitionCost / costs.conversionRate;
      const annualServiceCost = costs.serviceCost * 12;
      const twoYearServiceCost = annualServiceCost * 2 * costs.retentionRate;
      const totalCostPerUser = totalAcquisitionCost + twoYearServiceCost;
      
      return {
        channel: name,
        costPerUser: totalCostPerUser,
        conversionRate: costs.conversionRate,
        retentionRate: costs.retentionRate,
        roi: this.calculateChannelROI(totalCostPerUser, costs.retentionRate)
      };
    });

    // Tri par ROI décroissant
    channelROI.sort((a, b) => b.roi - a.roi);
    
    return {
      recommendedMix: this.allocateBudget(channelROI, budget, targetUsers),
      channelAnalysis: channelROI
    };
  }
}
```

## Projections Financières

### Scénarios de Croissance

```typescript
// Modèle de projection financière
export class FinancialProjections {
  generateProjections(years: number = 5) {
    const scenarios = {
      conservative: { growthRate: 0.5, churnRate: 0.15 },
      realistic: { growthRate: 0.8, churnRate: 0.12 },
      optimistic: { growthRate: 1.2, churnRate: 0.08 }
    };

    return Object.entries(scenarios).map(([scenario, params]) => ({
      scenario,
      projections: this.calculateYearlyProjections(years, params)
    }));
  }

  private calculateYearlyProjections(years: number, params: any) {
    let users = 5000; // Base initiale
    const averagePremium = 600; // FCFA/mois
    const projections = [];

    for (let year = 1; year <= years; year++) {
      // Croissance des utilisateurs
      const newUsers = users * params.growthRate;
      const churnedUsers = users * params.churnRate;
      users = users + newUsers - churnedUsers;

      // Revenus et coûts
      const annualRevenue = users * averagePremium * 12;
      const claimsPaid = annualRevenue * 0.65;
      const operatingCosts = annualRevenue * 0.15;
      const netIncome = annualRevenue - claimsPaid - operatingCosts;

      projections.push({
        year,
        users: Math.round(users),
        annualRevenue,
        claimsPaid,
        operatingCosts,
        netIncome,
        profitMargin: (netIncome / annualRevenue) * 100
      });
    }

    return projections;
  }
}
```

### Seuil de Rentabilité - Basé sur l'Analyse Économique

### Données de Référence du Document Source

Selon l'analyse économique détaillée :
- **Seuil de rentabilité** : **50,000 agriculteurs assurés**
- **Répartition des fonds** : 80% indemnisation, 15% coûts techniques, 5% croissance
- **Fonds de réserve gouvernemental** : 10% des primes pour catastrophes majeures

### Calcul de Rentabilité Détaillé

```typescript
// Analyse du seuil de rentabilité basée sur le modèle économique
export const BREAKEVEN_ANALYSIS = {
  minimumUsers: 50000,           // Seuil documenté dans l'analyse économique
  timeToBreakeven: 24,           // 24 mois (Phase 2 du déploiement)

  revenueModel: {
    averagePremium: 600,         // FCFA/mois (moyenne pondérée)
    monthlyRevenue: 30000000,    // 50k users × 600 FCFA = 30M FCFA/mois
    annualRevenue: 360000000     // 360M FCFA/an à la rentabilité
  },

  costStructure: {
    claimsFund: 0.80,            // 80% des primes → indemnisations
    technicalCosts: 0.15,        // 15% → coûts techniques (OpenEPI, SMS, etc.)
    growthReserves: 0.05,        // 5% → croissance et réserves
    governmentReserve: 0.10      // 10% fonds gouvernemental catastrophes
  },

  criticalMetrics: {
    claimRatio: 0.65,            // Maximum 65% en sinistres normaux
    catastropheReserve: 0.15,    // 15% réserve catastrophes (80%-65%)
    operationalMargin: 0.20,     // 20% marge opérationnelle
    churnRate: 0.12,             // Maximum 12% désabonnement annuel
    acquisitionCost: 800         // Maximum 800 FCFA par utilisateur
  },

  milestones: {
    phase1: { users: 25000, months: 12, status: 'Pilote multi-pays' },
    phase2: { users: 500000, months: 36, status: 'Expansion CEDEAO' },
    breakeven: { users: 50000, months: 24, status: 'Rentabilité atteinte' }
  }
};
```

### Facteurs de Risque et Mitigation

```typescript
export const RISK_MITIGATION = {
  climaticRisks: {
    risk: 'Catastrophes climatiques majeures (>2x normale)',
    mitigation: 'Fonds de réserve gouvernemental 10% + réassurance',
    impact: 'Couvert par réserves jusqu\'à 200% sinistralité normale'
  },

  technicalRisks: {
    risk: 'Augmentation coûts OpenEPI (>20%)',
    mitigation: 'Contrats long terme + APIs fallback',
    impact: 'Négociation volume + diversification sources'
  },

  marketRisks: {
    risk: 'Baisse adoption mobile money (<5%/an)',
    mitigation: 'Partenariats MTN/Orange + éducation',
    impact: 'Croissance mobile money 15%/an en Afrique Ouest'
  },

  competitiveRisks: {
    risk: 'Concurrence agressive (baisse primes >30%)',
    mitigation: 'Différenciation OpenEPI + service 980',
    impact: 'Avantage technologique et accessibilité'
  }
};
```
```

## Partenariats et Revenus Additionnels

### Sources de Revenus Complémentaires

```typescript
// Modèle de revenus diversifiés
export const ADDITIONAL_REVENUE_STREAMS = {
  creditScoring: {
    description: 'Scoring de crédit basé sur données OpenEPI',
    revenueModel: 'Commission sur prêts accordés',
    potentialRevenue: '2-5% du montant des prêts',
    partners: ['Banques agricoles', 'IMF', 'Coopératives'],
    implementation: 'Intégré dans onglet Analyses'
  },
  
  dataInsights: {
    description: 'Vente de données agricoles agrégées et anonymisées',
    revenueModel: 'Licence annuelle ou par requête',
    potentialRevenue: '10-50 FCFA par agriculteur par mois',
    partners: ['Gouvernements', 'ONG', 'Instituts de recherche'],
    dataTypes: ['Tendances climatiques', 'Rendements agricoles', 'Adoption technologique']
  },
  
  consultingServices: {
    description: 'Conseil en adaptation climatique',
    revenueModel: 'Forfait par projet ou abonnement',
    potentialRevenue: '50,000-500,000 FCFA par mission',
    partners: ['Projets de développement', 'Gouvernements locaux'],
    services: ['Cartographie des risques', 'Plans d\'adaptation', 'Formation']
  },
  
  equipmentFinancing: {
    description: 'Financement d\'équipements agricoles résistants au climat',
    revenueModel: 'Commission sur ventes financées',
    potentialRevenue: '3-8% du montant financé',
    partners: ['Fabricants d\'équipements', 'Distributeurs'],
    products: ['Systèmes d\'irrigation', 'Semences résistantes', 'Outils agricoles']
  }
};

// Calcul du potentiel de revenus additionnels
export class AdditionalRevenueCalculator {
  calculatePotential(userBase: number) {
    const creditScoringRevenue = userBase * 0.3 * 50000 * 0.03; // 30% demandent crédit, 50k FCFA moyen, 3% commission
    const dataInsightsRevenue = userBase * 25 * 12; // 25 FCFA/mois par utilisateur
    const consultingRevenue = 200000 * 12; // 200k FCFA/mois en moyenne
    const equipmentRevenue = userBase * 0.1 * 100000 * 0.05; // 10% achètent équipement, 100k FCFA moyen, 5% commission

    return {
      creditScoring: creditScoringRevenue,
      dataInsights: dataInsightsRevenue,
      consulting: consultingRevenue,
      equipment: equipmentRevenue,
      total: creditScoringRevenue + dataInsightsRevenue + consultingRevenue + equipmentRevenue,
      percentageOfPrimaryRevenue: ((creditScoringRevenue + dataInsightsRevenue + consultingRevenue + equipmentRevenue) / (userBase * 600 * 12)) * 100
    };
  }
}
```

## Stratégie de Financement

### Phases de Financement

```typescript
// Plan de financement par phases
export const FUNDING_STRATEGY = {
  phase1_seed: {
    amount: 50000000,           // 50M FCFA
    duration: '12 mois',
    objectives: [
      'Développement MVP complet',
      'Intégration OpenEPI avancée',
      'Pilote 5,000 utilisateurs',
      'Validation modèle économique'
    ],
    sources: ['Concours innovation', 'Business angels', 'Subventions développement']
  },
  
  phase2_seriesA: {
    amount: 200000000,          // 200M FCFA
    duration: '24 mois',
    objectives: [
      'Expansion 50,000 utilisateurs',
      'Déploiement 3 pays',
      'Partenariats Mobile Money',
      'Équipe 25 personnes'
    ],
    sources: ['Fonds d\'impact', 'Banque Mondiale', 'Investisseurs institutionnels']
  },
  
  phase3_seriesB: {
    amount: 500000000,          // 500M FCFA
    duration: '36 mois',
    objectives: [
      'Échelle 500,000 utilisateurs',
      'Couverture CEDEAO complète',
      'Services crédit intégrés',
      'Rentabilité opérationnelle'
    ],
    sources: ['Fonds de développement', 'IFC', 'Investisseurs privés']
  }
};
```

Ce modèle économique garantit la viabilité financière de ClimInvest tout en maintenant l'accessibilité pour les agriculteurs africains, grâce à l'optimisation des coûts par les données OpenEPI et la diversification des canaux d'accès.
