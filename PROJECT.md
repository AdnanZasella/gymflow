# GymFlow — Fullstack Java Repetitionsprojekt

> Detta dokument är källan till sanning för projektet. Uppdatera "Status"-sektionen
> längst ner efter varje avslutad deluppgift. Klistra in relevant del av detta
> dokument i nya konversationer med Claude för att ge kontext utan att behöva
> förklara om från scratch.

---

## 1. Projektöversikt

**Namn:** GymFlow
**Syfte:** Repetitionsprojekt för att återfå kunskap efter Fullstack Java-programmet
på Chas Academy. Täcker Java, PostgreSQL, webbutveckling, Backend API, säkerhet,
DevOps och produktion/deployment.

**Domän:** Bokningssystem för gym/träningspass (t.ex. yoga, spinning, crossfit,
PT-pass). En admin/tränare skapar pass i ett schema. Medlemmar registrerar sig,
loggar in och bokar sig på pass, med begränsat antal platser per pass.

**Kontext:** Projektet är tänkt att användas som portfolio-projekt vid LIA-sökning,
så namngivning, README och kodkvalitet bör hålla en nivå som ser bra ut för
potentiella handledare/arbetsgivare att titta på.

**Lärandemål utöver repetition:** Lära sig React från grunden (användaren har
tidigare bara jobbat med vanilla HTML/CSS/JS + Vite, ingen React-erfarenhet).

---

## 2. Teknikstack

| Del | Teknik |
|---|---|
| Backend | Java 21, Spring Boot 4.1 (Spring Framework 7) |
| Databas | PostgreSQL |
| ORM | Spring Data JPA / Hibernate |
| Auth | Spring Security + JWT |
| Frontend | React (Vite) |
| Containerisering | Docker + docker-compose |
| CI/CD | GitHub Actions |
| API-testning | Postman |
| IDE | IntelliJ IDEA (Community/Free) |
| Deployment | TBD (Railway/Render/Fly.io – gratis-tier) |

---

## 3. Funktionell scope

### 3.1 Auth & användare
- [ ] Registrering (email, lösenord, namn)
- [ ] Lösenordshashning med BCrypt
- [ ] Inloggning → returnerar JWT
- [ ] Roller: `USER` och `ADMIN`
- [ ] Skyddade endpoints beroende på roll
- [ ] Egen profil (visa/uppdatera egna uppgifter)

### 3.2 Pass/sessions (admin-hanterat)
- [ ] Admin: skapa träningspass (titel, typ t.ex. Yoga/Spinning/Crossfit,
      beskrivning, tid, plats/sal, tränare, kapacitet)
- [ ] Admin: redigera pass
- [ ] Admin: ta bort pass
- [ ] Alla: lista kommande pass (t.ex. veckoschema)
- [ ] Alla: se detaljer om ett specifikt pass
- [ ] (Ev. senare) Filtrera på passtyp (t.ex. bara "Yoga")

### 3.3 Bokning
- [ ] User: boka ett pass (om platser finns kvar)
- [ ] User: avboka ett pass
- [ ] User: se sina egna bokningar
- [ ] Kapacitetslogik: förhindra överbokning (race condition-säkert)
- [ ] Admin: se vilka som bokat ett visst pass

### 3.4 Extra "production-like" funktioner
- [ ] Sökning/filtrering av pass (datum, typ)
- [ ] Paginering av listor
- [ ] Rate limiting på login-endpoint (skydd mot brute force)
- [ ] Input-validering (backend: Bean Validation, frontend: formulärvalidering)
- [ ] Globala, tydliga felmeddelanden + korrekta statuskoder
- [ ] Loggning (t.ex. via SLF4J)

---

## 4. Arkitektur (backend, lagerindelning)

```
Controller  → tar emot HTTP-requests, validerar input
Service     → affärslogik (t.ex. "finns det platser kvar?")
Repository  → pratar med databasen (Spring Data JPA)
Entity      → databastabeller som Java-klasser
DTO         → vad som faktiskt skickas ut/in i API:et (aldrig exponera Entity direkt)
```

---

## 5. Databasschema (utkast)

**users**
| Kolumn | Typ | Not |
|---|---|---|
| id | UUID/bigint (PK) | |
| email | varchar, unik | |
| password_hash | varchar | BCrypt |
| full_name | varchar | |
| role | varchar/enum | USER / ADMIN |
| created_at | timestamp | |

**sessions** (träningspass)
| Kolumn | Typ | Not |
|---|---|---|
| id | UUID/bigint (PK) | |
| title | varchar | t.ex. "Morgonyoga" |
| type | varchar/enum | Yoga, Spinning, Crossfit, PT, osv. |
| description | text | |
| start_time | timestamp | |
| duration_minutes | int | |
| location | varchar | t.ex. "Sal 2" |
| capacity | int | max antal platser |
| instructor_id | FK → users.id | tränare (kan vara egen roll: TRAINER) |
| created_at | timestamp | |

**bookings** (kopplingstabell med extra data → inte ren `@ManyToMany`)
| Kolumn | Typ | Not |
|---|---|---|
| id | UUID/bigint (PK) | |
| user_id | FK → users.id | |
| session_id | FK → sessions.id | |
| booked_at | timestamp | |
| status | varchar/enum | CONFIRMED / CANCELLED |

Relation: **users ↔ sessions** via **bookings** (many-to-many med extra fält).

---

## 6. API-endpoints (utkast, fylls på under Fas 1–3)

| Metod | Path | Auth | Beskrivning |
|---|---|---|---|
| POST | /api/auth/register | Publik | Registrera ny användare |
| POST | /api/auth/login | Publik | Logga in, få JWT |
| GET | /api/sessions | Publik/USER | Lista kommande pass |
| GET | /api/sessions/{id} | Publik/USER | Detaljer om ett pass |
| POST | /api/sessions | ADMIN | Skapa nytt pass |
| PUT | /api/sessions/{id} | ADMIN | Redigera pass |
| DELETE | /api/sessions/{id} | ADMIN | Ta bort pass |
| POST | /api/bookings | USER | Boka ett pass |
| DELETE | /api/bookings/{id} | USER | Avboka |
| GET | /api/bookings/me | USER | Mina bokningar |
| GET | /api/sessions/{id}/bookings | ADMIN | Vilka har bokat detta pass |

---

## 7. Säkerhet — konkret implementation

- [ ] Lösenord: aldrig klartext, alltid BCrypt-hashat
- [ ] JWT genereras vid login, skickas som `Authorization: Bearer <token>`
- [ ] Endpoint-skydd: `/api/admin/**` eller motsvarande kräver ADMIN-roll
- [ ] Rate limiting: t.ex. max 5 inloggningsförsök/minut/IP (Bucket4j eller egen lösning)
- [ ] Input-validering: `@Valid` + `@NotBlank`, `@Email`, `@Size` osv.
- [ ] CORS-konfiguration (backend och frontend på olika portar lokalt)
- [ ] Hemligheter (DB-lösenord, JWT-secret) via miljövariabler, aldrig hårdkodat

---

## 8. DevOps — konkret implementation

- [ ] `Dockerfile` för backend
- [ ] `docker-compose.yml` — backend + PostgreSQL tillsammans lokalt
- [ ] GitHub Actions workflow: kör tester vid varje push
- [ ] GitHub Actions workflow: bygger Docker-image
- [ ] (Ev.) automatisk deploy vid push till main
- [ ] `.env` / miljövariabler hanteras korrekt (ej committade till git)

---

## 9. Byggordning (faser)

**Fas 1 — Projektgrund**
1. Spring Boot-projekt setup, PostgreSQL i Docker, grundstruktur
2. User-entitet, databaskoppling
3. Registrering & inloggning (BCrypt, JWT, Spring Security)

**Fas 2 — Kärnresurser**
4. Session/pass-entitet + admin-CRUD
5. Roller & endpoint-skydd (USER vs ADMIN)
6. Validering & felhantering

**Fas 3 — Bokningslogik**
7. Boka/avboka + kapacitetskontroll (race condition-säkert)
8. Sökning & paginering
9. Rate limiting på login

**Fas 4 — Frontend (React, lärs in under tiden)**
10. React-grunder (komponenter, state, props, hooks) — förklaras löpande
11. Login/register-UI, spara JWT, skyddade routes
12. Lista pass, boka-knapp, "mina bokningar"-vy, admin-vy

**Fas 5 — DevOps & Deployment**
13. Docker & docker-compose för hela stacken
14. GitHub Actions CI-pipeline (test → build)
15. Deployment till molntjänst

---

## 10. Statustracker — vad är klart?

> Uppdatera denna sektion löpande. Skriv datum + kort kommentar vid varje avklarad punkt.

### Fas 1 — Projektgrund
- [ ] Projektsetup (Spring Boot + Docker + PostgreSQL)
- [ ] User-entitet & databas
- [ ] Registrering & inloggning

### Fas 2 — Kärnresurser
- [ ] Session-entitet & admin-CRUD
- [ ] Roller & skydd
- [ ] Validering & felhantering

### Fas 3 — Bokningslogik
- [ ] Bokningslogik & kapacitetskontroll
- [ ] Sökning & paginering
- [ ] Rate limiting

### Fas 4 — Frontend
- [ ] React-grunder inlärda
- [ ] Login/register-UI
- [ ] Boka-UI

### Fas 5 — DevOps & Deployment
- [ ] Docker & compose
- [ ] GitHub Actions
- [ ] Deployment live

---

## 11. Repo-namn — BESTÄMT: `gymflow`

Repo skapat. Kvarvarande namn-relaterat att göra:
- [ ] Skriv en bra README.md med projektbeskrivning, tech-stack, setup-instruktioner
- [ ] (Ev.) lägg till en kort tagline i README, t.ex. "GymFlow — bokningssystem för gympass, byggt med Java, Spring Boot, React & PostgreSQL"

Andra namn som övervägdes: fitbook, sessionly, trainly, spring-gym-booking.

---

## 12. Öppna beslut / att bestämma senare

- [x] ~~Exakt namn på GitHub-repo~~ → **gymflow**
- [ ] Vilken molntjänst för deployment (Railway/Render/Fly.io)
- [ ] UUID vs auto-increment bigint för primary keys
- [ ] Bucket4j eller egen enkel rate limiter-lösning

---

## 13. Loggbok (kort anteckning per session)

> Fyll på här efter varje kodsession, så vi snabbt kan repetera var vi var.

- **2026-09-08** — Projektdokument skapat. Repo `gymflow` skapat på GitHub. CLAUDE.md skapat för Claude Code-arbetsflöde. Redo att börja Fas 1.
