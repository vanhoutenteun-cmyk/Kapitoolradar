# Fase 1: Cloudflare Pages + Cloudflare Access (besloten bèta)

Doel: kapitoolradar.nl alleen zichtbaar voor uitgenodigde e-mailadressen, met inlog via een e-mailcode.
De workflow (`.github/workflows/update.yml`) is al voorbereid. Hieronder staan de stappen die in het
Cloudflare-dashboard en in GitHub moeten gebeuren. Menunamen in het dashboard kunnen afwijken van wat hier staat.

Zolang stap 6 niet gedaan is, blijft GitHub Pages gewoon deployen en blijft de huidige site werken.

## 1. Domein naar Cloudflare
1. Maak een Cloudflare-account en voeg `kapitoolradar.nl` toe (Free-plan).
2. Controleer de geïmporteerde DNS-records. Neem vooral MX/TXT-records over als je mail op het domein gebruikt.
3. Zet bij je registrar de nameservers op de twee die Cloudflare opgeeft. Dat kan even duren.

Een apex-domein (zonder www) als custom domain op Pages vereist volgens mij dat het domein op Cloudflare staat. Daarom deze stap.

## 2. API-token en account-ID
1. My Profile > API Tokens > Create Token > Custom token.
2. Permissie: **Account > Cloudflare Pages > Edit**. Beperk het token tot je eigen account.
3. Het account-ID staat in het dashboard (o.a. op de overzichtspagina van het domein, rechterkolom).

## 3. Cloudflare Access instellen (vóór de eerste deploy)
1. Open Zero Trust en kies een teamnaam en het Free-plan (gratis tot 50 gebruikers).
2. Settings > Authentication > Login methods: voeg **One-time PIN** toe.
3. Access > Applications > Add an application > **Self-hosted**. Voeg als hostnamen toe:
   - `kapitoolradar.nl`
   - `www.kapitoolradar.nl`
   - `kapitoolradar.pages.dev`
   - `*.kapitoolradar.pages.dev` (preview-deploys)
4. Policy: Action **Allow**, Include **Emails** met de bèta-testers (jezelf eerst).
   Wie niet op de lijst staat, krijgt geen code.

Zonder de `pages.dev`-hostnamen is de data via `kapitoolradar.pages.dev` publiek bereikbaar.

## 4. Secrets in GitHub
Repo > Settings > Secrets and variables > Actions > New repository secret:
- `CLOUDFLARE_API_TOKEN`
- `CLOUDFLARE_ACCOUNT_ID`

Start daarna de workflow handmatig (Actions > Data bijwerken > Run workflow). De eerste run maakt het
Pages-project `kapitoolradar` aan als het nog niet bestaat en deployt naar `kapitoolradar.pages.dev`.

## 5. Eigen domein koppelen
Workers & Pages > kapitoolradar > Custom domains: voeg `kapitoolradar.nl` en `www.kapitoolradar.nl` toe.
Cloudflare vervangt dan de oude A-records (GitHub, 185.199.108-111.153) en de www-CNAME.

## 6. GitHub Pages uitzetten
1. Repo > Settings > Secrets and variables > Actions > **Variables**: maak `HOSTING` met waarde `cloudflare`.
   De workflow deployt dan niet meer naar GitHub Pages.
2. Repo > Settings > Pages: verwijder het custom domain en zet Pages uit, zodat de site niet via
   `github.io` publiek blijft.

## 7. Testen
- Open kapitoolradar.nl in een privévenster: je moet het inlogscherm van Cloudflare zien.
- Log in met een toegestaan adres en controleer of data.json laadt (geen melding "Voorbeelddata").
- Probeer ook `https://kapitoolradar.pages.dev/data.json` in een privévenster: dat moet ook om inlog vragen.

## Caches
`profielen.json` en `wetten.json` werden opgehaald van de publieke site (PREV_URL). Achter Access werkt dat
niet meer, dus de workflow bewaart ze nu met de GitHub Actions-cache (map `cache/`, variabele `CACHE_DIR`).
PREV_URL blijft alleen als terugval voor de allereerste run.
