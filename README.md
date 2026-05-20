<p align="center">
  <img src="./assets/ets_logo.png" alt="Logo ÉTS" width="220"/>
</p>

<p align="center"><i>Mandats des séances pratiques, Session Été 2026</i></p>
<p align="center">Préparé par <b>Ramy Ouabel</b> (chargé de laboratoire)</p>

---

> Ce repo contient le mandat de la **séance courante** dans ce README, et les mandats des séances passées dans le dossier [`archives/`](./archives/).
> Vérifiez ce repo avant chaque séance.

---

# Séance 4 : API Gateway + premier microservice

**Durée :** 1 heure
**Projet :** UniversalMarketPlace (Plateforme d'enchères)

---

## Aperçu de la séance

1. **Rattrapage Séance 3** (si pas terminé) : Figma, framework init + PR, README à jour.
2. **API Gateway** : un point d'entrée unique qui route vers vos microservices.
3. **Premier microservice** (au choix de l'équipe, idéalement le 1er de votre liste priorisée de séance 3) : avec persistance (SQLite recommandé) et endpoints **POST** + **GET** pour créer/lister la ressource.
4. **Docker Compose** : tout démarre avec une seule commande (`docker compose up`).

> **Stack libre.** Le repo de démo est en **Python/FastAPI**, mais vous pouvez utiliser n'importe quelle stack (Node/Express, Java/Spring Boot, Go, .NET, etc.) - tant que les **objectifs** ci-dessous sont atteints et que tout tourne dans Docker.

> Détails ci-dessous. Bonus optionnels en bas.

---

## À faire **avant** la séance

1. **Installer Docker Desktop** : https://www.docker.com/products/docker-desktop/
2. **Cloner et démarrer** le [repo de démo](https://github.com/olry/MGL844-Demo-Microservices) (Gateway + 2 services + NATS) pour voir une référence qui tourne :
   ```bash
   git clone https://github.com/olry/MGL844-Demo-Microservices.git
   cd MGL844-Demo-Microservices
   cp .env.example .env       # Windows : copy .env.example .env
   docker compose up -d --build
   ```
   Puis ouvrez http://localhost:3000 et testez les endpoints.
3. **Lire** rapidement `backend/gateway/main.py` et `backend/services/hello-service/` dans le repo de démo : c'est la **structure de référence** dont vous pouvez vous inspirer pour vos propres services.

---

## Rattrapage Séance 3 (si pas terminé)

Si votre équipe n'a pas fini le mandat de la [séance 3](./archives/seance-03-prototypage-ui-frontend.md), commencez par ça :

- [ ] Lien Figma (3 écrans) dans le README
- [ ] Framework Frontend initialisé sur une branche + PR vers `main` approuvée par au moins 1 coéquipier
- [ ] README à jour : framework choisi (avec justification), liste **priorisée** des microservices, première liste de technos
- [ ] `.gitignore` propre (pas de `node_modules/` commit)

**Date de remise Phase 1 :** voir [ENA](https://ena.etsmtl.ca/) sous l'onglet *Projet en équipe* (l'énoncé du projet ne fixe pas de date, elle est publiée sur ENA).

---

## Objectifs de la séance

1. **Comprendre** le rôle d'une API Gateway dans une architecture microservices.
2. **Implémenter** une gateway qui route vers un microservice interne.
3. **Implémenter** un microservice complet (au choix de l'équipe) avec persistance.
4. **Conteneuriser** le tout avec Docker Compose.
5. **Tester** les endpoints (POST création, GET liste).

> **Choix techniques libres** (langage, framework, ORM). Le repo de démo `olry/MGL844-Demo-Microservices` est un exemple en Python/FastAPI - inspirez-vous-en, mais utilisez la stack qui convient à votre équipe.

---

## Minimum obligatoire (toutes les équipes)

### 1. Structure du repo

Créez un dossier `backend/` (à côté de `frontend/`) avec **un dossier pour la gateway** et **un dossier pour votre premier microservice**, chacun avec son propre `Dockerfile`. Exemple :

```
backend/
├── docker-compose.yml          (peut aussi être à la racine)
├── gateway/
│   ├── Dockerfile
│   └── ... (code source dans la stack de votre choix)
└── services/
    └── <votre-service>/        (nom au choix : produit, enchere, utilisateur, etc.)
        ├── Dockerfile
        └── ... (code source + fichier de BD SQLite)
```

> Inspirez-vous de [`olry/MGL844-Demo-Microservices`](https://github.com/olry/MGL844-Demo-Microservices) (`backend/gateway/` et `backend/services/hello-service/`). C'est un exemple en Python/FastAPI - adaptez à votre stack.

### 2. API Gateway

- Une seule application exposée sur le port **8000** (ajustable si justifié).
- Une **table de routage** simple associant chaque préfixe d'URL à l'URL du service interne correspondant.
  - Exemple : `/<ressource>/*` est routé vers `http://<votre-service>:8000/*` (ex. `/produits/*` vers `http://produit:8000/*`).
- Un endpoint **`GET /health`** qui répond `200` avec `{"status": "ok"}`.
- Une **doc Swagger / OpenAPI valide** exposée (ex. `/docs` ou `/swagger`), listant les routes proxyfiées.
- **CORS** activé pour permettre au Frontend de l'appeler.

> Pourquoi une gateway ? Le Frontend n'appelle **qu'un seul point d'entrée**. La gateway sait quel service contacter. Si demain vous changez l'adresse du service `produit`, le Frontend n'a rien à changer.

### 3. Premier microservice (au choix de l'équipe)

Choisissez **un** microservice à implémenter (idéalement le 1er de votre liste priorisée de séance 3 : produit, enchere, utilisateur, portefeuille, etc.). Le reste de cette section décrit le minimum attendu, peu importe la ressource choisie.

- Persistance : **SQLite recommandé** (fichier `.db`, monté en volume Docker pour ne pas perdre les données entre `docker compose down`). Une autre BD légère est acceptée si justifiée.
- Utilisez un **ORM ou un client BD propre** à votre stack (SQLAlchemy, Prisma, JPA, GORM, EF Core, etc.). Pas de SQL brut concaténé.
- Modèle minimum : `id` + au moins 3 attributs métier pertinents pour la ressource choisie (ex. pour `produit` : `nom`, `description`, `prix`, `date_disponibilite`).

**Endpoints obligatoires** (remplacez `<ressource>` par le nom au pluriel de votre ressource) :

| Méthode | Chemin | Description | Réponse |
|---|---|---|---|
| `GET` | `/health` | Vérification que le service tourne | `200` + `{"status": "ok"}` |
| `POST` | `/<ressource>` | Créer une ressource | `201` + la ressource créée (avec `id`) |
| `GET` | `/<ressource>` | Lister toutes les ressources | `200` + liste |
| `GET` | `/<ressource>/{id}` | Détail d'une ressource | `200` ou `404` |

**Doc API obligatoire :** chaque service expose sa propre doc **Swagger / OpenAPI valide** (ex. `/docs`, `/swagger`, ou équivalent selon le framework). Elle doit refléter les endpoints réels et leurs schémas de requête/réponse.

> Validez les entrées avec un **mécanisme de validation natif** de votre framework (Pydantic, class-validator, Joi, Bean Validation, etc.). Pas de validation maison fragile.

### 4. Docker Compose

Un seul fichier `docker-compose.yml` qui démarre :
- `gateway` (port `8000` exposé)
- votre microservice (pas exposé à l'hôte, seulement sur le réseau Docker interne)

```bash
docker compose up -d --build
```

doit tout démarrer. Et `docker compose down` doit tout arrêter proprement.

### 5. Tests manuels

Documentez les commandes `curl` (ou captures Postman) montrant, pour la ressource que vous avez choisie :

1. **GET** `/health` sur la **gateway** retournant `200`.
2. **GET** `/<ressource>/health` (via la gateway, routé vers votre microservice) retournant `200`.
3. **POST** créant une ressource via la gateway. Exemple si vous avez choisi `produit` :
   ```bash
   curl -X POST http://localhost:8000/produits \
     -H "Content-Type: application/json" \
     -d '{"nom":"Vélo","description":"Vélo de route","prix":350.00,"date_disponibilite":"2026-06-01"}'
   ```
4. **GET** listant les ressources via la gateway.
5. **GET** d'une ressource inexistante retournant `404`.

### 6. Pull Request

- Travail sur une **branche** (`backend-init`, `feat/gateway-produit`, etc.).
- **PR** vers `main` avec **au moins 1 coéquipier en reviewer**.
- **Au moins 1 approbation requise avant le merge.** Ne pas merger sa propre PR sans revue.

---

## À finir en dehors de la séance (obligatoire)

- **Brancher le Frontend** (séance 3) sur la gateway : au moins un écran qui affiche les données réelles de votre microservice (ex. liste des ressources via `GET /<ressource>`). À faire en dehors de l'heure de labo si pas le temps en classe.

---

## Livrables avant la prochaine séance

| Livrable | Où | Obligatoire |
|---|---|:---:|
| Dossier `backend/gateway/` (code + Dockerfile) | Repo Github | Oui |
| Dossier `backend/services/<votre-service>/` (code + persistance + Dockerfile) | Repo Github | Oui |
| `docker-compose.yml` qui démarre les 2 services | Repo Github | Oui |
| Endpoints POST/GET `/<ressource>` fonctionnels via la gateway | Repo Github | Oui |
| `/health` sur gateway **et** sur votre microservice (GET simple, 200) | Repo Github | Oui |
| Doc Swagger / OpenAPI valide pour gateway **et** votre microservice | Repo Github | Oui |
| Exemples `curl` documentés dans le README (health, POST, GET, 404) | README | Oui |
| PR vers `main` approuvée par au moins 1 coéquipier avant merge | Repo Github | Oui |
| Frontend branché sur la gateway (au moins 1 écran avec données réelles) | Repo Github | Oui (peut être fini hors séance) |

---

## Outils recommandés

- **Docker Compose** (obligatoire) : https://docs.docker.com/compose/
- **Repo de référence** (Python/FastAPI) : https://github.com/olry/MGL844-Demo-Microservices

**Stacks possibles** (choisissez celle qui convient à l'équipe) :
- Python : [FastAPI](https://fastapi.tiangolo.com/) + [SQLAlchemy](https://docs.sqlalchemy.org/) + [Pydantic](https://docs.pydantic.dev/)
- Node.js : [Express](https://expressjs.com/) ou [NestJS](https://nestjs.com/) + [Prisma](https://www.prisma.io/) / [TypeORM](https://typeorm.io/)
- Java : [Spring Boot](https://spring.io/projects/spring-boot) + JPA
- Go : [Gin](https://gin-gonic.com/) ou [Echo](https://echo.labstack.com/) + [GORM](https://gorm.io/)
- .NET : [ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/) + Entity Framework Core

Tester les endpoints : `curl`, [Postman](https://www.postman.com/), [HTTPie](https://httpie.io/), ou la **doc OpenAPI/Swagger** si votre framework la génère.

---

## Conseils

- **Lancez `docker compose up --build` tôt.** La première build télécharge plusieurs images Python, ça prend du temps.
- **Une responsabilité par service.** La gateway **ne contient pas** de logique métier - elle route, c'est tout.
- **Logs lisibles.** Loguez chaque requête entrante dans la gateway - vous allez vous en remercier à la séance 5 (observabilité).
- **`.gitignore`** : excluez les fichiers de BD (`*.db`), les caches de votre langage (`__pycache__/`, `node_modules/`, `target/`, `bin/obj/`, etc.), les `.venv/` et `.env`.
- **Ne hard-codez pas les URLs** des services internes : utilisez les **noms de services Docker** (ex. `http://<votre-service>:8000`) - c'est le DNS interne de Docker Compose.

> Pourquoi pas de logique métier dans la gateway : si demain vous splittez un service en deux, seule la table de routage de la gateway change. Le Frontend ne voit rien.

---

## Critères de "fait" (vérification rapide)

- [ ] `docker compose up -d --build` à la racine démarre gateway + microservice sans erreur.
- [ ] `GET http://localhost:8000/health` (gateway) répond `200`.
- [ ] `GET http://localhost:8000/<ressource>/health` (microservice via gateway) répond `200`.
- [ ] **Doc Swagger valide** de la **gateway** accessible (ex. `http://localhost:8000/docs`).
- [ ] **Doc Swagger valide** du **microservice** accessible (via la gateway ou en direct sur le réseau Docker).
- [ ] `POST http://localhost:8000/<ressource>` crée une ressource et retourne `201` + l'objet créé.
- [ ] `GET http://localhost:8000/<ressource>` retourne la liste (incluant la ressource créée).
- [ ] `GET http://localhost:8000/<ressource>/{id_inexistant}` retourne `404`.
- [ ] Les données persistent après `docker compose down` puis `up` (volume monté).
- [ ] PR ouverte vers `main`, approuvée par au moins 1 coéquipier avant le merge.
- [ ] `.gitignore` ne contient pas `*.db`, `__pycache__/`, etc. commit.

---

*Document préparé par **Ramy Ouabel** (chargé de laboratoire).*
