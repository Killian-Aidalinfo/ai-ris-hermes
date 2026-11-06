# AI_Ris — un agent Hermes pour les projets CESI

AI_Ris est un agent IA construit avec [Hermes Agent](https://hermes-agent.nousresearch.com) (Nous Research). C'est un copilote technique et pédagogique pour les cubes et projets du CESI : développement, systèmes, réseaux, cybersécurité, DevOps et gestion de projet.

Ce dépôt contient de quoi recréer l'agent sur ton propre serveur :

| Fichier | Rôle |
|---|---|
| `SOUL.md` | La personnalité et la mission de l'agent (son « âme »). C'est le fichier à lire et à adapter en premier. |
| `config.yaml` | La configuration Hermes utilisée (modèle, outils, workspace web activé…). |
| `.env.example` | Les variables secrètes à remplir (clés, jetons, mot de passe). |
| `docker-compose.yml` | Lance l'agent dans Docker, avec l'API sur le port 8642 et le workspace web sur le port 9120. |

Testé avec Hermes Agent v0.23.0.

## Prérequis

- Un serveur Linux (une VM Ubuntu suffit) avec Docker et le plugin Compose.
- Un accès à un fournisseur de modèle. La config utilise `openai-codex` (compte ChatGPT) ; tu peux en choisir un autre pendant l'étape 3.

## Installation

### 1. Récupérer le dépôt

```bash
git clone https://github.com/Killian-Aidalinfo/ai-ris-hermes.git
cd ai-ris-hermes
```

### 2. Préparer le dossier de données

Le dossier `data/` contient tout ce que l'agent écrit : conversations, mémoire, clés. Il n'est jamais versionné.

```bash
mkdir -p data
cp SOUL.md config.yaml data/
cp .env.example data/.env
chmod 600 data/.env
```

Génère la clé de l'API et colle-la dans `API_SERVER_KEY` de `data/.env` :

```bash
openssl rand -hex 32
```

### 3. Connecter un modèle

Lance l'assistant de configuration de Hermes et suis les étapes (choix du fournisseur et connexion) :

```bash
docker compose run --rm hermes setup
```

Tu peux changer de modèle plus tard avec `docker compose run --rm hermes model`.

### 4. Démarrer l'agent

```bash
docker compose up -d
docker compose logs -f
```

### 5. Créer le mot de passe du workspace web

Le workspace web est exposé sur `0.0.0.0`, donc Hermes exige une connexion. Choisis un mot de passe, puis calcule son empreinte :

```bash
docker compose exec hermes /opt/hermes/.venv/bin/python -c "import sys; sys.path.insert(0, '/opt/hermes'); from plugins.dashboard_auth.basic import hash_password; print(hash_password(sys.argv[1]))" 'ton-mot-de-passe'
```

Dans `data/.env`, colle le résultat dans `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD_HASH`, et le résultat de `openssl rand -base64 32` dans `HERMES_DASHBOARD_BASIC_AUTH_SECRET`. Redémarre ensuite :

```bash
docker compose restart
```

### 6. Ouvrir le workspace

Va sur `http://<ip-de-ton-serveur>:9120` et connecte-toi avec `admin` et ton mot de passe.

## Discord (facultatif)

Pour discuter avec l'agent sur Discord, crée un bot sur le [portail développeur Discord](https://discord.com/developers/applications), puis remplis `DISCORD_BOT_TOKEN` et `DISCORD_ALLOWED_USERS` (ton ID Discord) dans `data/.env`, et redémarre.

## Sécurité

- Ne publie jamais le dossier `data/` : il contient tes clés, tes jetons de connexion et tes conversations. Le `.gitignore` l'exclut déjà.
- Le workspace passe en HTTP simple : le mot de passe circule en clair. Pour un usage sérieux, mets un reverse proxy HTTPS devant (Caddy, Traefik…) ou accède-y par un tunnel SSH.
- L'agent peut exécuter des commandes sur la machine : installe-le sur une VM dédiée, pas sur ton poste personnel.

## Adapter l'agent

Modifie `data/SOUL.md` pour changer sa personnalité, son domaine ou sa façon de répondre, puis redémarre avec `docker compose restart`. Toute la configuration est documentée dans les commentaires de `config.yaml` et sur [la doc Hermes](https://hermes-agent.nousresearch.com/docs).
