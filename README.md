AXS. tt
# Storingsmelder

Een klein meldingensysteem voor storingen. Je meldt een storing via een
formulier, je ziet de openstaande meldingen in een lijst, en je sluit ze af als
ze opgelost zijn.

Deze applicatie is het startpunt voor **Cloud computing (B-UCLL-MGN13A)**. Je
werkt er het hele semester mee. De applicatie zelf verandert bijna niet, wat
verandert is hoe en waar ze draait.

---

## Wat er in deze repo zit

```
app.py              de applicatie
requirements.txt    de pakketten die ze nodig heeft
templates/          de HTML van de webpagina
```

## Wat er bewust niet in zit

Geen Dockerfile en geen compose-bestand. Die schrijf je zelf, dat is precies
wat je in dit vak leert.

---

## Wat de applicatie nodig heeft

| | |
|---|---|
| Python | 3.11 of hoger |
| Pakketten | staan in `requirements.txt` |
| Poort | 8000, tenzij je `PORT` anders instelt |

## Instellingen

Alles wordt geregeld met environment variables. Er staat niets vast in de code,
en er is geen configuratiebestand dat je moet aanpassen.

| Variabele | Standaard | Wat het doet |
|---|---|---|
| `DATABASE_URL` | leeg | Leeg betekent opslaan in een SQLite-bestand. Vul je hier een PostgreSQL-adres in, dan gebruikt de applicatie die database |
| `SQLITE_PAD` | `/data/storingen.db` | Waar het SQLite-bestand komt te staan |
| `APP_TITEL` | `Storingsmelder` | De titel bovenaan de pagina |
| `PORT` | `8000` | De poort waarop de applicatie luistert |

### Twee standen

De applicatie draait met of zonder aparte database.

**Zonder `DATABASE_URL`** schrijft ze naar een SQLite-bestand. Dat werkt meteen
en je hebt niets anders nodig. Let op waar dat bestand terechtkomt, want daar
kom je later op terug.

**Met `DATABASE_URL`** praat ze met PostgreSQL. Bijvoorbeeld:

```
DATABASE_URL=postgresql://storing:geheim@db:5432/storingen
```

Je hoeft de tabel niet zelf aan te maken. De applicatie doet dat bij het
opstarten als ze nog niet bestaat.

---

## Controleren of ze draait

Naast de gewone pagina is er een adres dat kort antwoordt of alles in orde is:

```
GET /health
```

Draait alles, dan krijg je `{"status": "ok", ...}` met code 200. Kan de
applicatie haar database niet bereiken, dan krijg je code 503. Dat adres komt
later in het semester nog van pas.

---

## Wat je er dit semester mee doet

| Blok | Wat je doet |
|---|---|
| Week 1 tot 4 | Je zet deze applicatie in een container en laat ze samen met een database draaien |
| Week 5 tot 9 | Je zet diezelfde applicatie op een Kubernetes-cluster en maakt ze bereikbaar in je browser |
| Week 10 tot 11 | Je laat een pipeline het image bouwen en klaarzetten |

Je begint dus nooit opnieuw. Elke opdracht bouwt verder op de vorige.

---

## Deze repo is van jou

Je hebt hem gemaakt vanaf het sjabloon, dus dit is jouw kopie. Werk erin, commit
elke les, en push. Indienen doe je met de link naar een commit. Hoe dat precies
gaat, staat in de opdrachtbundel op Toledo.

## Het stappeplan: Storingsmelder in containers

De app draait in 2 containers: de app zelf (`web`) en een PostgreSQL-database (`db`). Alles start met één commando en de data overleeft een herstart.

### Starten vanaf het begin

nodig: Docker Desktop (met Docker Compose) en Git.
En daarna die de commando's hieronder

```
git clone https://github.com/AxS-UCLL/cloud-computing.git
cd cloud-computing
copy .env.example .env
```

Dan Open je `.env` en vul de drie waarden in, bijvoorbeeld:

```
POSTGRES_USER=storing
POSTGRES_PASSWORD=kies-een-wachtwoord
POSTGRES_DB=storingsmelder
```

dan starten

```
docker compose up -d --build
```

Open daarna http://localhost:8000. Controleer de status met `docker compose ps` (de database staat op `healthy`) en `http://localhost:8000/health` (opslag: PostgreSQL).

Stoppen:

```
docker compose down      # containers weg, data blijft bewaard
docker compose down -v   # containers én volumes weg, data is weg
```

### Wat er in de repo zit

- `Dockerfile`: bouwt de applicatie op `python:3.14-slim`
- `compose.yaml`: app en database samen, met volumes, healthcheck en `depends_on`
- `.dockerignore`: houdt `.git`, `.env` en databasebestanden uit het image
- `.env.example`: de namen van de instellingen, zonder waarden
- `.env` staat niet in de repo, want daar staat het wachtwoord in

### Het image op ghcr.io

Het image staat publiek op de GitHub Container Registry
om aaan de image te geraken doe je het comando dat hier onder staat.

```
docker pull ghcr.io/axs-ucll/storingsmelder:1.0
````