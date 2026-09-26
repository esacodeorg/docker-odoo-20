Odoo 20
=========

**docker-odoo-20** est une version dockerisée d'Odoo 20 Community.

**docker-odoo-20** is a dockerized Odoo 20 Community.

[Français](#français) · [English](#english)

> **Odoo 20 est en avant-première.** Les paquets sont les constructions
> quotidiennes officielles d'Odoo ; la version stable n'est pas encore sortie.
> Ce dépôt sert à découvrir la 20, préparer une migration et tester des
> modules — pas à faire tourner une production.

> **Odoo 20 is an early preview.** The packages are Odoo's own official nightly
> builds; the stable release is not out yet. This repository is for exploring
> 20, preparing a migration and testing modules — not for running production.

---

# Français

Aucune installation en dur : pas de dépendances système à démêler, pas de
wkhtmltopdf à faire fonctionner, pas de version de Python à négocier. Deux
commandes, et vous avez Odoo 20. Quand vous n'en voulez plus, vous supprimez —
il ne reste rien sur votre machine.

## Suivi des versions

Ce dépôt suit le calendrier d'Odoo, et se simplifiera au fur et à mesure.

| État | Ce que fait le dépôt |
|---|---|
| **Aujourd'hui** — image `odoo:20` absente de Docker Hub | il la construit lui-même, à partir de la recette officielle d'Odoo |
| **Entre-temps** | le paquet est mis à jour ici quand Odoo publie une version notable |
| **À la sortie officielle** | `image-odoo-20/` disparaît, le Dockerfile repasse sur l'image publiée, et le dépôt redevient identique à `docker-odoo-14` … `19` |

Le dossier supplémentaire est une béquille assumée et temporaire. Rien de ce
que vous mettez dans `odoo/addons/` ne bougera quand elle sera retirée.

## Étape 0 — Installer Docker

Si Docker est déjà installé, passez à l'étape 1.

**Windows et macOS** : installez [Docker Desktop](https://www.docker.com/products/docker-desktop/),
puis lancez-le. L'icône baleine doit être présente dans la barre des tâches
avant d'aller plus loin.

**Linux (Ubuntu, Debian)** :

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Déconnectez-vous et reconnectez-vous pour que l'appartenance au groupe prenne
effet, sinon chaque commande réclamera `sudo`.

**Vérifiez** que tout répond :

```bash
docker --version
docker compose version
```

Deux numéros de version s'affichent : vous êtes prêt. Si la seconde commande
échoue, votre Docker est trop ancien — mettez-le à jour plutôt que d'utiliser
l'ancien `docker-compose` avec un tiret.

## Étape 1 — Récupérer le dépôt

```bash
git clone https://github.com/esacodeorg/docker-odoo-20.git
cd docker-odoo-20
```

Une seule fois.

## Étape 2 — Construire l'image Odoo 20

C'est l'étape qui n'existe pas dans nos autres dépôts : Odoo n'a pas encore
publié l'image `odoo:20` sur Docker Hub, on la fabrique donc à partir de sa
recette officielle.

```bash
cd image-odoo-20
docker build -t odoo:20 .
cd ..
```

**Comptez dix à quinze minutes la première fois**, et environ 1,5 Go
téléchargé : Ubuntu, wkhtmltopdf, le client PostgreSQL et le paquet Odoo.
C'est long une fois, puis plus jamais.

Vérifiez que l'image existe :

```bash
docker images | grep odoo
```

## Étape 3 — Démarrer

```bash
docker compose up -d --build
```

Cette commande construit notre couche — quelques dizaines de secondes — puis
démarre Odoo et PostgreSQL. Vérifiez que les deux tournent :

```bash
docker compose ps
```

## Étape 4 — Ouvrir Odoo

Rendez-vous sur **http://localhost:8069**.

Au premier lancement, Odoo demande de créer une base de données. Le mot de
passe maître est celui du fichier `odoo/config/odoo.conf`. Choisissez un nom de
base, cochez les données de démonstration si vous voulez du contenu pour
explorer, et validez. La création prend une à deux minutes.

## Étape 5 — Ajouter vos modules

Déposez-les dans `odoo/addons/` — ils sont montés dans le conteneur sur
`/mnt/extra-addons`.

```bash
docker compose restart web
```

Puis, dans Odoo : activez le mode développeur, allez dans **Applications**,
cliquez sur **Mettre à jour la liste des applications**, et installez le vôtre.

C'est là que la migration devient concrète : ce qui refuse de se charger vous
dit précisément ce que la nouvelle version a changé.

## Au quotidien

```bash
docker compose stop          # arrêter
docker compose start         # redémarrer
docker compose logs -f web   # suivre les journaux
```

**Repartir de zéro**, bases de données comprises :

```bash
docker compose down -v
```

`-v` supprime les volumes, donc **toutes vos bases et vos fichiers joints**.
Sans `-v`, ils survivent et vous les retrouvez au prochain démarrage.

**Reconstruire sur un paquet Odoo plus récent.** Le paquet change chaque nuit.
Relevez la date et l'empreinte publiées par Odoo, reportez-les dans
`image-odoo-20/Dockerfile`, puis reconstruisez l'image :

```bash
curl -s https://raw.githubusercontent.com/odoo/docker/master/20.0/Dockerfile \
  | grep -E "ODOO_RELEASE|ODOO_SHA"
```

N'inventez pas l'empreinte : c'est elle qui garantit que le paquet installé est
bien celui d'Odoo.

## Si ça coince

**Le port 8069 est déjà pris.** Un autre Odoo tourne sur votre machine.
Changez le port publié dans `docker-compose.yml` : `"8169:8069"`, puis
`docker compose up -d`.

**Odoo ne répond pas alors que le conteneur tourne.** Regardez les journaux :

```bash
docker compose logs web | tail -30
```

Si vous lisez `HTTP service running on 127.0.0.1:8069`, c'est le piège de la
version — voir la section suivante.

**La construction de l'image échoue au téléchargement.** Le paquet nightly a
sans doute été remplacé. Relevez la nouvelle date et la nouvelle empreinte
(commande ci-dessus) et réessayez.

**`docker compose` n'existe pas.** Votre Docker est antérieur à la version 2.
Mettez-le à jour ; les commandes de ce dépôt supposent la forme moderne, sans
tiret.

## Un piège d'Odoo 20 à connaître

Odoo 20 **n'écoute plus sur toutes les interfaces par défaut**. Dans
`tools/config.py`, la valeur par défaut de `http_interface` passe de `''`
— toutes les interfaces — à `127.0.0.1`, et Odoo la force même lorsque le champ
est laissé vide.

Conséquence dans un conteneur : tout démarre, les journaux sont impeccables, et
le port publié ne répond pas.

```
HTTP service running on 127.0.0.1:8069
```

Le fichier `odoo/config/odoo.conf` de ce dépôt contient donc un
`http_interface = 0.0.0.0` explicite. En 18 et 19, il était inutile.

## Ce qu'il y a dans le dépôt

| Dossier | Contenu |
|---|---|
| `image-odoo-20/` | la recette de **l'image Docker**, reprise telle quelle chez Odoo. **Aucun module dedans** |
| `odoo/` | notre Dockerfile (`FROM odoo:20`) et la configuration |
| `odoo/addons/` | **vos modules** |
| `docker-compose.yml` | Odoo (8069) et PostgreSQL 17 |

Les modules de base d'Odoo — `base`, `sale`, `stock` et les autres — sont déjà
dans l'image. Vous n'avez jamais à les copier.

### Ports

| | |
|---|---|
| Odoo | 8069 |
| Long polling | 8072 |

Le 8072 n'est ouvert qu'en mode multi-processus, c'est-à-dire avec
`workers > 0` dans la configuration. En mode threadé — le défaut — tout passe
par 8069 et le 8072 ne répond pas : c'est normal.

---

# English

Nothing installed on the host: no system dependencies to untangle, no
wkhtmltopdf to get working, no Python version to negotiate. Two commands and
you have Odoo 20. When you are done, you delete it — nothing is left behind.

## Version tracking

This repository follows Odoo's calendar, and will get simpler over time.

| State | What the repository does |
|---|---|
| **Today** — no `odoo:20` image on Docker Hub | it builds one, from Odoo's own official recipe |
| **Meanwhile** | the package is bumped here whenever Odoo publishes a notable build |
| **At the official release** | `image-odoo-20/` goes away, the Dockerfile switches back to the published image, and the repository becomes identical to `docker-odoo-14` … `19` |

The extra folder is a deliberate, temporary crutch. Nothing you put in
`odoo/addons/` will move when it is removed.

## Step 0 — Install Docker

If Docker is already installed, go to step 1.

**Windows and macOS**: install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
and start it. The whale icon must be in the taskbar before you go further.

**Linux (Ubuntu, Debian)**:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Log out and back in so the group membership takes effect, otherwise every
command will ask for `sudo`.

**Check** that both answer:

```bash
docker --version
docker compose version
```

Two version numbers: you are ready. If the second command fails, your Docker is
too old — update it rather than falling back on the hyphenated
`docker-compose`.

## Step 1 — Get the repository

```bash
git clone https://github.com/esacodeorg/docker-odoo-20.git
cd docker-odoo-20
```

Once only.

## Step 2 — Build the Odoo 20 image

This is the step that does not exist in our other repositories: Odoo has not
published the `odoo:20` image on Docker Hub yet, so we build it from their
official recipe.

```bash
cd image-odoo-20
docker build -t odoo:20 .
cd ..
```

**Allow ten to fifteen minutes the first time**, and about 1.5 GB of downloads:
Ubuntu, wkhtmltopdf, the PostgreSQL client and the Odoo package. Long once,
then never again.

Check that the image exists:

```bash
docker images | grep odoo
```

## Step 3 — Start

```bash
docker compose up -d --build
```

This builds our layer — a few dozen seconds — then starts Odoo and PostgreSQL.
Check that both are up:

```bash
docker compose ps
```

## Step 4 — Open Odoo

Go to **http://localhost:8069**.

On first launch, Odoo asks you to create a database. The master password is the
one in `odoo/config/odoo.conf`. Pick a database name, tick the demo data if you
want content to explore, and confirm. Creation takes a minute or two.

## Step 5 — Add your modules

Drop them in `odoo/addons/` — they are mounted in the container at
`/mnt/extra-addons`.

```bash
docker compose restart web
```

Then, in Odoo: turn on developer mode, go to **Apps**, click **Update Apps
List**, and install yours.

This is where a migration stops being abstract: whatever refuses to load tells
you exactly what the new version changed.

## Day to day

```bash
docker compose stop          # stop
docker compose start         # start again
docker compose logs -f web   # follow the logs
```

**Start over**, databases included:

```bash
docker compose down -v
```

`-v` removes the volumes, so **all your databases and attachments**. Without
`-v` they survive and are still there on the next start.

**Rebuild on a newer Odoo package.** The package changes every night. Read the
date and the checksum Odoo publishes, put them in `image-odoo-20/Dockerfile`,
then rebuild the image:

```bash
curl -s https://raw.githubusercontent.com/odoo/docker/master/20.0/Dockerfile \
  | grep -E "ODOO_RELEASE|ODOO_SHA"
```

Do not invent the checksum: it is what guarantees the installed package really
is Odoo's.

## Troubleshooting

**Port 8069 is already in use.** Another Odoo is running on your machine.
Change the published port in `docker-compose.yml` to `"8169:8069"`, then
`docker compose up -d`.

**Odoo does not answer although the container is running.** Check the logs:

```bash
docker compose logs web | tail -30
```

If you read `HTTP service running on 127.0.0.1:8069`, that is the version trap
— see the next section.

**The image build fails while downloading.** The nightly package has most
likely been replaced. Read the new date and checksum (command above) and try
again.

**`docker compose` does not exist.** Your Docker predates version 2. Update it;
the commands in this repository assume the modern, hyphen-free form.

## An Odoo 20 trap worth knowing

Odoo 20 **no longer listens on every interface by default**. In
`tools/config.py`, the default value of `http_interface` moves from `''` — all
interfaces — to `127.0.0.1`, and Odoo forces it even when the field is left
empty.

Inside a container that means: everything starts, the logs look perfect, and
the published port does not answer.

```
HTTP service running on 127.0.0.1:8069
```

This repository's `odoo/config/odoo.conf` therefore carries an explicit
`http_interface = 0.0.0.0`. In 18 and 19 it was pointless.

## What is in the repository

| Folder | Contents |
|---|---|
| `image-odoo-20/` | the recipe for **the Docker image**, taken as-is from Odoo. **No modules in there** |
| `odoo/` | our Dockerfile (`FROM odoo:20`) and the configuration |
| `odoo/addons/` | **your modules** |
| `docker-compose.yml` | Odoo (8069) and PostgreSQL 17 |

Odoo's own modules — `base`, `sale`, `stock` and the rest — are already in the
image. You never have to copy them.

### Ports

| | |
|---|---|
| Odoo | 8069 |
| Long polling | 8072 |

8072 only opens in multi-process mode, that is with `workers > 0` in the
configuration. In threaded mode — the default — everything goes through 8069
and 8072 stays silent: that is expected.

---

## La famille / The family

| | | |
|---|---|---|
| [docker-odoo-14](https://github.com/esacodeorg/docker-odoo-14) | [docker-odoo-15](https://github.com/esacodeorg/docker-odoo-15) | [docker-odoo-16](https://github.com/esacodeorg/docker-odoo-16) |
| [docker-odoo-17](https://github.com/esacodeorg/docker-odoo-17) | [docker-odoo-18](https://github.com/esacodeorg/docker-odoo-18) | [docker-odoo-19](https://github.com/esacodeorg/docker-odoo-19) |
| [docker-odoo-20](https://github.com/esacodeorg/docker-odoo-20) | | |

## Crédits / Credits

Maintenu par **esacode — solutions numériques**
<https://www.esacode.org/erp-services/>

Contributions : <https://github.com/esaCodeBJ>

Les fichiers du dossier `image-odoo-20/` appartiennent à Odoo S.A. et sont
repris tels quels depuis <https://github.com/odoo/docker>.

The files in `image-odoo-20/` belong to Odoo S.A. and are taken as-is from
<https://github.com/odoo/docker>.
