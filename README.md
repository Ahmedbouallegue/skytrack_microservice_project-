# SkyTrack

## 🎯 Objectif du projet

**SkyTrack** est une plateforme moderne de gestion, de surveillance et d'analyse météorologique. Elle repose sur une **architecture microservices avec Spring Boot** : le système est découpé en **6 services indépendants** (un par membre de l'équipe). Chaque service possède sa propre base de données (PostgreSQL, MongoDB ou InfluxDB) et communique avec les autres via des appels réseau (REST / OpenFeign) ou des identifiants logiques.

### 🧩 Pourquoi cette répartition ?

- **Autonomie** : chaque membre développe sa propre brique sans perturber le code des autres.
- **Diversité des technologies** : le projet met en pratique plusieurs paradigmes de bases de données très recherchés en entreprise :
  - **PostgreSQL** (SQL relationnel) pour les données transactionnelles et structurées (Utilisateurs, Villes).
  - **MongoDB** (NoSQL orienté documents) pour les données flexibles ou semi-structurées (Prévisions, Alertes, Préférences).
  - **InfluxDB** (Time-Series) pour l'ingestion massive de données de capteurs en temps réel.

## 🔗 Analyse détaillée des relations entre les entités

Dans une architecture microservices pure, il n'y a **pas de clés étrangères physiques** (`@ManyToOne`, `@OneToMany` avec contrainte SQL) entre des entités qui appartiennent à des microservices différents. Les relations se font de deux manières :

- **Relations intra-services (physiques)** : à l'intérieur d'un même microservice, les entités sont directement reliées par des objets ou des identifiants.
- **Relations inter-services (logiques)** : les microservices se parlent en échangeant uniquement des IDs (ex. `cityId`, `userId`) via des appels API.

### 🔹 Microservice 1 : User Service (PostgreSQL)

**Entités :** `User` ↔ `Role`

- **Relation physique (intra-service)** : un `User` possède un rôle principal (`Role`). C'est une relation **Many-to-One (N..1)**.
- **Exemple** : l'utilisateur Ahmed a le rôle `ADMIN`. Techniquement, la table `user` contient une colonne `role_id` qui pointe vers la table `role`.

### 🔹 Microservice 2 : Location Service (PostgreSQL)

**Entités :** `City` ↔ `Region`

- **Relation physique (intra-service)** : une `City` (ville) est rattachée à une `Region` (région ou gouvernorat). C'est une relation **Many-to-One (N..1)**.
- **Exemple** : la ville de Tunis appartient à la région de Tunis / Grand Tunis. La table `city` possède une colonne `region_id`.

### 🔹 Microservice 3 : Forecast Service (MongoDB)

**Entités :** `Forecast` ↔ `WeatherDetail`

- **Relation physique / composition (intra-service)** : un bulletin de prévision (`Forecast`) est étroitement lié à ses détails techniques atmosphériques (`WeatherDetail`). C'est une relation **One-to-One / composition (1..1)**.
- **Lien logique avec les autres services** : le `Forecast` possède un champ `cityId`. Il ne contient pas l'objet `City` en entier, mais seulement son ID numérique (provenant du Location Service). Quand l'application veut afficher la météo de Paris, elle récupère le `Forecast` grâce au `cityId = 5` et va chercher le nom de la ville via une requête HTTP vers le Location Service.

### 🔹 Microservice 4 : Alert & Notification Service (MongoDB)

**Entités :** `Alert` ↔ `Notification`

- **Relation physique (intra-service)** : une alerte climatique (`Alert`) déclenche l'envoi de plusieurs notifications aux utilisateurs concernés (`Notification`). C'est une relation **One-to-Many (1..\*)**.
- **Liens logiques inter-services** :
  - L'`Alert` contient un `cityId` (pour savoir quelle ville est touchée, en lien avec le Location Service).
  - La `Notification` contient un `userId` (pour savoir à quel utilisateur l'envoyer, en lien logique avec le User Service).

### 🔹 Microservice 5 : IoT Sensor Service (InfluxDB)

**Entités :** `Station` ↔ `SensorMeasurement`

- **Relation temporelle (intra-service)** : une station météo physique (`Station`) envoie en continu des mesures de capteurs (`SensorMeasurement`). C'est une relation **One-to-Many (1..\*)** orientée flux temporel.
- **Fonctionnement** : chaque mesure enregistre un `timestamp`, un `stationId` et des métriques (température, humidité, pluie).

### 🔹 Microservice 6 : Analytics & Preferences Service (InfluxDB)

**Entités :** `HistoricalLog` (analyses) et `UserPreference` (préférences)

- **Relations logiques (inter-services)** :
  - `HistoricalLog` utilise un `cityId` pour stocker les moyennes climatiques d'une ville (lien avec le Location Service).
  - `UserPreference` utilise un `userId` pour retenir les préférences d'affichage d'un utilisateur (ex. Celsius ou Fahrenheit, alertes activées), en lien logique avec le User Service.

## 🗺️ Schéma récapitulatif des flux

```
[ Location Service ] <--- (cityId) ---> [ Forecast Service ]
        ^                                      ^
        | (cityId)                             | (cityId)
        v                                      v
[ Alert Service ]                        [ Analytics Service ]
[ User Service ] <--- (userId) --------> [ Notification / Preferences ]
```

## 💡 En résumé pour la soutenance / présentation

> **Question :** « Comment le service de prévision sait-il à quelle ville il appartient s'il n'y a pas de clé étrangère SQL ? »
>
> **Réponse type :** « Nos microservices sont totalement découplés pour garantir l'indépendance des bases de données. Le document MongoDB `Forecast` stocke uniquement l'attribut `cityId`. Pour afficher le nom complet de la ville, notre microservice effectue un appel REST synchrone (via Spring Cloud OpenFeign) vers le Location Service en utilisant cet ID. »

## Structure du projet

```
skytrack/
├── Backend/              # Code serveur (API, base de données)
├── Frontend/             # Interface utilisateur
├── Documentation/
│   ├── Presentation/     # Supports de présentation
│   └── Diagrams/         # Diagrammes (UML, architecture, etc.)
└── README.md
```

## Installation

_À compléter._

## Utilisation

_À compléter._
