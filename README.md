# tg4 — monetised fitness content Telegram bot

A Telegram bot that distributes fitness content behind a paywall and accepts payment directly inside the chat. Content is managed through an admin section built into the bot itself, so publishing new material requires no redeploy and no separate admin panel.

Built for the 100 Ideas for Belarus competition on 15 December 2025.

## Features

- Paid access to a content library using Telegram Stars, the in-chat payment currency
- Content types: workouts, videos, photos and nutrition material
- User preferences stored per account
- Admin section inside the bot, protected by Telegram user IDs
- Publish and edit content without touching the code
- PostgreSQL for durable storage, Redis for state
- SQL migrations applied in order, with an index audit script
- FastAPI service exposing a webhook alongside the bot

## Tech stack

| Layer | Technology |
| --- | --- |
| Language | Python 3 |
| Bot framework | aiogram 3 |
| HTTP layer | FastAPI with Uvicorn |
| Database | PostgreSQL via asyncpg |
| Migrations | Plain SQL, applied in order |
| Session storage | Redis, with in-memory fallback |
| Payments | Telegram Stars |
| Configuration | python-dotenv |
| Hosting | Vercel |

## Getting started

### Requirements

- Python 3.11 or newer
- PostgreSQL, local or managed
- A Telegram bot token from [@BotFather](https://t.me/BotFather)

### Environment variables

| Variable | Required | Description |
| --- | --- | --- |
| `BOT_TOKEN` | yes | Token issued by BotFather |
| `ADMIN_IDS` | yes | Comma-separated Telegram user IDs allowed into the admin section |
| `DATABASE_URL` | yes | PostgreSQL connection string |
| `PAYMENT_PROVIDER_TOKEN` | yes | Token for Telegram Stars payments |
| `REDIS_URL` | no | Redis connection string; the bot falls back to in-memory storage |
| `WEBHOOK_URL` | webhook mode | Public HTTPS URL used to receive updates |
| `UPLOADS_DIR` | no | Directory for uploaded media files |

Create a `.env` file in the project root:

```
BOT_TOKEN=123456:ABCDEF...
ADMIN_IDS=111111111,222222222
DATABASE_URL=postgresql://user:password@host/db?sslmode=require
PAYMENT_PROVIDER_TOKEN=...
REDIS_URL=redis://host:6379
```

### Installation

```bash
git clone https://github.com/glcskl/tg4.git
cd tg4
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Apply the migrations in order, then verify the indexes:

```bash
for f in migrations/*.sql; do psql "$DATABASE_URL" -f "$f"; done
python check_indexes.py
```

### Running

```bash
python bot.py
```

The `Procfile` runs `python bot.py`, which is what the hosting platform executes.

## Project structure

```
bot.py             entry point, dispatcher and webhook setup
config.py          environment configuration
database.py        PostgreSQL access layer
handlers.py        message and callback handlers
keyboards.py       inline keyboard layouts
check_indexes.py   index audit for the database
api/index.py       FastAPI application
migrations/        ordered SQL migrations
PERFORMANCE_AUDIT.md  database performance notes
```

## Database

Migrations are plain SQL and are applied in filename order, which keeps the schema reviewable and avoids an extra dependency:

| File | Purpose |
| --- | --- |
| `001_initial_schema.sql` | base schema |
| `002_add_user_prefs.sql` | per-user preferences |
| `003_add_missing_categories.sql` | content categories |
| `004_add_optimization_indexes.sql` | query optimisation indexes |

## Deployment

Deployed on Vercel. Set every variable from the table above in the project settings, apply the migrations once against the production database, and point the Telegram webhook at the deployed endpoint.

## Notes

This project is personal. Payments use Telegram Stars, so no card data ever reaches this codebase.