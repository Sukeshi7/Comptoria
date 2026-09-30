# Comptoria

> Application de gestion de notes de frais

[![CI](https://github.com/Sukeshi7/Comptoria/actions/workflows/ci.yml/badge.svg)](https://github.com/Sukeshi7/Comptoria/actions)
[![Next.Js](https://img.shields.io/badge/Next-16.3.5-white?logo=nextdotjs)](https://nextjs.org/)
[![TailwindCss](https://img.shields.io/badge/Tailwind-4.3.3-blue?logo=tailwindcss)](https://tailwindcss.com/)
[![Vitest](https://img.shields.io/badge/Vitest-5.0.1-green?logo=vitest)](https://vitest.dev/)
[![codecov](https://codecov.io/github/Sukeshi7/Comptoria/graph/badge.svg?token=A913WG9KW0)](https://codecov.io/github/Sukeshi7/Comptoria)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

---
**Production API**: tbd\
**OpenAPI Spec**: tbd

---

## Table of Contents

- [Overview](#overview)
- [Screenshots](#screenshots)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [Testing](#testing)
- [Deployment](#deployment)
- [Database Migrations](#database-migrations)
- [Project Structure](#project-structure)
- [Architecture Decision Records](#architecture-decision-records)
- [Roadmap](#roadmap)

## Overview

Comptoria est une application de gestion de notes de frais pensée pour les freelances et indépendants. L'idée : saisir une dépense professionnelle en quelques secondes (montant, catégorie, date, photo du reçu), et récupérer en fin de mois un récapitulatif prêt à transmettre à sa comptabilité.

Le scope est volontairement limité — pas de gestion multi-entreprise, pas de facturation client, pas de multi-devise. L'objectif de ce projet est de faire peu de choses, mais de bout en bout : authentification, persistance des données, tests, intégration continue et déploiement, plutôt que d'empiler des fonctionnalités superficielles.

**Démo en ligne** : pas encore disponible — voir [Deployment](#deployment).

## Screenshots

À venir, une fois l'interface de gestion des dépenses en place.

## Architecture

Comptoria est un monolithe Next.js : l'interface et l'API vivent dans la même application, déployée comme une seule unité.

```
┌────────────┐        ┌──────────────────────────┐          ┌─────────────┐         ┌──────────────┐
│  Client    │ ─────▶ │  Next.js App Router      │ ─────▶  │   Prisma    │ ─────▶ │  PostgreSQL  │
│ (React UI) │ ◀───── │  UI + Route Handlers     │ ◀─────  │   Client    │ ◀───── │              │
└────────────┘        └────────────┬─────────────┘          └─────────────┘         └──────────────┘
                                   │
                                   ▼
                            ┌─────────────┐
                            │  NextAuth   │
                            │  (session)  │
                            └─────────────┘
```

Le client React communique avec les Route Handlers de l'App Router, qui passent par Prisma pour lire/écrire en base PostgreSQL. L'authentification est gérée par NextAuth via un middleware qui protège les routes nécessitant une session active.

Pourquoi un monolithe plutôt que des microservices : à l'échelle d'un projet solo avec un seul type d'utilisateur et un volume de données modeste, découper en services séparés ajouterait de la complexité (déploiements multiples, communication réseau interne, cohérence des données) sans bénéfice réel. Le détail de ce choix est dans [ADR 0001](docs/adr/0001-choix-stack.md).

## Tech Stack

| Catégorie | Techno | Version |
|---|---|---|
| Framework | Next.js (App Router) | 16.3.5 |
| UI | React | 19.2.8 |
| Styles | Tailwind CSS | 4.3.3 |
| Langage | TypeScript | 5.9.3 |
| ORM | Prisma | 7.10.0 |
| Base de données | PostgreSQL | — |
| Authentification | NextAuth.js | 4.24.15 |
| Tests | Vitest + Testing Library | 5.0.1 |
| CI/CD | GitHub Actions | — |
| Couverture de tests | Codecov | — |
| Déploiement (prévu) | Railway | — |

Le détail des choix (pourquoi Prisma plutôt qu'un autre ORM, pourquoi NextAuth, etc.) est dans les [ADR](#architecture-decision-records) plutôt que dupliqué ici.

## Getting Started

**Prérequis** : Node.js 22+, npm, une instance PostgreSQL (locale ou via Docker).

```bash
git clone https://github.com/Sukeshi7/Comptoria.git
cd Comptoria
npm install
cp .env.example .env      # puis renseigne tes propres valeurs
npx prisma migrate dev
npm run dev
```

L'application est ensuite disponible sur `http://localhost:3000`.

## Environment Variables

| Variable | Description | Exemple |
|---|---|---|
| `DATABASE_URL` | URL de connexion à la base PostgreSQL | `postgresql://user:password@localhost:5432/comptoria` |
| `NEXTAUTH_SECRET` | Clé utilisée par NextAuth pour signer les sessions | générée via `openssl rand -base64 32` |
| `NEXTAUTH_URL` | URL de base de l'application | `http://localhost:3000` |

Ces variables sont listées (sans valeurs réelles) dans `.env.example`, versionné dans le repo — ton `.env` local, lui, ne l'est pas.

## API Reference

Routes prévues pour la gestion des dépenses (implémentation en cours) :

| Méthode | Route | Description |
|---|---|---|
| `POST` | `/api/depenses` | Créer une dépense |
| `GET` | `/api/depenses` | Lister les dépenses de l'utilisateur connecté |
| `GET` | `/api/depenses/:id` | Récupérer une dépense |
| `PATCH` | `/api/depenses/:id` | Modifier une dépense |
| `DELETE` | `/api/depenses/:id` | Supprimer une dépense |
| `GET` | `/api/depenses/export` | Générer le PDF récapitulatif du mois |

Exemple de corps de requête pour `POST /api/depenses` :

```json
{
  "montant": 24.00,
  "categorie": "Repas",
  "date": "2026-09-21",
  "recuUrl": "https://.../recu.jpg"
}
```

## Testing

```bash
npm run test            # lance la suite de tests une fois
npm run test:watch      # mode watch, pratique en dev
npm run test:coverage   # avec rapport de couverture
```

Les tests unitaires (fonctions pures, ex. calculs d'agrégation) vivent à côté du code qu'ils testent, dans des dossiers `__tests__/`. Les tests d'intégration sur les routes API (avec Supertest) arriveront avec l'implémentation du CRUD.

La couverture est mesurée par Vitest (`@vitest/coverage-v8`) et publiée automatiquement sur Codecov à chaque push — voir le badge en haut de ce fichier. Pas de seuil minimum formel fixé pour l'instant, ce sera revu une fois le CRUD couvert (voir [Roadmap](#roadmap)).

## Deployment

Pas encore déployé — prévu une fois le socle applicatif (auth + CRUD) fonctionnel. Le déploiement se déclenchera automatiquement à chaque push sur `main`, avec les mêmes variables d'environnement que `.env.example` configurées côté plateforme.

## Database Migrations

```bash
npx prisma migrate dev      # crée et applique une migration en local
npx prisma migrate deploy   # applique les migrations en production (via le pipeline de déploiement)
```

## Project Structure

```
Comptoria/
├── .github/workflows/      # Pipeline CI (lint, tests, couverture)
├── docs/adr/                # Architecture Decision Records
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── src/
│   ├── app/                 # Pages et Route Handlers (App Router)
│   ├── components/
│   └── lib/                 # Logique métier (calculs, helpers)
│       └── __tests__/       # Tests unitaires
├── vitest.config.mts
├── vitest.setup.ts
└── .env.example
```

## Architecture Decision Records

Les décisions techniques structurantes de ce projet sont documentées sous forme d'ADR (Architecture Decision Record) : un court fichier qui explique le contexte, le choix fait, et ses conséquences — pour que quelqu'un qui lit le repo comprenne le raisonnement, pas juste le résultat.

| ADR | Titre | Statut |
|---|---|---|
| [0001](docs/adr/0001-choix-stack.md) | Choix de la stack (Next.js, Prisma, PostgreSQL) | À rédiger |

## Roadmap

**Fait**
- Repo, CI (lint + tests) et couverture Codecov opérationnels
- Stack posée : Next.js, TypeScript, Tailwind, Prisma, NextAuth, Vitest
- Schéma de données (`User`, `Depense`, `Categorie`)

**En cours**
- Authentification NextAuth

**À venir**
- CRUD complet des dépenses
- Calculs d'agrégation (total par catégorie / mois)
- Export PDF mensuel
- Premier déploiement sur Railway
- Passe accessibilité de base

**Limites connues (assumées à ce stade)**
- Pas de gestion multi-devise
- Pas de tests end-to-end
- Pas de gestion multi-entreprise (un compte = un seul jeu de dépenses)