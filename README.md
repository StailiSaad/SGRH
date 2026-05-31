<p align="center">
  <img src="assets/logo_council.png" alt="Logo du Conseil Provincial de Safi" width="120"/>
</p>

<h1 align="center">SIRH — Système d'Information des Ressources Humaines</h1>

<p align="center">
  <strong>Conseil Provincial de Safi</strong><br>
  <em>Application mobile de gestion des ressources humaines</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-1.0.0-blue" alt="Version"/>
  <img src="https://img.shields.io/badge/platform-Android-brightgreen" alt="Platform"/>
  <img src="https://img.shields.io/badge/Kotlin-2.2+-7F52FF?logo=kotlin" alt="Kotlin"/>
  <img src="https://img.shields.io/badge/Spring_Boot-3.4-6DB33F?logo=springboot" alt="Spring Boot"/>
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql" alt="PostgreSQL"/>
  <img src="https://img.shields.io/badge/Hugging_Face-FFD21E?logo=huggingface" alt="Hugging Face"/>
</p>

---

## 📋 Table des matières

- [Aperçu](#aperçu)
- [Fonctionnalités](#fonctionnalités)
- [Architecture technique](#architecture-technique)
- [API Endpoints](#api-endpoints)
- [Captures d'écran](#captures-décran)
- [Installation](#installation)
- [Schémas de base de données](#schémas-de-base-de-données)
- [Technologies utilisées](#technologies-utilisées)
- [Livrables](#livrables)
- [Sécurité et confidentialité](#sécurité-et-confidentialité)
- [Auteur](#auteur)

---

## 🔍 Aperçu

**SIRH** (Système d'Information des Ressources Humaines) est une **application mobile Android** développée dans le cadre d'un **stage d'étude** au sein du **Conseil Provincial de Safi**. Cette application est conçue exclusivement pour les **fonctionnaires et agents** du conseil afin de moderniser et digitaliser la gestion quotidienne des ressources humaines.

Le système se compose de :

- 📱 **Application mobile Android** (Kotlin / Jetpack Compose) — interface utilisateur pour employés et administrateurs RH
- 🖥️ **API REST backend** (Kotlin / Spring Boot 3) — logique métier et exposition des données
- 🗄️ **Base de données PostgreSQL** — stockage persistant des données RH
- ☁️ **Hébergement** sur **Hugging Face Spaces** via Docker

L'application couvre l'ensemble du cycle de gestion RH :

- Gestion complète des dossiers du personnel
- Demandes de congés avec calcul automatique du solde
- Sollicitations de documents administratifs
- Génération de décisions administratives (PDF)
- Notifications push en temps réel
- Tableau de bord administrateur avec indicateurs clés

> ⚠️ **Confidentialité :** Cette application et l'ensemble de son code source sont destinés **exclusivement** à un usage interne au Conseil Provincial de Safi. Toute diffusion, reproduction ou utilisation non autorisée est strictement interdite.

---

## ✨ Fonctionnalités

### 🔐 Authentification & Sécurité
- Connexion sécurisée par **JWT (JSON Web Token)** avec signature HS256
- Tokens valides 24h avec renouvellement automatique
- Architecture **Spring Security** avec filtres JWT personnalisés
- Mots de passe chiffrés via **BCrypt**
- Contrôle d'accès basé sur les rôles : `ADMIN_RH` et `EMPLOYE`
- Routes publiques : login, liste des services

### 👑 Portail Administrateur (ADMIN_RH)

| Fonctionnalité | Description |
|---|---|
| **Tableau de bord** | Vue consolidée du personnel, indicateurs clés, accès rapide |
| **Gestion des dossiers** | Ajout, modification, consultation et suppression des fonctionnaires |
| **Recherche avancée** | Filtrage par nom, prénom ou CIN |
| **Validation des congés** | Acceptation ou refus des demandes avec mise à jour automatique du solde |
| **Traitement des demandes** | Gestion des sollicitations de documents (attestations, bulletins) |
| **Décisions administratives** | Génération de PDF (قرارات) et accès aux modèles .docx (70+ templates) |
| **Notifications** | Centre de notifications avec badge dynamique |

### 👤 Portail Employé (Espace Personnel)

| Fonctionnalité | Description |
|---|---|
| **Profil personnel** | Identité, situation administrative, affectation, historique de carrière |
| **Demande de congé** | Formulaire dématérialisé avec calcul en temps réel du solde restant |
| **Sollicitation de documents** | Demande d'attestations de travail, fiches de paie, etc. |
| **Suivi des requêtes** | Consultation du statut des demandes (en attente, validée, refusée) |
| **Centre de notifications** | Alertes interactives avec badge de messages non lus |

### 📊 Gestion des données
- **Import Excel** : Chargement des données employés via fichier `.xlsx` (N° PPR, Nom, CIN, Grade, Service)
- **Génération de décisions** : Production de documents PDF via **Apache PDFBox**
- **Modèles de documents** : 70+ modèles de décisions administratives en arabe (format .docx) intégrés dans l'application

---

## 🏗️ Architecture Technique

### Architecture globale

```
┌──────────────────────────────────────────────────────────────────────────┐
│                      APPLICATION ANDROID (Kotlin)                         │
├──────────────────────────────────────────────────────────────────────────┤
│                          Couche Présentation (UI)                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │  Login   │ │Dashboard │ │  Profile  │ │  Conges  │ │  AdminPanel  │  │
│  │  Screen  │ │  Screen  │ │  Screen   │ │  Screen  │ │  Screens     │  │
│  └────┬─────┘ └────┬─────┘ └─────┬────┘ └────┬─────┘ └──────┬───────┘  │
│       └────────────┴─────────────┴───────────┴──────────────┘          │
│                         ↓  State (ViewModel + StateFlow)                │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                    ViewModels (Business Logic)                   │     │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────┐ │     │
│  │  │  Auth    │ │ Employe  │ │  Conge   │ │ Demande  │ │ Notif │ │     │
│  │  │ViewModel │ │ViewModel │ │ViewModel  │ │ViewModel │ │VM     │ │     │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └──┬───┘ │     │
│  └───────┴────────────┴────────────┴────────────┴──────────┘─────┘     │
│                         ↓  Retrofit / OkHttp                            │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                    Couche Réseau (Network)                      │     │
│  │  ┌────────────────────────────────────────────────────────┐    │     │
│  │  │  ApiService (Retrofit Interface)                        │    │     │
│  │  │  • Intercepteur JWT (Bearer Token)                      │    │     │
│  │  │  • Intercepteur Logging HTTP                            │    │     │
│  │  │  • Timeouts : 30s connect / 30s read                    │    │     │
│  │  └────────────────────────────────────────────────────────┘    │     │
│  └────────────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────────────┘
         │
         │ HTTPS / JSON
         ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                    API REST (Spring Boot 3 / Kotlin)                      │
├──────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────────────────┐   │
│  │   Controllers   │  │   Services     │  │  Security (JWT/BCrypt)   │   │
│  │  • Auth         │  │  • ExcelImport │  │  • JwtTokenUtil          │   │
│  │  • Employe      │  │                │  │  • JwtRequestFilter      │   │
│  │  • Conge        │  │                │  │  • CustomUserDetailsSvc  │   │
│  │  • DemandeAdmin │  │                │  │  • WebSecurityConfig     │   │
│  │  • Document     │  │                │  │                          │   │
│  │  • Notification │  │                │  │                          │   │
│  │  • ServiceDept  │  │                │  │                          │   │
│  └────────┬───────┘  └────────┬───────┘  └──────────┬───────────────┘   │
│           └──────────────────┴──────────────────────┘                    │
│                         ↓ Spring Data JPA / Hibernate                     │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Repositories + Entities (Employe, Conge, Demande, Document,   │     │
│  │                     Notification, ServiceDepartment, Utilisateur)│    │
│  └───────────────────────────┬────────────────────────────────────┘     │
└──────────────────────────────┼──────────────────────────────────────────┘
                               │
                               ▼
              ┌──────────────────────────────────────┐
              │     PostgreSQL (Supabase)             │
              │  Tables : employes, services,         │
              │  utilisateurs, conges,                │
              │  demandes_administratives,            │
              │  notifications, documents             │
              │  RLS (Row Level Security) activé      │
              └──────────────────────────────────────┘
```

### Flux de données

```
┌──────────────┐    HTTPS/JSON     ┌──────────────────┐    JDBC     ┌──────────────┐
│  Application  │ ────────────────▶ │  API Spring Boot │ ──────────▶ │  PostgreSQL  │
│  Android      │ ◀──────────────── │  (Hugging Face)  │ ◀────────── │  (Supabase)  │
│  (Kotlin)     │    JWT Token      └──────────────────┘    Results  └──────────────┘
└──────────────┘
       │
       │ Notifications push (WorkManager - polling 45s)
       ▼
┌──────────────────────────────────────────────────────────────┐
│  Notifications système Android avec NotificationChannel      │
│  • Canal HAUTE importance pour alertes temps réel             │
│  • Heads-up notifications pour approbation/refus             │
└──────────────────────────────────────────────────────────────┘
```

### Base de données (7 tables)

```
┌────────────────┐     ┌──────────────────┐
│   employes     │     │   services        │
├────────────────┤     ├──────────────────┤
│ id (PK)        │◀───▶│ id (PK)          │
│ nom_prenom     │     │ nom              │
│ nom_prenom_ar  │     │ division         │
│ cin (UQ)       │     │ direction        │
│ numero_admin   │     └──────────────────┘
│ grade          │
│ echelle        │     ┌──────────────────┐
│ echellon       │     │   utilisateurs    │
│ indice         │     ├──────────────────┤
│ service_id(FK) │     │ id (PK)          │
│ ... (20+ cols) │     │ username (UQ)    │
└────────┬───────┘     │ password (BCrypt)│
         │             │ role             │
         │             │ employe_id (FK)  │
         │             └──────────────────┘
         │
    ┌────┴────┐      ┌──────────────────────┐
    │ conges  │      │ demandes_administratives│
    ├─────────┤      ├──────────────────────┤
    │ id (PK) │      │ id (PK)              │
    │ employe │      │ employe_id (FK)      │
    │  _id(FK)│      │ type_document        │
    │ date_deb│      │ statut               │
    │ date_fin│      │ date_creation        │
    │ statut  │      └──────────────────────┘
    │ type    │
    └─────────┘      ┌──────────────────────┐
                     │   notifications      │
    ┌──────────┐     ├──────────────────────┤
    │ documents│     │ id (PK)              │
    ├──────────┤     │ employe_id (FK)      │
    │ id (PK)  │     │ titre                │
    │ employe  │     │ message              │
    │  _id(FK) │     │ lue (bool)           │
    │ nom      │     │ date_creation        │
    │ chemin   │     │ type / reference_id  │
    │ date_up  │     └──────────────────────┘
    └──────────┘
```

### Cycle de vie d'une demande de congé

```
1. Employé soumet une demande de congé
       │
       ▼
2. API : POST /api/conges/demande/{employeId}
   • Vérification : pas de dates dans le passé, max 60 jours, solde suffisant
   • Création du congé (statut = EN_ATTENTE)
   • Création d'une notification pour l'admin
       │
       ▼
3. Administrateur reçoit une notification push
       │
       ├──▶ PUT /api/conges/{id}/valider  → Notification à l'employé
       │
       └──▶ PUT /api/conges/{id}/refuser  → Notification à l'employé
       │
       ▼
4. Employé notifié du changement de statut (temps réel)
```

---

## 🔌 API Endpoints

### Authentification

| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/api/auth/login` | Connexion (body: `{"username", "password"}`) → JWT + rôle + employeId | Public |

### Employés

| Méthode | Endpoint | Description | Rôle requis |
|---|---|---|---|
| `GET` | `/api/employes` | Liste de tous les employés | Tout utilisateur |
| `GET` | `/api/employes/{id}` | Détail d'un employé | Tout utilisateur |
| `GET` | `/api/employes/search?query=` | Recherche par nom ou CIN | Tout utilisateur |
| `POST` | `/api/employes` | Ajouter un employé + créer son compte | ADMIN_RH |
| `PUT` | `/api/employes/{id}` | Modifier un employé | ADMIN_RH |
| `DELETE` | `/api/employes/{id}` | Supprimer un employé | ADMIN_RH |

### Congés (Leave)

| Méthode | Endpoint | Description | Rôle requis |
|---|---|---|---|
| `GET` | `/api/conges` | Liste de toutes les demandes | ADMIN_RH |
| `GET` | `/api/conges/employe/{employeId}` | Demandes d'un employé | Employé concerné |
| `GET` | `/api/conges/employe/{employeId}/solde` | Solde de congés restant | Employé concerné |
| `POST` | `/api/conges/demande/{employeId}` | Nouvelle demande de congé | Employé concerné |
| `PUT` | `/api/conges/{id}/valider` | Valider une demande | ADMIN_RH |
| `PUT` | `/api/conges/{id}/refuser` | Refuser une demande | ADMIN_RH |

### Demandes Administratives

| Méthode | Endpoint | Description | Rôle requis |
|---|---|---|---|
| `GET` | `/api/demandes` | Liste de toutes les demandes | ADMIN_RH |
| `GET` | `/api/demandes/employe/{employeId}` | Demandes d'un employé | Employé concerné |
| `POST` | `/api/demandes/employe/{employeId}` | Nouvelle demande documentaire | Employé concerné |
| `PUT` | `/api/demandes/{id}/traiter?nouveauStatut=` | Traiter une demande | ADMIN_RH |

### Documents & Décisions

| Méthode | Endpoint | Description | Rôle requis |
|---|---|---|---|
| `GET` | `/api/documents/decision/{employeId}` | Générer PDF décision administrative | ADMIN_RH |

### Notifications

| Méthode | Endpoint | Description | Rôle requis |
|---|---|---|---|
| `GET` | `/api/notifications/employe/{employeId}` | Notifications d'un employé | Employé concerné |
| `GET` | `/api/notifications/employe/{employeId}/non-lues` | Notifications non lues | Employé concerné |
| `PUT` | `/api/notifications/{id}/lire` | Marquer comme lue | Tout utilisateur |
| `DELETE` | `/api/notifications/{id}` | Supprimer une notification | Tout utilisateur |

### Services

| Méthode | Endpoint | Description |
|---|---|---|
| `GET` | `/api/services` | Liste des services/directions |

---

## 📱 Captures d'écran

> 📄 Les 22 captures d'écran ci-dessous sont extraites du dossier de livrables visuels. Cliquez sur une image pour l'agrandir.

### 🔐 Authentification & Interface

<p align="center">
  <img src="screenshots/02-authentification.png" alt="Authentification" width="350"/>
  <img src="screenshots/03-mode-clair-sombre.png" alt="Mode Clair/Sombre" width="350"/>
</p>

### 👑 Portail Administrateur (Gestion RH)

<p align="center">
  <img src="screenshots/04-admin-tableau-de-bord.png" alt="Tableau de bord" width="350"/>
  <img src="screenshots/05-admin-tableau-de-bord-2.png" alt="Tableau de bord 2" width="350"/>
</p>

<p align="center">
  <img src="screenshots/06-recherche-avancee.png" alt="Recherche avancée" width="350"/>
  <img src="screenshots/07-recherche-avancee-2.png" alt="Recherche avancée 2" width="350"/>
</p>

<p align="center">
  <img src="screenshots/08-ajout-fonctionnaire.png" alt="Ajout fonctionnaire" width="350"/>
  <img src="screenshots/09-ajout-fonctionnaire-2.png" alt="Ajout fonctionnaire 2" width="350"/>
</p>

<p align="center">
  <img src="screenshots/10-ajout-fonctionnaire-3.png" alt="Ajout fonctionnaire 3" width="350"/>
  <img src="screenshots/11-modification-agent.png" alt="Modification agent" width="350"/>
</p>

<p align="center">
  <img src="screenshots/12-validation-conges.png" alt="Validation congés" width="350"/>
  <img src="screenshots/13-validation-administrative.png" alt="Validation administrative" width="350"/>
</p>

<p align="center">
  <img src="screenshots/14-notifications-admin.png" alt="Notifications admin" width="350"/>
</p>

### 👤 Portail Employé (Espace Personnel)

<p align="center">
  <img src="screenshots/15-employe-portail.png" alt="Portail employé" width="350"/>
  <img src="screenshots/16-employe-dossier-numerique.png" alt="Dossier numérique" width="350"/>
</p>

<p align="center">
  <img src="screenshots/17-employe-details.png" alt="Détails agent" width="350"/>
  <img src="screenshots/18-demande-conge.png" alt="Demande congé" width="350"/>
</p>

<p align="center">
  <img src="screenshots/19-demande-conge-2.png" alt="Demande congé 2" width="350"/>
  <img src="screenshots/20-sollicitation-documents.png" alt="Sollicitation documents" width="350"/>
</p>

<p align="center">
  <img src="screenshots/21-notifications-employe.png" alt="Notifications employé" width="350"/>
</p>

---

## 🚀 Installation

### APK Android

Téléchargez le fichier APK depuis ce dépôt :

```
SGRH.apk
```

**Installation via ADB :**
```bash
adb install SGRH.apk
```

**Installation manuelle :**
1. Transférez le fichier `SGRH.apk` sur votre appareil Android
2. Ouvrez le gestionnaire de fichiers
3. Activez l'installation depuis sources inconnues (Paramètres → Sécurité)
4. Ouvrez le fichier APK et confirmez l'installation

### Configuration requise

- **Android** : API 21+ (Android 5.0 Lollipop minimum)
- **Espace de stockage** : ~150 Mo
- **Connexion Internet** : Requise pour l'authentification et la synchronisation

---



## 🗄️ Schémas de base de données

### Architecture et flux des données

**Schéma conceptuel de la base de données (7 tables)**

<img src="assets/deliverables/flux/bd%20xhema.PNG" alt="BD Schema"/>

**Architecture logicielle complète du système**

<img src="assets/deliverables/flux/RCHITECTURE%20LOGICIEL.PNG" alt="Architecture Logicielle"/>

**Diagramme des flux de données**

<img src="assets/deliverables/flux/FLUX%20DONNES.PNG" alt="Flux de Données"/>

**Parcours et rôles utilisateurs (Admin RH / Employé)**

<img src="assets/deliverables/flux/cheminrole.png" alt="Chemin des Rôles"/>

**Architecture en couches — Niveau base**

<img src="assets/deliverables/flux/COUCHE%20BSSE.PNG" alt="Couche Base"/>

**Architecture en couches — Niveau haut**

<img src="assets/deliverables/flux/COUCHE%20HUTE.PNG" alt="Couche Haute"/>

**Cas d'utilisation — Employé (profil, congés, documents, notifications)**

<img src="assets/deliverables/flux/CS%20EMP.PNG" alt="Cas d'usage EMP"/>

**Cas d'utilisation — Administrateur RH (gestion du personnel, validations)**

<img src="assets/deliverables/flux/CS%20RH.PNG" alt="Cas d'usage RH"/>

**Cas d'utilisation — Modules transverses**

<img src="assets/deliverables/flux/CS%20TRNSVERSE.PNG" alt="Cas d'usage Transverse"/>

**Diagramme de flux global des processus RH**

<img src="assets/deliverables/flux/FLO.PNG" alt="Flux"/>

**Schéma de sécurité (JWT, BCrypt, RLS Supabase)**

<img src="assets/deliverables/flux/SECURTIE%20FLU.PNG" alt="Sécurité"/>

**Diagramme de Gantt du projet**

<img src="assets/deliverables/flux/gin%20de%20temps.PNG" alt="Gantt"/>

---

## 🛠️ Technologies utilisées

### Frontend Mobile

| Technologie | Version | Utilisation |
|---|---|---|
| **Android SDK** | API 34 (target) | Plateforme mobile |
| **Kotlin** | 2.2.21 | Langage de programmation |
| **Jetpack Compose** | Dernière (BOM) | UI déclarative moderne |
| **Material Design 3** | — | Thème futuriste (glassmorphism) |
| **Navigation Compose** | — | Routage et navigation entre écrans |
| **Retrofit 2** | 2.9.0 | Client HTTP typé pour l'API REST |
| **OkHttp** | 4.x | Intercepteurs JWT + logging HTTP |
| **Gson** | — | Sérialisation/désérialisation JSON |
| **ViewModel** | — | Architecture MVVM (gestion d'état) |
| **StateFlow** | — | États réactifs pour l'UI |
| **Coroutines** | — | Programmation asynchrone |
| **WorkManager** | — | Notifications push polling (45s) |
| **DataStore / SharedPreferences** | — | Stockage local des préférences |

### Backend

| Technologie | Version | Utilisation |
|---|---|---|
| **Spring Boot** | 3.4.3 | Framework REST API |
| **Kotlin** | 2.2.21 | Langage backend |
| **Spring Data JPA / Hibernate** | — | ORM et mapping objet-relationnel |
| **Spring Security** | — | Authentification et autorisation |
| **JWT (jjwt)** | 0.11.5 | Tokens d'authentification (HS256) |
| **BCrypt** | — | Hachage des mots de passe |
| **Apache PDFBox** | 2.0.28 | Génération de documents PDF |
| **Apache POI** | 5.2.3 | Import Excel des données employés |
| **Maven** | — | Build et gestion des dépendances |

### Base de données

| Technologie | Utilisation |
|---|---|
| **PostgreSQL** | Base de données principale (Supabase) |
| **MySQL** | Base de développement local |
| **Supabase RLS** | Row Level Security pour la protection des données |
| **Spring Data JPA** | Requêtes et migrations automatiques |

### Déploiement

| Technologie | Utilisation |
|---|---|
| **Docker** | Conteneurisation de l'API backend |
| **Hugging Face Spaces** | Hébergement cloud du backend |
| **Hugging Face URL** | `https://sdawgxxx-sirh-grh.hf.space` |

---

## 📦 Livrables

Le dossier [`assets/deliverables/`](assets/deliverables/) contient l'ensemble des livrables du projet :

| Fichier | Description |
|---|---|
| 📄 `Livrables Visuels _ Dossier de captures d'écran de l'application mobile.pdf` | Dossier complet de 22 captures d'écran |
| 📁 `flux/` | Schémas d'architecture, flux, cas d'utilisation |
| 🖼️ `flux/bd xhema.PNG` | Schéma conceptuel de la base de données (7 tables) |
| 🖼️ `flux/cheminrole.png` | Parcours des rôles (Admin RH / Employé) |
| 🖼️ `flux/COUCHE BSSE.PNG` | Architecture — Couche basse |
| 🖼️ `flux/COUCHE HUTE.PNG` | Architecture — Couche haute |
| 🖼️ `flux/CS EMP.PNG` | Cas d'utilisation — Employé |
| 🖼️ `flux/CS RH.PNG` | Cas d'utilisation — Administrateur RH |
| 🖼️ `flux/CS TRNSVERSE.PNG` | Cas d'utilisation — Transverse |
| 🖼️ `flux/FLO.PNG` | Diagramme de flux global |
| 🖼️ `flux/FLUX DONNES.PNG` | Diagramme des flux de données |
| 🖼️ `flux/gin de temps.PNG` | Diagramme de Gantt du projet |
| 🖼️ `flux/RCHITECTURE LOGICIEL.PNG` | Architecture logicielle complète |
| 🖼️ `flux/SECURTIE FLU.PNG` | Schéma de sécurité (JWT, BCrypt, RLS) |

---

## 🔒 Sécurité et confidentialité

Conformément aux exigences du Conseil Provincial de Safi :

- **Authentification JWT obligatoire** : Aucun accès aux données sans token valide
- **Mots de passe BCrypt** : Chiffrement fort des mots de passe utilisateurs
- **Communications HTTPS** : Toutes les données transitent par TLS
- **RLS Supabase** : Row Level Security pour isoler les données par utilisateur
- **Rôles distincts** : `ADMIN_RH` (gestion complète) / `EMPLOYE` (accès restreint à son dossier)
- **Notifications push haute importance** : Alertes temps réel pour les validations
- **Tokens 24h** : Expiration automatique des sessions avec reconnexion requise

> ⚠️ **Note importante :** Ce dépôt ne contient **aucune donnée nominative** réelle des agents du Conseil Provincial de Safi. Les données présentées dans les captures d'écran sont des illustrations à but démonstratif.

---

## 👤 Auteur

**Saad Staili** — Stagiaire développeur

- 📧 Email : saadstaili1903@gmail.com
- 💼 LinkedIn : [Saad Staili](https://linkedin.com/in/saad-staili)
- 🐙 GitHub : [@StailiSaad](https://github.com/StailiSaad)

---

<p align="center">
  <em>Projet développé dans le cadre d'un stage d'étude au</em><br>
  <strong>Conseil Provincial de Safi</strong><br>
  <sub>© 2026 — Tous droits réservés</sub>
</p>
