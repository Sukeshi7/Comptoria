# ADR 0001 : Choix de la stack

## Contexte

Comptoria est une application de gestion de notes de frais pour freelances : un utilisateur authentifié, des dépenses liées à cet utilisateur, un calcul d'agrégation, un export PDF. Le volume de données et le nombre de types d'utilisateurs sont volontairement faibles — c'est un projet solo à but de démonstration technique, pas un produit destiné à scaler.

L'enjeu du choix de stack n'était pas la performance à grande échelle, mais : rester sur des outils largement utilisés en entreprise en France (donc reconnaissables en entretien), garder une seule application à déployer plutôt qu'un ensemble de services, et éviter la complexité qui n'apporterait rien à ce périmètre.

## Décision

- **Framework** : Next.js (App Router) — sert à la fois l'interface (React) et l'API (Route Handlers) dans une seule application, sans backend séparé à maintenir.
- **Langage** : TypeScript — typage statique sur l'ensemble du projet, y compris le schéma de données via Prisma.
- **Styles** : Tailwind CSS v4 — classes utilitaires plutôt que CSS modules, pour itérer vite sur l'UI sans context-switch entre fichiers.
- **Base de données** : PostgreSQL — les données de l'application sont relationnelles par nature (un utilisateur a plusieurs dépenses, chaque dépense appartient à une catégorie), avec des besoins d'agrégation (sommes par catégorie/mois) que SQL exprime nativement.
- **ORM** : Prisma — épinglé explicitement sur la version stable **7.10.0**, et non sur le tag `latest` du registre npm, qui pointait au moment de l'installation vers une release candidate de la version 8 (`8.0.0-rc.15`). Une RC embarque des dépendances internes encore instables (nouveau toolchain de build interne), inutile de prendre ce risque sur un projet dont le but est justement de démontrer un usage sérieux des outils.
- **Authentification** : NextAuth.js — solution standard de l'écosystème Next.js, évite d'écrire une gestion de sessions/tokens maison pour un besoin d'authentification classique.
- **Tests** : Vitest — compatible avec l'écosystème Vite déjà présent via Next.js, syntaxe proche de Jest.

## Conséquences

Le choix d'un monolithe Next.js plutôt que d'une architecture en services séparés simplifie le déploiement (une seule application) et l'authentification (pas de communication inter-services à sécuriser), au prix d'un couplage plus fort entre UI et logique serveur — acceptable à cette échelle, à reconsidérer si le projet devait un jour accueillir plusieurs équipes ou services indépendants.

Le pin explicite de Prisma en 7.10.0 signifie qu'une mise à jour vers Prisma 8 devra être une décision consciente une fois cette version stabilisée, pas un `npm update` silencieux.