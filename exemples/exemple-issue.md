# Exemple d'une bonne issue (user story)

Voici un exemple de la **forme** que devrait avoir une issue dans votre board pour qu'elle soit utile à toute l'équipe. Copiez ce gabarit pour chacune de vos user stories.

---

## Titre de l'issue (court, orienté action)

> Exemple : `[Acheteur] Parcourir la marketplace`

Format conseillé : `[Acteur ou Microservice] Action courte`. Reste lisible dans une liste sans avoir à ouvrir l'issue.

---

## Description (user story)

**En tant que** Acheteur,
**je veux** parcourir la liste des enchères en cours sur la plateforme,
**afin de** trouver un produit qui m'intéresse et pouvoir miser dessus.

---

## Contexte / motivation

Ce flux est l'entrée principale dans la plateforme pour un Acheteur. Sans cet écran, l'utilisateur ne peut pas découvrir les produits et donc pas miser. C'est l'une des user stories prioritaires de la Phase 1.

Référence cas d'utilisation : *Parcourir la marketplace* (voir diagramme du périmètre dans le README).

---

## Critères d'acceptation

Conditions qui rendent l'issue « faite ». À cocher au fur et à mesure.

- [ ] La page affiche une **liste d'enchères** (titre, prix de départ, image, date de fin).
- [ ] L'utilisateur peut **filtrer** par catégorie ou par prix.
- [ ] Cliquer sur une enchère ouvre l'écran *Détail d'une enchère*.
- [ ] La page se charge en moins de 2 secondes avec 50 enchères affichées.
- [ ] Le design respecte la maquette Figma associée.

---

## Tâches techniques

Découpage en sous-tâches concrètes pour les développeurs.

- [ ] Créer le composant `MarketplacePage` dans le Frontend.
- [ ] Créer l'endpoint `GET /encheres` dans le microservice `Enchère`.
- [ ] Brancher le Frontend sur l'endpoint via le Gateway.
- [ ] Ajouter le filtre par catégorie côté Backend (paramètre de requête).
- [ ] Tests unitaires sur le composant et l'endpoint.

---

## Notes / Liens

- **Maquette Figma :** `<lien vers l'écran "Parcourir la marketplace">`
- **Microservices concernés :** `Enchère`, `Produit`, `Gateway`
- **Dépend de :** issue #X (« Init microservice Enchère »)
- **Bloque :** issue #Y (« [Acheteur] Détail d'une enchère »)

---

## Métadonnées de l'issue

| Champ | Valeur |
|---|---|
| **Labels** | `user-story`, `frontend`, `acheteur`, `phase-1` |
| **Milestone** | Phase 1 : Conception |
| **Assigné à** | `@membre-equipe` |
| **Effort estimé** | 4 à 6 heures |
| **Priorité** | Haute |

---

## Pourquoi cette structure ?

- Un **titre court** permet de scanner le board en un coup d'œil.
- La **user story** force à se rappeler **pour qui** on construit.
- Les **critères d'acceptation** définissent « fait » sans ambiguïté (utile pour la revue de PR).
- Les **tâches techniques** transforment l'intention en travail concret pour le sprint.
- Les **liens** (Figma, microservices, dépendances) évitent de devoir chercher l'info ailleurs.
- Les **métadonnées** alimentent le rapport Phase 1 (priorité, effort, etc.).
