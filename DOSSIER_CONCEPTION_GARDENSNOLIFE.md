# DOSSIER DE CONCEPTION - SYSTÈME DE GESTION DES MISSIONS
## GardensNoLife

**Version :** 4.0  
**Date :** Décembre 2024  
**Entreprise :** GardensNoLife - Conception et réalisation de jardins à la carte  
**Localisation :** Toulouse, Occitanie (clientèle mondiale)

---

## TABLE DES MATIÈRES

1. [Vue Fonctionnelle du Système](#1-vue-fonctionnelle-du-système)
2. [Justification des Choix de Modélisation des Processus Métiers](#2-justification-des-choix-de-modélisation-des-processus-métiers)
3. [Justification des Choix de Modélisation des Données Métiers](#3-justification-des-choix-de-modélisation-des-données-métiers)
4. [Architecture Logicielle](#4-architecture-logicielle)
5. [Interactions Système](#5-interactions-système)
6. [Éléments de Contexte et Gestion des Erreurs](#6-éléments-de-contexte-et-gestion-des-erreurs)

---

## 1. VUE FONCTIONNELLE DU SYSTÈME

### 1.1 Contexte Métier

GardensNoLife est une entreprise toulousaine spécialisée dans la conception et réalisation de jardins à la carte avec une clientèle mondiale. Le système actuel de gestion des missions (tableur, emails, papier) nécessite une automatisation complète.

**Problématique :** Le processus actuel de gestion des demandes de mission est entièrement manuel (tableur, emails, papiers), ce qui génère des erreurs et une perte de temps considérable.

**Objectif :** Automatiser et rationaliser le processus de demande et validation des missions avec intégration au système RH.

### 1.2 Diagramme de Cas d'Utilisation Global

```plantuml
@startuml CasUtilisationGlobal
title Système de Gestion des Missions - GardensNoLife

actor "Collaborateur" as COLLAB
actor "Manager" as MANAGER 
actor "Service RH" as RH
actor "Système" as SYSTEM

package "Gestion des Missions" {
  usecase "Saisir demande mission" as UC1
  usecase "Suivre mes demandes" as UC2
  usecase "Annuler demande" as UC3
  
  usecase "Étudier demande" as UC4
  usecase "Valider/Refuser mission" as UC5
  
  usecase "Analyser opportunité" as UC7
  usecase "Gérer quotas jours mission" as UC8
  usecase "Modifier avance frais" as UC9
  
  usecase "Vérifier solde jours" as UC12
  usecase "Calculer empreinte CO2" as UC13
  usecase "Notifier collaborateur" as UC15
}

COLLAB --> UC1
COLLAB --> UC2
COLLAB --> UC3

MANAGER --> UC4
MANAGER --> UC5

RH --> UC7
RH --> UC8
RH --> UC9

SYSTEM --> UC12
SYSTEM --> UC13
SYSTEM --> UC15

@enduml
```

### 1.3 Acteurs du Système

| Acteur | Rôle | Responsabilités |
|--------|------|----------------|
| **Collaborateur** | Demandeur de mission | Saisie des demandes, suivi, validation finale des conditions modifiées |
| **Manager** | Validateur hiérarchique | Validation/refus des demandes, décision finale en cas de désaccord avec RH |
| **Service RH** | Gestionnaire administratif | Analyse d'opportunité, gestion des quotas, validation des demandes exceptionnelles |
| **Système** | Automatisation | Vérifications automatiques, calculs, notifications |

---

## 2. JUSTIFICATION DES CHOIX DE MODÉLISATION DES PROCESSUS MÉTIERS

### 2.1 Évolution des Processus par Version

#### Version 1 - Processus de Base
Modélisation simple avec gateway de synchronisation pour gérer les désaccords Manager/RH.

#### Version 4 - Processus Complet avec Empreinte Carbone

```plantuml
@startuml ProcessusV4Complet
title Processus V4 - Complet avec Empreinte Carbone

|Collaborateur|
start
:Saisir demande mission + Avance frais;

|Système|
:Vérifier solde jours mission;
if (Jours suffisants?) then (Non)
  |Service RH|
  :Évaluer demande exceptionnelle;
  if (Demande acceptable?) then (Non)
    :Refus;
    stop
  endif
endif

:Calculer empreinte CO2;
if (Quota CO2 dépassé?) then (Oui)
  :Refus automatique;
  stop
endif

|Manager|
:Étudier demande;

|Service RH|
:Analyser opportunité;
:Modifier avance frais;

|Système|
:Calculer taux de change;
:Envoyer proposition;

|Collaborateur|
if (Acceptation?) then (Non)
  :Refus collaborateur;
  stop
else (Oui)
  |Système|
  :Déduire jours de solde;
  :Mission approuvée;
endif

stop

@enduml
```

### 2.2 Patterns BPMN Utilisés

1. **Exclusive Gateway (XOR)** : Décisions mutuellement exclusives
2. **Parallel Gateway (AND)** : Exécution parallèle des validations
3. **Inclusive Gateway** : Gestion des cas exceptionnels
4. **Service Tasks** : Intégrations système automatiques
5. **User Tasks** : Interventions humaines avec formulaires

---

## 3. JUSTIFICATION DES CHOIX DE MODÉLISATION DES DONNÉES MÉTIERS

### 3.1 Modèle de Données Conceptuel

```plantuml
@startuml ModeleConceptuel
title Modèle Conceptuel des Données

entity "DemandeMission" as MISSION {
  * id : UUID
  * collaborateur : String
  * destination : String
  * dateDebut : OffsetDateTime
  * dateFin : OffsetDateTime
  * motif : String
  * avanceFrais : Decimal
  * avanceFraisModifiee : Decimal
  * monnaieLocale : String
  * tauxChange : Decimal
  * statut : String
  * decisionManager : Boolean
  * decisionRH : Boolean
  * distanceKm : Double
  * empreinteCO2 : Double
  * moyenTransport : String
}

enum StatutMission {
  BROUILLON
  EN_COURS
  APPROUVEE
  REFUSEE
  ANNULEE
}

enum MoyenTransport {
  VEHICULE_ELECTRIQUE
  TRAIN
  AVION
}

MISSION --> StatutMission
MISSION --> MoyenTransport

@enduml
```

### 3.2 Justifications des Choix de Données

#### 3.2.1 Entité DemandeMission
**Choix :** Entité centrale avec tous les attributs de gestion
**Justifications :**
- **avanceFrais + avanceFraisModifiee** : Traçabilité des modifications RH
- **monnaieLocale + tauxChange** : Support international complet
- **empreinteCO2 + moyenTransport** : Calculs environnementaux
- **statut énuméré** : États contrôlés (BROUILLON, EN_COURS, APPROUVEE, REFUSEE, ANNULEE)

#### 3.2.2 Gestion des Quotas
**Choix :** Attributs directs sur Collaborateur
**Justifications :**
- Performance : accès direct sans jointures
- Cohérence : mise à jour atomique
- Simplicité : pas de gestion de périodes complexes

#### 3.2.3 Historique et Traçabilité
**Choix :** Table d'audit séparée
**Justifications :**
- Conformité réglementaire
- Debugging et support utilisateur
- Analytics et reporting

### 3.3 Modèle de Données Physique

```plantuml
@startuml ModelePhysiqueDonnees
title Modèle Physique - Base de Données

class DemandeMission {
  + persistenceId : Long
  + collaborateur : String
  + destination : String
  + dateDebut : OffsetDateTime
  + dateFin : OffsetDateTime
  + motif : String
  + avanceFrais : BigDecimal
  + avanceFraisModifiee : BigDecimal
  + monnaieLocale : String
  + tauxChange : BigDecimal
  + statut : String
  + decisionManager : Boolean
  + decisionRH : Boolean
  + distanceKm : Double
  + empreinteCO2 : Double
  + moyenTransport : String
  + dateCreation : OffsetDateTime
  + dateValidation : OffsetDateTime
  --
  + calculerDuree() : Integer
  + calculerEmpreinteCO2() : Double
  + validerQuotaCO2() : Boolean
}

enum StatutMission {
  BROUILLON
  EN_COURS
  APPROUVEE
  REFUSEE
  ANNULEE
}

enum MoyenTransport {
  VEHICULE_ELECTRIQUE
  TRAIN
  AVION
}

DemandeMission --> StatutMission
DemandeMission --> MoyenTransport

@enduml
```

---

## 4. ARCHITECTURE LOGICIELLE

### 4.1 Diagramme de Composants

```plantuml
@startuml DiagrammeComposants
title Architecture Logicielle - GardensNoLife

package "Interface Utilisateur" {
  component [Formulaires Web Bonita] as FORMS
  component [Portail Collaborateur] as PORTAL
}

package "Moteur de Processus" {
  component [Bonita Engine] as ENGINE
  component [Gestionnaire de Processus] as PROCESS_MGR
}

package "Logique Métier" {
  component [Service Missions] as MISSION_SVC
  component [Service Quotas] as QUOTA_SVC
  component [Service CO2] as CO2_SVC
  component [Service Notifications] as NOTIF_SVC
}

package "Intégrations Externes" {
  component [API ADEME CO2] as ADEME_API
  component [API Taux de Change] as EXCHANGE_API
  component [Service Email] as EMAIL_SVC
}

package "Couche de Données" {
  component [BDM Bonita] as BDM
  database [Base de Données] as DB
}

FORMS --> ENGINE
ENGINE --> PROCESS_MGR
PROCESS_MGR --> MISSION_SVC
MISSION_SVC --> CO2_SVC
CO2_SVC --> ADEME_API
MISSION_SVC --> BDM
BDM --> DB

@enduml
```

### 4.2 Diagramme de Déploiement

```plantuml
@startuml DiagrammeDeploiement
title Architecture de Déploiement

node "Zone Applicative" {
  node "Serveur Bonita" as APP {
    component [Bonita Community] as BONITA
    component [JVM Tomcat] as JVM
  }
}

node "Zone Données" {
  node "Serveur BDD" as DB_SERVER {
    database [PostgreSQL] as POSTGRES
  }
}

cloud "Services Externes" {
  component [API ADEME] as EXT_ADEME
  component [API Exchange Rates] as EXT_EXCHANGE
  component [Service SMTP] as EXT_SMTP
}

BONITA --> POSTGRES
BONITA --> EXT_ADEME
BONITA --> EXT_EXCHANGE
BONITA --> EXT_SMTP

@enduml
```

### 4.3 Justifications Architecturales

#### 4.3.1 Choix de Bonita Community
**Justifications :**
- Conformité au cahier des charges
- Intégration native des processus BPMN
- Gestion intégrée des formulaires et notifications
- Coût maîtrisé pour l'entreprise

#### 4.3.2 Architecture Multi-Tiers
**Justifications :**
- Séparation des responsabilités
- Scalabilité horizontale possible
- Maintenance facilitée
- Sécurité par cloisonnement

#### 4.3.3 Intégrations API Externes
**Justifications :**
- **API ADEME** : Données officielles et fiables pour CO2
- **API Taux de Change** : Données temps réel pour conversions
- **Service SMTP** : Notifications robustes

---

## 5. INTERACTIONS SYSTÈME

### 5.1 Diagramme de Séquence - Saisie Demande Mission

```plantuml
@startuml SequenceSaisieDemande
title Séquence Saisie et Validation Demande Mission

actor "Collaborateur" as COLLAB
participant "Formulaire Web" as FORM
participant "Bonita Engine" as ENGINE
participant "Service Mission" as MISSION_SVC
participant "Service Quota" as QUOTA_SVC
participant "API ADEME" as ADEME

COLLAB -> FORM : Saisir demande mission
FORM -> ENGINE : Démarrer processus
ENGINE -> MISSION_SVC : Créer demande
ENGINE -> QUOTA_SVC : Vérifier solde jours

alt Solde insuffisant
    QUOTA_SVC --> ENGINE : Solde insuffisant
    ENGINE -> MISSION_SVC : Demande exceptionnelle
else Solde suffisant
    QUOTA_SVC --> ENGINE : Solde OK
end

ENGINE -> MISSION_SVC : Calculer empreinte CO2
MISSION_SVC -> ADEME : GET calcul CO2
ADEME --> MISSION_SVC : Empreinte CO2

alt Quota CO2 dépassé
    MISSION_SVC --> ENGINE : Quota dépassé
    ENGINE --> COLLAB : Mission refusée
else Quota CO2 OK
    ENGINE --> COLLAB : Demande transmise
end

@enduml
```

### 5.2 Diagramme de Séquence - Validation Manager et RH

```plantuml
@startuml SequenceValidationManagerRH
title Séquence Validation Manager et Service RH

actor "Manager" as MANAGER
actor "Service RH" as RH
participant "Interface Manager" as MGR_UI
participant "Interface RH" as RH_UI
participant "Bonita Engine" as ENGINE
participant "Service Mission" as MISSION_SVC
participant "Service Notification" as NOTIF_SVC
participant "Base Données" as DB

MANAGER -> MGR_UI : Accéder tâche validation
activate MGR_UI

MGR_UI -> ENGINE : Récupérer demande mission
activate ENGINE

ENGINE -> MISSION_SVC : GET détails mission
activate MISSION_SVC

MISSION_SVC -> DB : SELECT mission
DB --> MISSION_SVC : Données mission
MISSION_SVC --> ENGINE : Détails mission
ENGINE --> MGR_UI : Affichage demande

MANAGER -> MGR_UI : Valider/Refuser + Commentaire
MGR_UI -> ENGINE : Soumettre décision Manager

ENGINE -> MISSION_SVC : Enregistrer décision Manager
MISSION_SVC -> DB : UPDATE decisionManager
DB --> MISSION_SVC : Confirmation

== Validation parallèle RH ==

par Processus RH en parallèle
    RH -> RH_UI : Accéder tâche analyse
    activate RH_UI
    
    RH_UI -> ENGINE : Récupérer demande mission
    ENGINE -> MISSION_SVC : GET détails mission
    MISSION_SVC --> ENGINE : Détails mission
    ENGINE --> RH_UI : Affichage demande
    
    RH -> RH_UI : Analyser opportunité + Modifier avance frais
    RH_UI -> ENGINE : Soumettre décision RH
    
    ENGINE -> MISSION_SVC : Enregistrer décision RH
    MISSION_SVC -> DB : UPDATE decisionRH + avanceFraisModifiee
    DB --> MISSION_SVC : Confirmation
    
    deactivate RH_UI
end

== Synchronisation des décisions ==

ENGINE -> MISSION_SVC : Comparer décisions
alt Décisions identiques
    MISSION_SVC -> DB : UPDATE statut final
    ENGINE -> NOTIF_SVC : Notifier collaborateur
else Décisions différentes
    ENGINE -> MGR_UI : Rediriger vers Manager (dernier mot)
    MGR_UI --> MANAGER : Nouvelle tâche validation finale
end

deactivate MISSION_SVC
deactivate ENGINE
deactivate MGR_UI

@enduml
```

### 5.3 Diagramme de Séquence - Gestion Avance Frais et Conversion

```plantuml
@startuml SequenceAvanceFraisConversion
title Séquence Gestion Avance Frais et Conversion Monétaire

actor "Service RH" as RH
actor "Collaborateur" as COLLAB
participant "Interface RH" as RH_UI
participant "Interface Collab" as COLLAB_UI
participant "Bonita Engine" as ENGINE
participant "Service Mission" as MISSION_SVC
participant "API Exchange" as EXCHANGE_API
participant "Service Notification" as NOTIF_SVC
participant "Base Données" as DB

RH -> RH_UI : Modifier avance frais
activate RH_UI

RH_UI -> ENGINE : Soumettre nouvelle avance
activate ENGINE

ENGINE -> MISSION_SVC : Mettre à jour avance frais
activate MISSION_SVC

MISSION_SVC -> DB : UPDATE avanceFraisModifiee
DB --> MISSION_SVC : Confirmation

== Calcul taux de change ==

ENGINE -> MISSION_SVC : Calculer conversion monétaire
MISSION_SVC -> EXCHANGE_API : GET taux de change (EUR -> devise locale)
activate EXCHANGE_API
EXCHANGE_API --> MISSION_SVC : Taux actuel
deactivate EXCHANGE_API

MISSION_SVC -> DB : UPDATE tauxChange + monnaieLocale
DB --> MISSION_SVC : Confirmation

== Notification collaborateur ==

ENGINE -> NOTIF_SVC : Envoyer proposition finale
activate NOTIF_SVC

NOTIF_SVC -> COLLAB_UI : Email + notification in-app
activate COLLAB_UI

NOTIF_SVC -> DB : LOG notification envoyée
DB --> NOTIF_SVC : Confirmation

COLLAB_UI --> COLLAB : Affichage proposition modifiée

== Réponse collaborateur ==

COLLAB -> COLLAB_UI : Accepter/Refuser proposition
COLLAB_UI -> ENGINE : Réponse collaborateur

alt Acceptation
    ENGINE -> MISSION_SVC : Finaliser mission
    MISSION_SVC -> DB : UPDATE statut APPROUVEE
    
    ENGINE -> MISSION_SVC : Déduire jours mission
    MISSION_SVC -> DB : UPDATE soldeJoursMission collaborateur
    
    ENGINE -> NOTIF_SVC : Confirmer mission approuvée
else Refus
    ENGINE -> MISSION_SVC : Annuler mission
    MISSION_SVC -> DB : UPDATE statut ANNULEE
    
    ENGINE -> NOTIF_SVC : Confirmer mission annulée
end

deactivate COLLAB_UI
deactivate NOTIF_SVC
deactivate MISSION_SVC
deactivate ENGINE
deactivate RH_UI

@enduml
```

---

## 6. ÉLÉMENTS DE CONTEXTE ET GESTION DES ERREURS

### 6.1 Contraintes et Règles Métier

#### 6.1.1 Contraintes Administratives
- **Quota jours mission :** 25 jours maximum par collaborateur par an
- **Quota CO2 :** 1 tonne équivalent CO2 par collaborateur par an
- **Demandes exceptionnelles :** Validation RH obligatoire si quotas dépassés

#### 6.1.2 Règles de Transport et Calcul CO2
```plantuml
@startuml ReglesTransportCO2
title Règles Métier - Calcul Transport et CO2

start

:Calculer distance Toulouse ↔ Destination;

if (Distance < 200 km?) then (Oui)
  :Véhicule électrique;
  :Empreinte CO2 = 0.05 kg/km;
elseif (Distance > 700 km?) then (Oui)
  :Avion;
  :Appel API ADEME;
  :Empreinte CO2 = valeur API;
else (200-700 km)
  :Train;
  :Empreinte CO2 = 0.03 kg/km;
endif

:Calculer empreinte totale;
:Vérifier quota annuel CO2;

if (Quota dépassé?) then (Oui)
  :Refus automatique;
  stop
else (Non)
  :Continuer processus;
endif

stop

@enduml
```

#### 6.1.3 Règles de Validation
- **Manager :** Décision finale en cas de désaccord avec RH
- **RH :** Peut modifier l'avance de frais librement
- **Collaborateur :** Peut refuser la mission après modification RH

### 6.2 Gestion des Erreurs et Cas d'Exception

#### 6.2.1 Matrice de Gestion des Erreurs

| Type d'Erreur | Cause | Action | Responsable |
|---------------|-------|---------|-------------|
| **Quota jours dépassé** | Demande > solde disponible | Redirection vers processus exceptionnel | Système → RH |
| **Quota CO2 dépassé** | Empreinte calculée > quota restant | Refus automatique + notification | Système |
| **API ADEME indisponible** | Timeout/erreur réseau | Utilisation valeurs par défaut + log | Système |
| **API Taux change indisponible** | Timeout/erreur réseau | Utilisation taux BCE + alerte admin | Système |
| **Email notification échec** | SMTP indisponible | Retry 3x + notification alternative | Système |
| **Désaccord Manager/RH** | Décisions opposées | Processus de réétude Manager | Métier |

#### 6.2.2 Diagramme de Gestion des Erreurs API

```plantuml
@startuml GestionErreursAPI
title Gestion des Erreurs - APIs Externes

start

:Appel API externe;

if (Réponse OK?) then (Oui)
  :Traiter réponse;
  :Continuer processus;
else (Non)
  if (Type erreur?) then (Timeout)
    :Retry après 5s;
    if (Retry 3x échec?) then (Oui)
      :Utiliser valeur par défaut;
      :Logger erreur;
      :Alerter administrateur;
    endif
  elseif (Erreur 4xx) then
    :Logger erreur demande;
    :Utiliser valeur par défaut;
  elseif (Erreur 5xx) then
    :Logger erreur serveur;
    :Utiliser valeur par défaut;
    :Alerter administrateur;
  endif
endif

:Continuer processus avec données disponibles;

stop

@enduml
```

### 6.3 Éléments de Sécurité

#### 6.3.1 Authentification et Autorisation
- **SSO :** Integration avec Active Directory GardensNoLife
- **RBAC :** Rôles définis (Collaborateur, Manager, RH, Admin)
- **API Security :** Tokens JWT pour APIs externes

#### 6.3.2 Protection des Données
- **RGPD :** Anonymisation possible des données après archivage
- **Chiffrement :** Communications HTTPS/TLS 1.3
- **Audit :** Traçabilité complète des actions utilisateurs

### 6.4 Performance et Scalabilité

#### 6.4.1 Indicateurs de Performance
- **Temps de traitement :** < 5 secondes par demande
- **Disponibilité :** 99.5% (hors maintenance programmée)
- **Capacité :** 100 demandes simultanées

#### 6.4.2 Stratégies d'Optimisation
- **Cache Redis :** Taux de change, quotas utilisateurs
- **Pool de connexions :** Base de données optimisée
- **Pagination :** Listes de demandes par lots de 20

### 6.5 Monitoring et Supervision

```plantuml
@startuml ArchitectureMonitoring
title Architecture Monitoring et Supervision

package "Applications" {
  component [Bonita Engine] as BONITA
  component [Service Missions] as MISSION
  component [APIs Externes] as APIS
}

package "Collecte Métriques" {
  component [Prometheus Agent] as PROM_AGENT
  component [Log Collector] as LOG_COLLECTOR
  component [APM Agent] as APM
}

package "Stockage et Analyse" {
  database [Prometheus TSDB] as PROM_DB
  database [Elasticsearch] as ELASTIC
  component [Grafana] as GRAFANA
  component [Kibana] as KIBANA
}

package "Alerting" {
  component [AlertManager] as ALERT
  component [PagerDuty] as PAGER
  component [Email Alerts] as EMAIL_ALERT
}

' Collecte
BONITA --> PROM_AGENT : métriques
MISSION --> PROM_AGENT : métriques
APIS --> LOG_COLLECTOR : logs
BONITA --> APM : traces

' Stockage
PROM_AGENT --> PROM_DB
LOG_COLLECTOR --> ELASTIC
APM --> ELASTIC

' Visualisation
PROM_DB --> GRAFANA
ELASTIC --> KIBANA

' Alerting
PROM_DB --> ALERT
ALERT --> PAGER : urgences
ALERT --> EMAIL_ALERT : warnings

@enduml
```

### 6.6 Plan de Continuité et Sauvegarde

#### 6.6.1 Stratégie de Sauvegarde
- **Base de données :** Sauvegarde complète quotidienne + log transaction
- **Fichiers processus :** Versioning Git + backup sur NAS
- **Configuration :** Infrastructure as Code (Ansible)

#### 6.6.2 Plan de Reprise d'Activité
- **RTO :** 4 heures maximum
- **RPO :** 1 heure maximum (perte de données acceptable)
- **Site de secours :** Datacenter partenaire avec réplication async

---

## CONCLUSION

Ce dossier de conception présente une solution complète et évolutive pour la gestion des missions chez GardensNoLife. L'architecture proposée répond aux exigences fonctionnelles tout en intégrant les contraintes modernes de sécurité, performance et monitoring.

**Points forts de la solution :**
- ✅ Automatisation complète du processus manuel existant
- ✅ Intégration des contraintes administratives et environnementales
- ✅ Architecture scalable et maintenable
- ✅ Gestion robuste des erreurs et cas d'exception
- ✅ Conformité RGPD et sécurité moderne

**Prochaines étapes :**
1. Validation de l'architecture avec les équipes techniques
2. Développement des maquettes Bonita
3. Tests d'intégration avec les APIs externes
4. Formation des utilisateurs et mise en production progressive 