<p align="center">
  <img src="./assets/ets_logo.png" alt="Logo ÉTS" width="220"/>
</p>

<p align="center"><i>Mandats des séances pratiques, Session Été 2026</i></p>
<p align="center">Préparé par <b>Ramy Ouabel</b> (chargé de laboratoire)</p>

---

> Ce repo contient le mandat de la **séance courante** dans ce README, et les mandats des séances passées dans le dossier [`archives/`](./archives/).
> Vérifiez ce repo avant chaque séance.

---

# Séances 5 et 6 : Authentification (JWT) + Logging structuré

**Durée :** 1 heure
**Projet :** UniversalMarketPlace (Plateforme d'enchères)

---

## Aperçu de la séance

1. **Auth microservice** : un nouveau microservice qui gère l'inscription, la connexion et l'émission d'un **JWT** (JSON Web Token).
2. **Protection des routes** : refuser l'accès aux endpoints sensibles si le JWT est absent ou invalide.
3. **Login page** côté frontend : envoyer les identifiants, recevoir le token, le stocker, le renvoyer dans les requêtes suivantes.
4. **Logs structurés (JSON)** par service : chaque service écrit ses logs en JSON sur `stdout`, avec les champs `ts`, `level`, `service`, `msg` au minimum.

> **Stack libre.** Le repo de démo est en Python/FastAPI avec PyJWT et passlib/bcrypt. Vous pouvez utiliser n'importe quelle stack tant que les **objectifs** ci-dessous sont atteints.

---

## À faire **avant** la séance

1. **Mettre à jour** le [repo de démo](https://github.com/olry/MGL844-Demo-Microservices) (un `auth-service` y a été ajouté) :
   ```bash
   cd MGL844-Demo-Microservices
   git pull
   docker compose up -d --build
   ```

---

## Objectifs de la séance

1. **Comprendre** le rôle d'un token JWT : signature, expiration, payload, vérification.
2. **Implémenter** un microservice d'authentification (inscription, connexion, émission de JWT).
3. **Protéger** au moins une route d'un autre microservice : accès refusé sans token valide.
4. **Brancher** la login page du frontend sur le service d'authentification.
5. **Émettre** des logs JSON structurés dans chaque service.

---

## Concept : pourquoi JWT ?

Un **JWT** est une chaîne signée que le serveur émet après une connexion réussie. Le client la stocke et la renvoie dans l'en-tête `Authorization: Bearer <token>` à chaque requête.

Avantages :
- **Stateless** : le serveur n'a rien à stocker côté session, la signature suffit à valider le token.
- **Portable** : tout microservice qui connaît la clé (ou la clé publique en RS256) peut vérifier le token sans appeler le service d'auth.
- **Court** : durée de vie limitée (ex. 1h), ce qui réduit l'impact d'un token volé.

Un JWT contient 3 parties séparées par des points : `header.payload.signature`. Le payload est en clair (base64), donc **ne jamais y mettre de mot de passe ou de donnée sensible**.

---

## Minimum obligatoire (toutes les équipes)

### 1. Microservice `auth-service`

Endpoints obligatoires :

| Méthode | Chemin | Description | Réponse |
|---|---|---|---|
| `GET` | `/health` | Vérification que le service tourne | `200` + `{"status": "ok"}` |
| `POST` | `/auth/register` | Créer un compte | `201` + `{id, username}` |
| `POST` | `/auth/login` | Vérifier les identifiants et émettre un JWT | `200` + `{access_token, token_type, expires_in}` ou `401` |
| `GET` | `/auth/me` | Retourner l'utilisateur courant (lit le JWT dans `Authorization`) | `200` + `{id, username}` ou `401` |

**Règles non-négociables :**
- **Hachage des mots de passe** avec bcrypt, argon2 ou équivalent. Jamais en clair en BD, jamais dans les logs.
- **JWT signé** avec un secret lu depuis une variable d'environnement (`JWT_SECRET`). Pas de secret hardcodé.
- **Expiration** : champ `exp` dans le payload, durée raisonnable (ex. 60 minutes).
- **Même message d'erreur** pour "utilisateur inconnu" et "mauvais mot de passe" (évite l'énumération des comptes).

### 2. Protection des routes

**Toutes les routes** de vos microservices (autres que `auth-service`) doivent refuser l'accès sans JWT valide. **Seule exception : `/health`**, qui doit rester accessible sans token (Docker et la gateway l'appellent pour vérifier que le service tourne).

Pour chaque route protégée :
- Pas de header `Authorization` : `401`.
- Token signé avec une autre clé : `401`.
- Token expiré : `401`.
- Token valide : la route répond normalement.

Côté `auth-service`, `/auth/login` et `/auth/register` restent évidemment **publics** (sinon personne ne peut se connecter).

> **Choix d'architecture libre :** vérification dans la gateway, dans chaque service, ou les deux. Justifiez brièvement dans le README.

### 3. Login page (frontend)

- Un formulaire `username` + `password` qui appelle `POST /auth/login` via la gateway.
- Stocker le token reçu (localStorage, sessionStorage, cookie httpOnly, etc.).
- Renvoyer le token dans `Authorization: Bearer <token>` pour les requêtes suivantes.
- Afficher un message d'erreur si l'authentification échoue.
- Si le token expire (401 reçu), rediriger vers la login page.

### 4. Logs structurés (JSON)

Chaque service écrit ses logs **en JSON** sur `stdout`, une ligne par log. Champs minimum :

```json
{"ts": "2026-05-27T10:15:32.123+00:00", "level": "INFO", "service": "auth-service", "msg": "login_success", "user_id": 42}
```

À logguer **au minimum** dans `auth-service` :
- `login_success` (info) avec `user_id` et `username`.
- `login_failed` (warning) avec `username` (jamais le mot de passe).
- `register_conflict` (warning) si username déjà pris.
- `token_invalid` (warning) sur tentative d'accès avec token invalide.

### 5. Pull Request

- Branche dédiée (`feat/auth`, `feat/jwt`, etc.).
- **PR vers `main` avec au moins 1 coéquipier en reviewer.**
- **Au moins 1 approbation requise avant le merge.**

---

## Exemple de référence (Python/FastAPI)

Le repo de démo `olry/MGL844-Demo-Microservices` contient maintenant `backend/services/auth-service/`. Voici les morceaux clés à comprendre. **Inspirez-vous-en pour votre stack**, ne copiez pas mécaniquement si ce n'est pas Python.

### Hachage du mot de passe (`security.py`)

```python
from passlib.context import CryptContext

_pwd = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(plain: str) -> str:
    return _pwd.hash(plain)

def verify_password(plain: str, hashed: str) -> bool:
    return _pwd.verify(plain, hashed)
```

### Émission d'un JWT (`security.py`)

```python
from datetime import datetime, timedelta, timezone
import jwt
from auth_app.config import settings

def create_access_token(user_id: int, username: str) -> str:
    now = datetime.now(timezone.utc)
    payload = {
        "sub": str(user_id),
        "username": username,
        "iat": int(now.timestamp()),
        "exp": int((now + timedelta(minutes=settings.jwt_expires_min)).timestamp()),
    }
    return jwt.encode(payload, settings.jwt_secret, algorithm=settings.jwt_algorithm)
```

### Route `/auth/login` (`controllers/auth.py`)

```python
@router.post("/login", response_model=TokenResponse)
async def login(payload: LoginRequest, db: AsyncSession = Depends(get_db)):
    user = await get_by_username(db, payload.username)
    if user is None or not verify_password(payload.password, user.password_hash):
        logger.warning("login_failed", extra={"username": payload.username})
        raise HTTPException(status_code=401, detail="invalid credentials")
    token = create_access_token(user_id=user.id, username=user.username)
    logger.info("login_success", extra={"user_id": user.id, "username": user.username})
    return TokenResponse(access_token=token, expires_in=settings.jwt_expires_min * 60)
```

### Dépendance "current_user" (à copier dans toute route protégée)

```python
async def current_user(authorization: str | None = Header(default=None)) -> dict:
    if authorization is None or not authorization.lower().startswith("bearer "):
        raise HTTPException(status_code=401, detail="missing bearer token",
                            headers={"WWW-Authenticate": "Bearer"})
    token = authorization.split(" ", 1)[1].strip()
    try:
        return decode_access_token(token)
    except Exception:
        raise HTTPException(status_code=401, detail="invalid or expired token",
                            headers={"WWW-Authenticate": "Bearer"})
```

### Logs JSON (`logging_config.py`, extrait)

```python
class JsonFormatter(logging.Formatter):
    def format(self, record):
        payload = {
            "ts": datetime.now(timezone.utc).isoformat(timespec="milliseconds"),
            "level": record.levelname,
            "service": "auth-service",
            "logger": record.name,
            "msg": record.getMessage(),
        }
        for k, v in record.__dict__.items():
            if k not in RESERVED:
                payload[k] = v
        return json.dumps(payload, default=str)
```

> Le fichier complet est dans le repo de démo. Lisez-le, lancez-le, puis adaptez à votre stack (Node/Winston, Java/Logback JSON, Go/zerolog, .NET/Serilog, etc.).

---

## Équivalents pour les autres stacks

| Stack | JWT | Hachage mot de passe | Logs JSON |
|---|---|---|---|
| Node.js | [`jsonwebtoken`](https://www.npmjs.com/package/jsonwebtoken) | [`bcrypt`](https://www.npmjs.com/package/bcrypt) ou `argon2` | [`pino`](https://github.com/pinojs/pino), [`winston`](https://github.com/winstonjs/winston) |
| Java/Spring | `spring-boot-starter-oauth2-resource-server` ou `jjwt` | `BCryptPasswordEncoder` | Logback `JsonLayout` |
| Go | `github.com/golang-jwt/jwt/v5` | `golang.org/x/crypto/bcrypt` | `zerolog`, `zap` |
| .NET | `Microsoft.AspNetCore.Authentication.JwtBearer` | `PasswordHasher<T>` | `Serilog` + `Serilog.Formatting.Json` |

---

## Tests manuels

Avec le repo de démo qui tourne (`docker compose up -d`), depuis l'extérieur (port `8000` = gateway) :

```bash
# 1. Créer un compte
curl -X POST http://localhost:8000/auth/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"hunter2!"}'

# 2. Se connecter et récupérer le token
TOKEN=$(curl -s -X POST http://localhost:8000/auth/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"hunter2!"}' | jq -r .access_token)
echo "$TOKEN"

# 3. Accéder à /auth/me avec le token (200)
curl -H "Authorization: Bearer $TOKEN" http://localhost:8000/auth/auth/me

# 4. Sans token (401)
curl -i http://localhost:8000/auth/auth/me

# 5. Avec un token bidon (401)
curl -i -H "Authorization: Bearer notarealtoken" http://localhost:8000/auth/auth/me
```

> Le double `/auth/auth/` n'est pas une faute de frappe : le premier `/auth` est le préfixe de routage de la gateway vers `auth-service`, le second est le préfixe du `APIRouter` interne au service. Vos équipes peuvent simplifier ce nommage si elles veulent.

---

## Pièges classiques

- **Secret JWT hardcodé** dans le code : à proscrire. Toujours via env var.
- **Mot de passe loggé** par erreur (ex. `logger.info(payload)` qui contient le payload Pydantic complet). Loguez des champs explicites, pas l'objet brut.
- **CORS oublié** : si la login page envoie le header `Authorization`, le serveur doit l'autoriser dans la config CORS (`allow_headers=["*"]` ou explicitement `Authorization`).
- **Token stocké dans `localStorage`** : pratique mais vulnérable au XSS. Acceptable pour ce labo, mais à mentionner comme limitation dans le README. Alternative : cookie httpOnly.
- **Pas d'expiration** sur le JWT : un token sans `exp` est valide à vie. À éviter même pour un labo.
- **Logs en texte libre** : `logger.info(f"user {x} connected")` perd la structure. Préférer `logger.info("login_success", extra={"user_id": x})`.

---

## Bonus optionnels

- **Refresh token** : un second token plus long pour renouveler l'access token sans redemander le mot de passe.
- **Rôles** dans le payload (`{"role": "admin"}`) et vérification côté route protégée.
- **RS256** (clé asymétrique) au lieu de HS256 : le service d'auth signe avec une clé privée, les autres services vérifient avec la clé publique. Plus propre en multi-service.
- **Corrélation des requêtes** : générer un `X-Request-ID` dans la gateway, le propager aux services, l'inclure dans tous les logs.
- **Rate limiting** sur `/login` (ex. 5 tentatives/min/IP) pour ralentir le brute force.

---

## Critères de "fait" (vérification rapide)

- [ ] `docker compose up -d --build` démarre le `auth-service` sans erreur.
- [ ] `POST /auth/register` crée un compte (mot de passe haché en BD).
- [ ] `POST /auth/login` avec bons identifiants retourne un JWT.
- [ ] `POST /auth/login` avec mauvais mot de passe retourne `401`.
- [ ] `GET /auth/me` avec token valide retourne l'utilisateur.
- [ ] `GET /auth/me` sans token retourne `401`.
- [ ] Toutes les routes des autres services (sauf `/health`) refusent l'accès sans token valide.
- [ ] Login page frontend fonctionnelle (login OK et login KO testés).
- [ ] Logs visibles en JSON dans `docker compose logs auth-service`.
- [ ] PR ouverte vers `main`, approuvée par au moins 1 coéquipier avant le merge.

---

*Document préparé par **Ramy Ouabel** (chargé de laboratoire).*
