# DIAGRAMMES PLANTUML - PROJET GARDENSNOLIFE

Ce dossier contient tous les diagrammes PlantUML pour le système de gestion des missions de GardensNoLife.

## 📋 Liste des Diagrammes

### 1. **01_diagramme_cas_utilisation.puml**
**Diagramme de Cas d'Utilisation**
- **Objectif :** Vue fonctionnelle complète du système
- **Acteurs :** Collaborateur, Manager, Service RH, Système
- **Cas d'usage :** Processus complet de demande de mission V4
- **Spécificités :** Intègre les contraintes CO2, quotas jours, et gestion des devises

### 2. **02_diagramme_sequence_saisie_mission.puml** 
**Diagramme de Séquence - Saisie Demande Mission**
- **Objectif :** Interactions détaillées lors de la saisie d'une demande
- **Phases :** Saisie → Vérifications automatiques → Calcul CO2
- **Intégrations :** API ADEME, vérification quotas, validation automatique
- **Cas d'échec :** Quota jours/CO2 dépassé, refus automatique

### 3. **03_diagramme_sequence_validation_manager_rh.puml**
**Diagramme de Séquence - Validation Manager et RH**
- **Objectif :** Processus de validation humaine parallèle
- **Acteurs :** Manager, Service RH, Collaborateur
- **Phases :** Validation Manager → Validation RH → Synchronisation → Réponse Collaborateur
- **Gestion :** Désaccords, modification avance frais, conversion devises

### 4. **04_modele_conceptuel_donnees.puml**
**Modèle Conceptuel de Données**
- **Entités principales :** Collaborateur, DemandeMission, HistoriqueMission
- **Gestion quotas :** QuotaCollaborateur, ParametresSysteme
- **Traçabilité :** HistoriqueMission, NotificationEmail
- **Énumérations :** StatutMission, MoyenTransport

### 5. **05_architecture_logicielle_composants.puml**
**Architecture Logicielle - Composants**
- **Style :** Architecture C4 en couches
- **Couches :** Présentation → Moteur → Logique Métier → Intégration → Données
- **Services :** Mission, Quota, CO2, Notification, Validation, Devises
- **Intégrations :** ADEME, Taux de change, Email, Géolocalisation

### 6. **06_diagramme_deploiement.puml**
**Diagramme de Déploiement**
- **Architecture :** Multi-tiers avec haute disponibilité
- **Zones :** DMZ, Applicative, Services, Données, Monitoring
- **Technologies :** Bonita Community, PostgreSQL, Redis, Nginx
- **Monitoring :** Prometheus, Grafana, ELK Stack

## 🚀 Utilisation des Diagrammes

### Génération des Images
```bash
# Installer PlantUML
npm install -g plantuml

# Générer toutes les images
plantuml *.puml

# Générer un diagramme spécifique
plantuml 01_diagramme_cas_utilisation.puml
```

### Visualisation en Ligne
- **PlantUML Online :** https://www.plantuml.com/plantuml/uml/
- **VS Code Extension :** PlantUML Preview
- **IntelliJ Plugin :** PlantUML Integration

## 📝 Contexte Projet

### Évolution du Système (V1 → V4)
- **V1 :** Processus de base (Collaborateur → Manager → RH)
- **V2 :** Contraintes administratives (25 jours/an)
- **V3 :** Gestion défraiements et conversion devises
- **V4 :** Empreinte carbone et quotas CO2 (1 tonne/an)

### Règles Métier Clés
- **Transport < 200km :** Véhicule électrique
- **Transport > 700km :** Avion (API ADEME)
- **Transport 200-700km :** Train
- **Manager prévaut** en cas de désaccord avec RH

### APIs Externes
- **ADEME :** https://impactco2.fr/doc/api (Clé: 51198b04...)
- **Taux de change :** Service temps réel EUR → devises locales
- **Email :** SMTP Gmail avec TLS/SSL

## 🔧 Configuration PlantUML

### Thèmes Recommandés
```plantuml
!theme aws-orange
!include <C4/C4_Context>
!include <C4/C4_Component>
```

### Extensions Utiles
- **Icônes AWS :** `!include <aws/common>`
- **Icônes Azure :** `!include <azure/AzureRaw/...>`
- **C4 Model :** `!include <C4/C4_Context>`

## 📊 Métriques et KPIs

### Performance Attendue
- **Temps traitement demande :** < 5 secondes
- **Disponibilité :** 99.5%
- **Capacité :** 100 demandes simultanées

### Quotas Système
- **Jours mission :** 25 jours/collaborateur/an
- **Empreinte CO2 :** 1 tonne équivalent/collaborateur/an
- **Avance frais :** Modifiable par RH uniquement

---

**Auteur :** Équipe Architecture GardensNoLife  
**Version :** 4.0  
**Dernière MAJ :** Décembre 2024 