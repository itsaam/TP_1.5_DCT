# Index des ADR — SupEvents

Ce dossier contient les Architecture Decision Records du projet SupEvents, au format Nygard.

| ID | Titre | Statut | Date |
|---|---|---|---|
| [ADR-001](ADR-001.md) | Adopter un verrou pessimiste PostgreSQL pour l'allocation de la dernière place | Accepté | 2026-04-29 |
| [ADR-002](ADR-002.md) | Appliquer un retry exponentiel borné avec bascule en DLQ pour les événements asynchrones | Accepté | 2026-04-29 |
| [ADR-003](ADR-003.md) | Émettre un JWT applicatif court doublé d'un refresh token stateful en Redis | Accepté | 2026-04-29 |

## Convention

- Numérotation incrémentale, jamais réutilisée même en cas de dépréciation.
- Un ADR déprécié reste dans le dossier ; son statut passe à `Déprécié` ou `Remplacé par ADR-XXX`.
- Tout nouvel ADR utilise [`template.md`](template.md).
