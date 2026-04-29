# §6.4 — Dictionnaire de données

**Projet** : SupEvents
**Auteurs** : Samy Abdelmalek, Tristan Sanjuan
**Date** : 2026-04-29

> Extrait du TP 1.4 reproduit ici pour garantir la cohérence avec §7. Toute évolution doit être faite ici, pas masquée dans §7.

## Entités

### `User`
| Champ | Type | Contraintes | Source |
|---|---|---|---|
| `id` | UUID | PK | généré |
| `sub_oidc` | string | UNIQUE NOT NULL | SSO école (claim `sub`) |
| `email` | string | NOT NULL | SSO `userinfo` |
| `email_status` | enum(`valid`, `invalid`) | NOT NULL, default `valid` | mis à jour par `NotificationModule` |
| `name` | string | NOT NULL | SSO `userinfo` |
| `role` | enum(`student`, `organizer`, `admin`) | NOT NULL, default `student` | provisioning |
| `created_at` | timestamp | NOT NULL | |
| `deleted_at` | timestamp | nullable (soft delete RGPD) | |

### `Event`
| Champ | Type | Contraintes |
|---|---|---|
| `id` | UUID | PK |
| `organizer_id` | UUID | FK `User.id` |
| `title` | string | NOT NULL |
| `starts_at` | timestamp | NOT NULL |
| `capacity` | int | NOT NULL, > 0 |
| `seats_reserved` | int | NOT NULL, default 0, ≤ capacity |
| `status` | enum(`draft`, `published`, `cancelled`, `archived`) | NOT NULL |
| `version` | int | NOT NULL, default 0 (réservé pour locking optimiste futur) |

### `Ticket`
| Champ | Type | Contraintes |
|---|---|---|
| `id` | UUID | PK |
| `user_id` | UUID | FK `User.id`, NOT NULL |
| `event_id` | UUID | FK `Event.id`, NOT NULL |
| `status` | enum(`pending`, `confirmed`, `cancelled`, `used`) | NOT NULL |
| `idempotency_key` | UUID | NOT NULL, UNIQUE avec `user_id` |
| `qr_signature` | string | nullable, rempli au passage en `confirmed` |
| `expires_at` | timestamp | NOT NULL pour `pending` |
| `created_at`, `updated_at` | timestamp | |

### `NotificationPreference`
| Champ | Type | Contraintes |
|---|---|---|
| `user_id` | UUID | FK `User.id`, PK |
| `email_enabled` | bool | default true |
| `inapp_enabled` | bool | default true |
| `types_disabled` | string[] | array d'event types opt-out |

### `Notification` (in-app)
| Champ | Type | Contraintes |
|---|---|---|
| `id` | UUID | PK |
| `user_id` | UUID | FK |
| `type` | string | NOT NULL |
| `payload` | jsonb | NOT NULL |
| `read_at` | timestamp | nullable |
| `created_at` | timestamp | NOT NULL |

### `RefreshToken` (Redis)
| Champ | Type | TTL |
|---|---|---|
| `jti` | string (UUID v4) | clé Redis |
| `user_id` | UUID | |
| `hash` | string (sha256 du token) | |
| `created_at`, `last_used_at` | timestamp | TTL 30 jours |

### `IdempotencyEntry` (Redis ou table dédiée — webhook Stripe)
| Champ | Type |
|---|---|
| `event_id` | string (Stripe event.id) |
| `processed_at` | timestamp |
| TTL | 7 jours |

## Cohérence avec §7
Toutes les entités référencées par §7.1 (TicketModule), §7.2 (NotificationModule), §7.3 (AuthModule) figurent ci-dessus. Toute nouvelle entité introduite par une fiche module doit d'abord être ajoutée ici.
