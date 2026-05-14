# Exemple : changements typiques d'une PR liée à une issue

Cette page sert d'exemple. Elle montre **ce que vous devriez retrouver dans une PR** qui ferme l'issue [`#1 [Acheteur] Parcourir la marketplace`](../../issues/1).

> Note : dans un vrai projet, ce fichier serait remplacé par du code réel (composant React, endpoint FastAPI, tests, etc.). Ici c'est un placeholder pédagogique.

## Fichiers qui seraient ajoutés/modifiés

| Fichier | Changement |
|---|---|
| `frontend/src/pages/MarketplacePage.jsx` | Nouveau composant React qui affiche la liste des enchères |
| `frontend/src/api/encheres.js` | Client HTTP qui appelle `GET /encheres` |
| `frontend/src/components/EnchereCard.jsx` | Sous-composant : carte d'une enchère individuelle |
| `services/enchere/app/routes.py` | Nouvel endpoint `GET /encheres` (FastAPI) |
| `services/enchere/app/models.py` | Modèle SQLAlchemy `Enchere` |
| `services/enchere/tests/test_routes.py` | Tests pytest pour le nouvel endpoint |
| `frontend/src/pages/__tests__/MarketplacePage.test.jsx` | Tests Jest pour le composant |

## Commandes pour tester localement

```bash
docker compose up --build
# Frontend
open http://localhost:3000
# Backend (vérifier l'endpoint)
curl http://localhost:8000/encheres
```

## Notes pour les reviewers

- L'endpoint utilise SQLite (placeholder ; sera migré vers la vraie DB en séance 5).
- Le filtre par catégorie est implémenté côté Backend uniquement (pour l'instant).
- Pas de pagination encore (issue future à créer si besoin).
