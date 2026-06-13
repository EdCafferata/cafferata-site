# Automatisering — NAS, Mac mini of OpenClaw?

> Onderzoek door Claude op 13 juni 2026, autonoom uitgevoerd terwijl Ed weg was.
> Vraag: hoe automatiseer ik builds/releases en agent-taken het beste — op de NAS,
> via OpenClaw, of op de Mac mini?

## Het korte antwoord

Het zijn drie verschillende soorten "automatisering". Niet kiezen tussen, maar
**combineren** — elk doet waar het goed in is:

| | **Mac mini** | **Synology NAS** | **OpenClaw** |
|---|---|---|---|
| Wat | macOS-buildmachine | Linux-server 24/7 aan | autonome AI-agent |
| Sterk in | **iOS-builds & releases** (Xcode/fastlane) | web hosten, cron, Docker, backups | "doe iets & app me" via Telegram/WhatsApp |
| Kan géén | 24/7 stroomzuinig draaien | **iOS bouwen** (geen Xcode) | zelf bouwen — het *orkestreert* |
| Draait | launchd + GitHub Actions runner | Container Manager + Task Scheduler | op de Mac mini óf NAS |

**De harde regel:** een iOS-app bouwen en naar TestFlight/App Store sturen kan
**uitsluitend op macOS** (Xcode is vereist). De NAS kan dat niet. Dus alle iOS-apps
(DICOM Player, BVK, RNH, Tattoe, OC-Consent) horen op de **Mac mini**. De NAS doet
het web-werk (Alltrexx, de sites) en geplande klusjes. OpenClaw is een optionele
slimme laag erbovenpop.

---

## 1. Mac mini — de iOS-buildmachine (aanrader voor de apps)

Je hebt fastlane al werkend (BVK/RNH: `fastlane beta`/`release`; Tattoe: `bundle
exec fastlane beta`). Maak daar een **self-hosted GitHub Actions runner** bij, dan
bouwt en publiceert de Mac mini automatisch zodra je een tag/commit pusht.

**Waarom self-hosted i.p.v. GitHub's cloud-macOS:**
- Gratis/onbeperkte buildminuten (cloud-macOS-runners zijn duur per minuut).
- Apple Silicon = ~5× snellere builds, geen virtualisatie.
- Je certificaten/keys staan al lokaal; geen secrets naar de cloud.

**Opzet (eenmalig):**
1. GitHub repo → Settings → Actions → Runners → "New self-hosted runner" → macOS.
2. Download + `./config.sh` met de repo-token, `./svc.sh install` → draait als
   service via **launchd** (start bij boot, pollt GitHub, geen open poort nodig).
3. Workflow `.github/workflows/release.yml` die op een tag `v*` draait:
   `runs-on: self-hosted` → `bundle exec fastlane beta` (of `release`).
4. Secrets (App Store Connect API-key) als repo-secrets of uit de lokale Keychain.

**Alternatief zonder GitHub Actions:** een `launchd`-job of `cron` op de Mac mini die
periodiek `git pull` + `fastlane beta` draait. Simpeler, maar minder inzicht/logging
dan Actions. Voor jouw aantal apps is de Actions-runner het overwegen waard omdat je
dan per repo een nette knop + logs hebt.

> Tip: één runner kan alle iOS-repo's bedienen als je hem op org-niveau registreert
> i.p.v. per repo.

**Klaar template:** [`docs/voorbeeld-ios-release.yml`](voorbeeld-ios-release.yml) —
kopieer naar een iOS-repo als `.github/workflows/release.yml`. Draait alleen op een
tag `v*` en pas zodra er een Mac mini-runner is; tot dan doet het niets.

---

## 2. Synology NAS — web, cron & containers (draait al 24/7)

De NAS is ideaal voor alles wat géén macOS nodig heeft. Je gebruikt 'm al voor de
sites en straks Alltrexx. Twee ingebouwde motors:

**Container Manager (Docker):**
- Draait de Alltrexx-stack (MySQL + Spring Boot + frontend) — staat al klaar in
  `/volume1/Backup-Ed/alltrexx-nas`.
- Kan een **Linux** self-hosted GitHub Actions runner draaien (in een container) voor
  alles wat op Linux bouwt: de Alltrexx-backend (Maven/Docker), de statische sites.
  Let op: Docker-in-Docker vereist toegang tot `docker.sock` (chown naar uid 1000 +
  symlink), en die symlink overleeft een reboot niet → herstel via een Task
  Scheduler boot-up-taak.

**DSM Task Scheduler (de "cron" van Synology):**
Perfect voor terugkerende klusjes, geen code nodig:
- Nachtelijke `git pull` + redeploy van de sites (`rsync` naar `/volume1/web`).
- Backups van de webroot en de Alltrexx-database.
- Certificaat-check / herstart van containers op een schema.
- De boot-up-taak voor de docker.sock-symlink (zie boven).

**Rol-verdeling NAS:** hosten + plannen + Linux-builds. Niet: iOS.

---

## 3. OpenClaw — de autonome agent-laag (optioneel, mét waarschuwing)

**Wat het is:** OpenClaw ("Molty") is een open-source persoonlijke AI-agent (van
PSPDFKit-oprichter Peter Steinberger). Draait lokaal, praat via WhatsApp/Telegram/
Signal/Slack, en *doet* dingen: shell-commando's, browser-automatisering, e-mail,
agenda, bestanden. Je brengt je eigen API-key mee, model-agnostisch.

**Waar het in jouw setup past:** niet als buildmachine, maar als laag die *namens
jou* taken uitvoert en je een berichtje stuurt — bijv. "check elke ochtend of er een
app is afgekeurd en app me", "draai dit onderzoek en zet het op GitHub", "herstart
de Alltrexx-container als hij ligt". Precies het soort "blijf zelf doorgaan"-werk.

**Waar draaien:**
- **Op de Mac mini** als je wilt dat hij ook Xcode/fastlane kan aansturen (alles op
  één macOS-host). Meest krachtig.
- **Op de NAS / een klein Linux-bakje** voor puur headless taken. Minimaal 2 vCPU /
  4 GB RAM, Node 20+, Docker; draait zelfs op een Raspberry Pi 4/5.

**⚠️ Security — dit is belangrijk, OpenClaw voert shell uit:**
- Draai als **non-root**, zet **sandboxing** aan in de config.
- **Gateway niet aan het internet hangen** — bindt standaard op 127.0.0.1; houden zo,
  achter de firewall; eventueel via VPN/SSO benaderen.
- Gebruik **DM-pairing** (geen open policy) zodat alleen jij hem kunt aansturen.
- **Skills audit:** een onderzoek vond **341 kwaadaardige skills** van de 2.857 in
  ClawHub. Installeer alleen skills die je hebt gecontroleerd; gebruik de scanner
  (Clawdex). Begin minimaal.
- Bij voorkeur op een **apart toestel**, niet je hoofd-Mac.

Kort: OpenClaw is gaaf voor agent-klusjes en notificaties, maar behandel het als een
gebruiker met shell-toegang — afschermen en minimaal houden.

---

## Aanbevolen opzet (gefaseerd)

**Fase 1 — nu het meeste rendement:**
- Mac mini: self-hosted GitHub Actions runner → iOS-apps bouwen/uploaden op een tag.
  Begin met één app (DICOM Player of een tracker) als proef, breid daarna uit.
- NAS: Task Scheduler-taak voor nachtelijke site-deploy + database-backup.

**Fase 2 — Alltrexx volledig geautomatiseerd:**
- NAS: Linux GH-runner in Container Manager die de Alltrexx-backend bouwt en de
  stack herstart bij een push.

**Fase 3 — agent-laag (optioneel):**
- OpenClaw op de Mac mini (of een apart Linux-bakje), afgeschermd, voor
  monitoring + "app-me"-taken en autonoom onderzoek. Streng op security.

## Waarom deze verdeling
- iOS **moet** op macOS → Mac mini, en dat is meteen je krachtigste machine voor
  builds.
- De NAS staat tóch al 24/7 aan → ideaal voor hosten, cron en backups; stroomzuinig.
- OpenClaw is geen vervanging van de andere twee maar een *regisseur* erbovenop —
  handig, maar met shell-toegang dus goed afschermen.

## Bronnen
- https://esinx.net/blog/ios-ci-cd-selfhosted/ (iOS CI/CD self-hosted Mac mini runner)
- https://macstadium.com/blog/github-actions-self-hosted-runner-for-ios-ci-at-macstadium
- https://github.com/andrelin/synology-github-runner (self-hosted runner op Synology)
- https://www.damirscorner.com/blog/posts/20230414-RunGitHubActionsOnASynologyNas.html
- https://mariushosting.com/synology-schedule-start-stop-for-docker-containers/
- https://www.digitalocean.com/resources/articles/what-is-openclaw
- https://www.dsebastien.net/how-to-self-host-openclaw-securely-on-a-vps-a-security-first-guide/
- https://clawtrust.ai/blog/openclaw-server-requirements
