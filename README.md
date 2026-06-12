# Cafferata Apps — website

Statische website voor op de eigen NAS: landingspagina met alle apps (live in de
App Store én in ontwikkeling), een zakelijk deel voor **The IT Crowd** en een
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

## Deployen op de Synology NAS

1. Zet **Web Station** aan in DSM (Package Center → Web Station installeren).
2. Kopieer de bestanden naar de webroot:
   ```bash
   rsync -av --delete --exclude '.git' ./ admin@NAS-IP:/volume1/web/
   ```
   (of via File Station naar de gedeelde map `web` slepen)
3. Klaar — de site draait op `http://NAS-IP/`. Koppel eventueel een eigen
   domein + Let's Encrypt-certificaat via DSM → Control Panel → Security.

## Huisstijl

Kleuren komen uit het medische dark theme van Dicom Player:
achtergrond `#080E1A`, surface `#0F1A2E`, card `#162035`,
accent teal `#00C2CB`, blauw `#2F80ED`.

## Apps bijwerken

Nieuwe app live? Voeg een `<article class="card">` toe in `index.html` onder
`#live` (of `#onderweg`) — kopieer een bestaande kaart en pas icoon
(`icon-teal`/`icon-blue`/…), badge (`badge-live`/`badge-soon`/`badge-dev`)
en links aan.
