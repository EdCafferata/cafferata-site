# Cafferata Apps — website

🔒 Laatste security check: 2026-08-19 23:10 CEST

Statische website voor **cafferata.info**: landingspagina met alle apps (live in
de App Store én in ontwikkeling), een zakelijk deel voor **The IT Crowd** en een
privédeel voor vakanties en losse projectjes.

🟢 **Live sinds 12 juni 2026** op https://cafferata.info (geldig SSL).

🌐 Geen build-stap, geen dependencies — statische HTML-pagina's en één stylesheet.
Werkt volledig offline en op elke webserver.

## Structuur

| Bestand | Inhoud |
|---|---|
| `index.html` | Landingspagina — apps live + onderweg, doorklik naar beide werelden |
| `itcrowd.html` | Zakelijk — diensten, "Werken bij The IT Crowd" |
| `prive.html` | Privé — vakanties, MeshCore, losse projectjes |
| `costa-brava.html` | Reisverhaal Costa Brava dag voor dag |
| `costa-brava-fotos.html` | Volledig fotoalbum Costa Brava (136 foto's + video, lightbox) |
| `assets/style.css` | Huisstijl — Apple/Liquid Glass, licht thema |

## Lokaal bekijken

```bash
cd cafferata-site
python3 -m http.server 8765
# → http://localhost:8765
```

## Deployen op cafferata.info (NAS)

Deze site staat op de **root** `https://cafferata.info/`. De oude WordPress-site
(Cafferata's Piper Rising) is verhuisd naar `https://piper.cafferata.info` en
draait daar zelfstandig door — niet aankomen vanuit deze repo.

**Deploy-commando:**
```bash
rsync -av --exclude '.git' --exclude '.DS_Store' --exclude 'README.md' \
  /Volumes/Backup-Ed/AI/cafferata-site/ /Volumes/web/
```
(`/Volumes/web` is de SMB-mount van de NAS-webroot `/volume1/web`.)

De `canonical`/Open Graph-tags, `robots.txt` en `sitemap.xml` staan ingesteld
op de root (`https://cafferata.info/`). `.htaccess` in de webroot regelt caching
(HTML no-cache, assets 1 dag) en 301-redirects van oude WordPress-URL's naar
`piper.cafferata.info`.

⚠️ Bij elke wijziging in `style.css`: bump de `?v=YYYYMMDD`-query op alle
`<link rel="stylesheet">`-tags, anders blijft de asset-cache (1 dag) de oude
versie serveren.

## Huisstijl — Apple / Liquid Glass

Licht Apple-thema (achtergrond `#f5f5f7`, tekst `#1d1d1f`, knopblauw `#0071e3`)
met SF-typografie via `-apple-system`. Alle panelen (nav, kaarten, knoppen)
zijn **Liquid Glass**: `backdrop-filter: blur(26px) saturate(180%)`, speculaire
randen en sheen, met daaronder een zacht bewegende kleurgloed (aurora-animatie,
respecteert `prefers-reduced-motion`). Browsers zonder `backdrop-filter` krijgen
automatisch een dekkende fallback via `@supports`.

## Apps bijwerken

Nieuwe app live? Voeg een `<article class="card">` toe in `index.html` onder
`#live` (of `#onderweg`) — kopieer een bestaande kaart en pas icoon
(`icon-teal`/`icon-blue`/…), badge (`badge-live`/`badge-soon`/`badge-dev`)
en links aan.
