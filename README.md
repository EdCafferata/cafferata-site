# Cafferata Apps — website

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

Op `cafferata.info` (82.172.143.72, nginx) draait nu een WordPress-site over
**Cafferata's Piper Rising**. Twee opties om deze site ernaast of ervoor te zetten:

**Optie A — subdomein (aanrader, WordPress blijft staan):**
1. Voeg in DNS een record toe: `apps.cafferata.info` → zelfde IP.
2. Maak een nginx server-block (of Web Station virtual host) voor
   `apps.cafferata.info` met als root de map met deze bestanden.
3. Let's Encrypt-certificaat erbij en klaar — de boot-site blijft onaangetast.
   Pas daarna in de HTML de `canonical`/`og:url` tags aan naar `apps.cafferata.info`.

**Optie B — site op de root:**
1. Kopieer de bestanden naar de webroot van het `cafferata.info` server-block:
   ```bash
   rsync -av --delete --exclude '.git' ./ admin@NAS-IP:/volume1/web/cafferata/
   ```
2. Verhuis WordPress dan eerst naar bijv. `piper.cafferata.info` zodat de
   boot-site bereikbaar blijft.

De `canonical`/Open Graph-tags, `robots.txt` en `sitemap.xml` staan nu ingesteld
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
