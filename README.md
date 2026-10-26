# rallly-docker-infra

Infrastructure Docker du projet « Dev avec Docker » (M2 DEVFULLSTACK) : reverse proxy **Caddy**, SSO **Authentik** (OIDC) et **Rallly** (fork avec features), le tout en Compose.

> État : stack locale en cours de mise en place. Déploiement VM Énov, CI/CD et features à venir.

## Architecture

```
navigateur ──► Caddy (seul port publié)
                 ├─ auth.localhost   ► authentik-server ─► authentik-db (Postgres)
                 │                     authentik-worker
                 ├─ rallly.localhost ► rallly ─► rallly-db (Postgres), garage (S3)
                 └─ mail.localhost   ► mailpit (SMTP de lab)
```

- Réseau `proxy` : Caddy et les services web. Caddy y porte les alias des noms d'hôte, donc Rallly joint Authentik avec la même URL que le navigateur (nécessaire à l'OIDC).
- Réseau `backend` : interne, sans accès extérieur. Bases de données et stockage ne publient aucun port.

## Démarrage local

Prérequis : Docker et Docker Compose, un utilisateur autorisé à utiliser Docker.

```bash
git clone https://github.com/nicolascarbo/rallly-docker-infra.git
cd rallly-docker-infra
cp .env.example .env
```

Remplace ensuite les valeurs factices de `.env` (voir ci-dessous), puis :

```bash
docker compose up -d
```

| Service | URL | Identifiants |
|---|---|---|
| Authentik | http://auth.localhost | `akadmin` / `AUTHENTIK_BOOTSTRAP_PASSWORD` |
| Rallly | http://rallly.localhost | via SSO (après configuration OIDC) |
| Mailpit | http://mail.localhost | aucun |

`*.localhost` est résolu en `127.0.0.1` par les navigateurs, sans toucher à `/etc/hosts`.

## Configuration du `.env`

`.env` est ignoré par Git. Seul `.env.example` (valeurs factices) est commité.

Les variables de la première section (hôtes, port, schéma) peuvent rester telles quelles en local. Les secrets sont à générer :

| Variable | Rôle | Comment la remplir |
|---|---|---|
| `AUTHENTIK_SECRET_KEY` | Clé de chiffrement d'Authentik, à ne plus changer ensuite | `openssl rand -base64 60` |
| `PG_PASS` | Mot de passe de la base d'Authentik | `openssl rand -hex 24` |
| `AUTHENTIK_BOOTSTRAP_EMAIL` | Email du compte admin `akadmin` | ton email |
| `AUTHENTIK_BOOTSTRAP_PASSWORD` | Mot de passe du compte admin | au choix |
| `SECRET_PASSWORD` | Clé de session de Rallly (32 caractères minimum) | `openssl rand -hex 24` |
| `SUPPORT_EMAIL` | Adresse de contact affichée par Rallly | au choix |
| `POSTGRES_PASSWORD` | Mot de passe de la base de Rallly | `openssl rand -hex 24` |
| `GARAGE_RPC_SECRET` | Secret interne de Garage (64 caractères hexadécimaux) | `openssl rand -hex 32` |
| `S3_ACCESS_KEY_ID` | Clé d'accès S3, format imposé : `GK` + 24 caractères hexadécimaux | `echo "GK$(openssl rand -hex 12)"` |
| `S3_SECRET_ACCESS_KEY` | Secret S3 (64 caractères hexadécimaux) | `openssl rand -hex 32` |
| `OIDC_*` | SSO Rallly, à remplir après avoir créé le provider dans Authentik | laisser vide au début |

## Reste à faire

- [ ] Configurer le SSO Authentik → Rallly, les groupes/policies et l'invitation externe
- [ ] Durcissement et scan d'images
- [ ] CI/CD (GitHub Actions → GHCR)
- [ ] Fork de Rallly et features (plafond de participants, expiration, liste d'attente)
- [ ] Déploiement VM Énov et écarts local/VM
