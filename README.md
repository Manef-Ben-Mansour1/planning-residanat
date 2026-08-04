# Carnet de révision — Résidanat

Application web monopage pour suivre ta progression de révision du concours de Résidanat en médecine : checklist par spécialité, agenda mensuel, statistiques (pages restantes, spécialités acquises), synchronisation multi-appareils.

## Fonctionnalités

- **Suivi des cours** — coche chaque cours, ajoute une date de révision prévue.
- **Agenda mensuel** — planifie tes révisions jour par jour, auto-détection des cours mentionnés.
- **Auth email/mot de passe** — chaque utilisateur a ses propres données.
- **Sync cloud automatique** — chaque changement est sauvegardé dans Supabase (~400 ms débouncé).
- **Multi-appareils** — connecte-toi depuis n'importe quel navigateur, tu retrouves ta progression.

## Stack

- HTML/CSS/JS vanilla dans un seul fichier
- [Supabase](https://supabase.com) pour l'auth + PostgreSQL
- Déployé sur GitHub Pages

## Setup local

1. Clone le repo :
   ```bash
   git clone https://github.com/<toi>/planning-residanat.git
   cd planning-residanat
   ```

2. Crée ta config locale :
   ```bash
   cp config.example.js config.js
   ```
   Puis édite `config.js` avec tes valeurs (voir section Supabase ci-dessous).

3. Ouvre `planning_residanat.html` dans un navigateur — c'est tout, aucun build.

## Configuration Supabase

1. Crée un projet sur [supabase.com](https://supabase.com) (gratuit, sans carte).
2. Menu **SQL Editor** → **New query** → colle et exécute :
   ```sql
   create table notebooks (
     user_id uuid primary key references auth.users on delete cascade,
     data jsonb,
     updated_at timestamptz default now()
   );
   alter table notebooks enable row level security;
   create policy "own_notebook" on notebooks for all
     using (auth.uid() = user_id) with check (auth.uid() = user_id);
   ```
3. **Authentication → Providers → Email** : décoche "Confirm email" si tu veux tester sans lien de confirmation.
4. **Project Settings → API** : copie **Project URL** et la clé **anon public**, colle-les dans `config.js`.

### Sécurité

- La clé `anon public` est **conçue** pour être exposée côté client. La sécurité vient des règles Row Level Security (`auth.uid() = user_id`) : personne ne peut lire les données des autres.
- La clé `service_role` (dans la même page) **ne doit jamais être exposée**. Elle n'est utilisée nulle part dans ce repo.
- `config.js` est ignoré par git (`.gitignore`).

## Déploiement (GitHub Pages)

Le repo contient un workflow `.github/workflows/deploy.yml` qui :

1. Génère `config.js` à partir de deux **GitHub Secrets** (`SUPABASE_URL` et `SUPABASE_ANON_KEY`).
2. Déploie sur GitHub Pages.

### Étapes une seule fois

1. **Settings → Secrets and variables → Actions → New repository secret** :
   - `SUPABASE_URL` = ton Project URL
   - `SUPABASE_ANON_KEY` = ta clé anon public
2. **Settings → Pages → Build and deployment → Source** : sélectionne **GitHub Actions**.
3. Push sur `main` → le workflow se lance → ton site est en ligne à `https://<toi>.github.io/planning-residanat/`.
4. Dans Supabase, ajoute cette URL dans **Authentication → URL Configuration → Site URL** (et Redirect URLs) pour que l'auth accepte les redirections.

## Structure

```
planning_residanat.html   → app complète (HTML + CSS + JS)
config.example.js         → template de config (commit)
config.js                 → tes clés locales (gitignoré)
.github/workflows/        → CI/CD pour GitHub Pages
```
