# Cafferata Apps — website

🔒 Laatste security check: 2026-07-15 22:45 CEST

Statische website voor **cafferata.info**: landingspagina met alle apps (live in
de App Store én in ontwikkeling), een zakelijk deel voor **The IT Crowd** en een
privédeel voor vakanties en losse projectjes.

🌐 Geen build-stap, geen dependencies — drie HTML-pagina's en één stylesheet.
Werkt volledig offline en op elke webserver.

## Structuur

| Bestand | Inhoud |
|---|---|
| `index.html` | Landingspagina — apps live + onderweg, doorklik naar beide werelden |
| `itcrowd.html` | Zakelijk — diensten, team, Werken bij The IT Crowd |
| `prive.html` | Privé — vakantie (Costa Brava 2026), MeshCore, experimenten |
| `assets/style.css` | Huisstijl — donker thema in de stijl van Dicom Player |

## Lokaal bekijken

```bash
cd cafferata-site
python3 -m http.server 8765
# → http://localhost:8765
```

## Deployen op cafferata.info (NAS)

**Besluit (12 juni 2026):** deze site komt op de **root** `https://cafferata.info/`.
De WordPress-site over **Cafferata's Piper Rising** die daar nu draait, verhuist
naar `https://piper.cafferata.info` — de privépagina linkt daar al naartoe.

Volgorde (eerst WordPress verhuizen, dan deze site op de root):

1. **DNS:** voeg een A-record toe: `piper.cafferata.info` → 82.172.143.72
   (zelfde IP als de root).
2. **Webserver:** maak in nginx (of DSM → Web Station → virtual host) een
   server-block voor `piper.cafferata.info` dat naar de bestaande
   WordPress-installatie wijst, en vraag er een Let's Encrypt-certificaat bij aan.
3. **WordPress:** pas het site-adres aan naar `https://piper.cafferata.info`
   (Instellingen → Algemeen, of in `wp-config.php`:
   `define('WP_HOME','https://piper.cafferata.info');` +
   `define('WP_SITEURL','https://piper.cafferata.info');`).
   Controleer dat de site op het subdomein werkt vóór stap 4.
4. **Deze site op de root:** kopieer de bestanden naar de webroot van het
   `cafferata.info` server-block:
   ```bash
   rsync -av --delete --exclude '.git' ./ admin@NAS-IP:/volume1/web/cafferata/
   ```
5. Optioneel: zet in het root-block een redirect van oude WordPress-URL's
   (bijv. `/wp-content/...`, `/?p=...`) naar `https://piper.cafferata.info$request_uri`
   zodat oude links en zoekresultaten blijven werken.

De `canonical`/Open Graph-tags, `robots.txt` en `sitemap.xml` staan al ingesteld
op de root (`https://cafferata.info/`).

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
