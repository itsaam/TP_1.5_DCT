# §8 — Tableau synoptique des API et événements

**Projet** : SupEvents
**Auteurs** : Samy Abdelmalek, Tristan Sanjuan
**Date** : 2026-04-29

> Extrait du TP 1.4 reproduit ici pour garantir la cohérence avec §7.

## §8.1 — Endpoints REST

| Module | Méthode | Chemin | Auth | Codes retour | Réf §7 |
|---|---|---|---|---|---|
| Ticket | POST | `/api/events/{eventId}/tickets` | JWT | 201, 400, 401, 403, 409, 422 | §7.1 |
| Ticket | GET | `/api/tickets/{ticketId}` | JWT | 200, 401, 403, 404 | §7.1 |
| Ticket | GET | `/api/tickets/{ticketId}/qr` | JWT | 200, 401, 403, 404, 410 | §7.1 |
| Ticket | POST | `/api/tickets/{ticketId}/cancel` | JWT | 204, 401, 403, 404, 409 | §7.1 |
| Ticket | POST | `/api/tickets/{ticketId}/validate` | JWT (organizer) | 200, 401, 403, 404, 409 | §7.1 |
| Notif | GET | `/api/users/me/notifications` | JWT | 200, 401 | §7.2 |
| Notif | PATCH | `/api/users/me/notifications/{id}/read` | JWT | 204, 401, 404 | §7.2 |
| Notif | GET | `/api/users/me/notification-preferences` | JWT | 200, 401 | §7.2 |
| Notif | PUT | `/api/users/me/notification-preferences` | JWT | 200, 400, 401 | §7.2 |
| Auth | GET | `/api/auth/login` | — | 302 | §7.3 |
| Auth | GET | `/api/auth/callback` | — | 302, 400, 401 | §7.3 |
| Auth | POST | `/api/auth/refresh` | refresh cookie | 200, 401 | §7.3 |
| Auth | POST | `/api/auth/logout` | JWT | 204, 401 | §7.3 |
| Auth | GET | `/api/auth/me` | JWT | 200, 401 | §7.3 |

## §8.2 — Événements RabbitMQ

### Publiés

| Nom | Topic | Producteur | Consommateurs |
|---|---|---|---|
| `ticket.reserved` | `tickets.events` | TicketModule | PaymentModule |
| `ticket.confirmed` | `tickets.events` | TicketModule | NotificationModule |
| `ticket.cancelled` | `tickets.events` | TicketModule | PaymentModule, NotificationModule |
| `ticket.used` | `tickets.events` | TicketModule | (analytics) |
| `notification.delivered` | `notifications.events` | NotificationModule | (métriques) |
| `notification.failed.permanent` | `notifications.events` | NotificationModule | (alertes ops) |
| `user.provisioned` | `auth.events` | AuthModule | UserModule, NotificationModule |
| `user.role.changed` | `auth.events` | AuthModule | (audit) |
| `user.session.revoked` | `auth.events` | AuthModule | (audit) |

### Schémas JSON (extraits)

`ticket.reserved` :
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["ticketId", "userId", "eventId", "amount", "currency", "occurredAt"],
  "properties": {
    "ticketId": { "type": "string", "format": "uuid" },
    "userId": { "type": "string", "format": "uuid" },
    "eventId": { "type": "string", "format": "uuid" },
    "amount": { "type": "integer", "minimum": 0 },
    "currency": { "type": "string", "enum": ["EUR"] },
    "occurredAt": { "type": "string", "format": "date-time" }
  }
}
```

`ticket.confirmed` :
```json
{
  "type": "object",
  "required": ["ticketId", "userId", "eventId", "qrSignature", "occurredAt"],
  "properties": {
    "ticketId": { "type": "string", "format": "uuid" },
    "userId": { "type": "string", "format": "uuid" },
    "eventId": { "type": "string", "format": "uuid" },
    "qrSignature": { "type": "string" },
    "occurredAt": { "type": "string", "format": "date-time" }
  }
}
```

`user.provisioned` :
```json
{
  "type": "object",
  "required": ["userId", "subOidc", "email", "role", "occurredAt"],
  "properties": {
    "userId": { "type": "string", "format": "uuid" },
    "subOidc": { "type": "string" },
    "email": { "type": "string", "format": "email" },
    "role": { "type": "string", "enum": ["student", "organizer", "admin"] },
    "occurredAt": { "type": "string", "format": "date-time" }
  }
}
```

(Les autres schémas suivent le même pattern : tous les événements portent `occurredAt` en ISO 8601 et un identifiant d'agrégat racine.)

## §8.3 — Cohérence avec §7

| Vérification | État |
|---|---|
| Tous les endpoints de §7 figurent ici | ✅ |
| Tous les événements publiés/consommés de §7 figurent ici | ✅ |
| Tous les codes HTTP de §7 sont annoncés ici | ✅ |
| Toutes les entités référencées sont au §6.4 | ✅ |
