# Übergabedokument — futuremove.one (SFM · Sunra Elektroroller Österreich)

**Stand:** Oktober 2026
**Auftraggeber:** Energeia Vertriebs GmbH · Jan Wellmann
**Zweck dieses Dokuments:** Übergabe des Projekts an einen Entwickler, der die Website an ein **eigenes E-Commerce-Backend** anbindet (Warenkorb, Checkout, Zahlung, Bestandsführung).

---

## 1. Überblick

futuremove.one ist die Produkt- und Vorbestell-Website für die SFM-/Sunra-Elektroroller, die Energeia in Österreich vertreibt. Sie ist aktuell eine **statische Single-Page-Website** (eine einzige `index.html` mit inline CSS/JS), gehostet über **GitHub Pages**.

Funktionen heute:
- Zweisprachig **Deutsch/Englisch** (automatische DACH-Erkennung → Deutsch zuerst, Umschalter DE/EN oben rechts).
- Cinematic Hero, Modell-Finder (3 Fragen), Modell-Übersicht (10 Modelle), Preisvergleich.
- **Live-Verfügbarkeit** aus einer zentralen Datenquelle.
- Formulare für **Reservierung / Vorbestellung** und **Händler-Anfragen**, die in ein Backend geschrieben werden und E-Mails auslösen.

Das aktuelle Backend ist eine **Google-Apps-Script-Übergangslösung** (siehe Abschnitt 7). Es ist bewusst über eine **schmale, klar definierte Schnittstelle** (Abschnitt 6) angebunden, damit es 1:1 durch ein eigenes Backend ersetzt werden kann.

---

## 2. Hosting, Domain, Live-URL

- **Live-URL:** https://futuremove.one
- **Hosting:** GitHub Pages (Branch `main`, Root-Verzeichnis).
- **Repository:** `github.com/JanWellmann23/sfm-dach-brief`
- **Custom Domain:** über die Datei `CNAME` im Repo (`futuremove.one`). Datei `.nojekyll` ist vorhanden (verhindert Jekyll-Verarbeitung).
- **DNS:** Die DNS-Einträge für futuremove.one (A-Records auf GitHub Pages bzw. CNAME) liegen beim Domain-Registrar des Auftraggebers. Zugang zu den DNS-Einstellungen bitte beim Auftraggeber anfragen — nur nötig, falls das Hosting später umgezogen wird.
- **Deployment:** Jeder Push auf `main` wird von GitHub Pages automatisch veröffentlicht (ca. 1 Minute). Kein Build-Schritt, kein Framework.

---

## 3. Repository-Struktur

| Datei / Ordner        | Zweck |
|-----------------------|-------|
| `index.html`          | **Die Live-Website** (zweisprachige 2.0-Version). Enthält HTML, CSS, JS inline. |
| `index2.html`         | Identische Kopie von `index.html` (Arbeits-/Sicherungskopie). |
| `index1.html`         | Alte v1-Version (4-sprachig) — nur als Backup/Rollback. |
| `assets-v2/`          | Produktbilder (WebP, transparenter Hintergrund), nach Modell sortiert. |
| `assets/`             | Alte Assets der v1 (nicht mehr genutzt). |
| `_preview_2.0/`       | Lokaler Vorschau-Ordner (per `.gitignore` ausgeschlossen, nicht im Repo). |
| `CNAME`               | Custom-Domain-Eintrag (`futuremove.one`). |
| `.nojekyll`           | Schaltet Jekyll auf GitHub Pages ab. |
| `README.md`           | Kurzbeschreibung. |
| `HANDOVER.md`         | **Dieses Dokument.** |

> Hinweis: `index.html` und `index2.html` sind aktuell deckungsgleich. GitHub Pages liefert `index.html` aus. `index2.html`/`index1.html` können nach der Übergabe entfernt werden.

---

## 4. Technischer Aufbau der Website

- **Eine Datei, kein Framework, kein Build.** Vanilla HTML/CSS/JS. CSS im `<style>`, Logik im `<script>` am Seitenende.
- **Sektionen:** Hero (Cinematic) → Manifesto → Finder (3-Fragen-Matcher) → Modell-Übersicht (`#grid`) → Preisvergleich → Reservierung (`#reservieren`) → Händler (`#haendler`) → Footer.
- **Mehrsprachigkeit (DE/EN):**
  - Alle statischen Texte tragen `data-i18n` / `data-i18n-html` / `data-i18n-ph`-Attribute; Übersetzungen liegen im JS-Objekt **`T`** (`T.de`, `T.en`).
  - Sprachwahl: `detectLang()` → gespeicherte Wahl (`localStorage` Key `fm_lang`) > Browsersprache `de*` oder Zeitzone DACH → `de`, sonst `en`.
  - Umschalter: `#langtog` (DE/EN). `applyLang(lang)` setzt alle Texte, rendert dynamische Teile neu und merkt sich die Wahl.
- **Produktdaten** stehen im JS-Array **`MODELS`** (siehe Abschnitt 5). Übersicht, Finder und Reservierungsformular werden daraus gerendert.
- **Scroll-Reveal / Hero-Animation** rein clientseitig.

---

## 5. Datenmodell (Produkte)

Die 10 Modelle liegen im Array `MODELS` in `index.html`. Pro Modell u. a.:

```js
{
  id:'dream7', name:'Dream 7', tag:{de:'Flaggschiff',en:'Flagship'},
  speed:110, power:'6 kW', batt:'72 V / 72 Ah', price:5999,
  colors:[['Schwarz','#1b1b1d'],['Grau','#8a8f98'],['Orange','#e8651f']],
  img:'assets-v2/dream7/c.webp',
  status:'available',                 // 'available' | 'preorder' (Fallback)
  stock:{'Schwarz':12,'Grau':12,'Orange':12},  // Fallback-Bestände
  blurb:{de:'…',en:'…'}
}
```

**Wichtig — Produktschlüssel:**
- Die Website identifiziert Produkte heute über `model_id` (interne ID) und über **Modellname + Farbe** (`model` + `color`). **SKU/Artikelnummer ist aktuell NICHT im `MODELS`-Array hinterlegt.**
- Die **maßgeblichen Artikelnummern (SKU, Format `A260…`, farbspezifisch)** stehen in der **SFM-Preisliste 2027** (PDF, beim Auftraggeber im Ordner `China Transport/ENERGEIA`).
- **Empfehlung für das neue Backend:** SKU/Artikelnummer als zentralen Schlüssel einführen und ins `MODELS`-Array aufnehmen. Das ist auch die Brücke zu Warenwirtschaft/BaseLinker (Abschnitt 9).

**Preise & Bestände** sind die „source of truth" aus der SFM-Preisliste 2027 (Preise, UVP/HEK) und der Bestandsliste „Angebot Bestände" (Stand 09.10.2026). Diese Unterlagen beim Auftraggeber anfordern.

---

## 6. ⭐ Backend-Schnittstelle (das Wichtigste für die Anbindung)

Die Website spricht genau **einen Endpunkt** an, hinterlegt als **eine Zeile** in `index.html`:

```js
const API_BASE = "…/exec";   // aktuell die Google-Apps-Script-Web-App-URL
```

Um ein eigenes Backend anzubinden, genügt es im ersten Schritt, **diesen Endpunkt durch einen eigenen zu ersetzen**, der die zwei folgenden Verträge erfüllt. Danach kann schrittweise Warenkorb/Checkout/Zahlung ergänzt werden.

### 6.1 Verfügbarkeit lesen — `GET API_BASE`
Antwort: **JSON-Array** mit je einem Eintrag pro Modell+Farbe:

```json
[
  {"model":"Dream 7","color":"Schwarz","status":"Sofort lieferbar","verfuegbar":12},
  {"model":"Future City","color":"Grün","status":"Sofort lieferbar","verfuegbar":10},
  {"model":"EM1","color":"Schwarz","status":"Vorbestellung","verfuegbar":0}
]
```
- `status`: Text, der „lieferbar" enthält → gilt als sofort verfügbar; sonst Vorbestellung.
- `verfuegbar`: Zahl (verfügbare Stück).
- Verwendung: `syncLive()` liest das beim Laden, aktualisiert Badges, Bestandsanzeige und die Farb-Optionen („X verfügbar / ausverkauft"). **Matching erfolgt über `model` (Name) + `color`.** Schlägt der Aufruf fehl, greifen die statischen Fallback-Werte aus `MODELS`.
- **CORS:** Der Endpunkt muss `Access-Control-Allow-Origin: *` (oder futuremove.one) setzen, damit der Browser die Antwort lesen darf.

### 6.2 Vorgang schreiben — `POST API_BASE`
- Body: **JSON als `Content-Type: text/plain;charset=utf-8`** (bewusst, um CORS-Preflight zu vermeiden).
- Die Website sendet „fire-and-forget" (`send()` liest die Antwort nicht aus). Das eigene Backend sollte trotzdem sauber antworten (idealerweise JSON + CORS-Header).

**Reservierung / Vorbestellung:**
```json
{
  "type":"reservation",            // oder "preorder"
  "model":"Dream 7", "model_id":"dream7", "color":"Orange",
  "name":"…", "email":"…", "phone":"…",
  "lang":"de",                     // "de" | "en" — für sprachrichtige Bestätigungen
  "submitted_at":"2026-10-12T09:00:00.000Z"
}
```

**Händler-Anfrage:**
```json
{
  "type":"dealer",
  "shop":"…", "city":"…", "name":"…", "email":"…", "phone":"…",
  "message":"…", "lang":"de",
  "submitted_at":"2026-10-12T09:00:00.000Z"
}
```

Relevante Stellen im Code (`index.html`): `const API_BASE`, Funktion `send()`, `syncLive()`, die Submit-Handler von `#resform` und `#dealform`.

---

## 7. Aktuelles Backend (Übergangslösung — Google Apps Script)

Damit die Seite ab Tag 1 funktioniert, läuft aktuell ein **Google-Apps-Script-Web-App**, gebunden an ein Google Sheet („SFM Bestand & Verfügbarkeit (Energeia)"):

- **`doGet`** → liefert den Verfügbarkeits-Feed (Abschnitt 6.1) live aus dem Sheet.
- **`doPost`** → schreibt jeden Vorgang in das Tab **`Bestellungen`**, erhöht bei Reservierung/Vorbestellung die Spalte **Reserviert/Verkauft** (Verfügbar = Bestand − Reserviert, per Formel) und versendet E-Mails (siehe Abschnitt 8).
- SFM/Energeia pflegt die Verfügbarkeiten direkt im Sheet; die Website liest sie live.
- Der vollständige Quellcode liegt beim Auftraggeber (`Code.gs`). Der Apps-Script-Editor ist über das Sheet erreichbar (Erweiterungen → Apps Script).

**Für die Ablösung:** Das eigene Backend muss lediglich die zwei Verträge aus Abschnitt 6 erfüllen. Danach `API_BASE` auf den neuen Endpunkt setzen, committen, pushen — fertig. Das Apps Script / Sheet kann parallel weiterlaufen, bis das neue Backend verifiziert ist, und danach stillgelegt werden.

> Redeploy-Hinweis zum Apps Script (falls noch daran gearbeitet wird): Code **speichern** → Bereitstellen → **Bereitstellungen verwalten** → vorhandene Bereitstellung **bearbeiten** → Version **„Neue Version"** → Bereitstellen. Nur so bleibt die `/exec`-URL gleich. „Neue Bereitstellung" erzeugt eine **neue** URL (dann `API_BASE` anpassen).

---

## 8. E-Mails / Benachrichtigungen

Aktuell über **Google MailApp** (Absender = das Google-Konto, dem das Script gehört):
- **Interne Benachrichtigung** jeder Reservierung / Vorbestellung / Händler-Anfrage → **`service@energeia-shop.at`, mit Kopie an `janwellmann@proton.me`**.
- **Kunden-/Händlerbestätigung** → an die eingegebene Adresse des Absenders, zweisprachig (DE/EN je nach `lang`), **mit Kopie (BCC) an `janwellmann@proton.me`**.
- Signatur/Impressum: **Energeia Vertriebs GmbH** + vollständige Rechtszeile.

> Soll-Routing (so im Produktivbetrieb beizubehalten): alle Kunden- und Händler-Vorgänge laufen bei **`service@energeia-shop.at`** auf, mit Kopie an **`janwellmann@proton.me`**. In der Übergangslösung ist das bereits so konfiguriert.

**Wichtig — Absenderadresse muss geändert werden:** Aktuell ist der Absender das private Google-Konto. Für den Produktivbetrieb **muss der Absender auf eine offizielle Energeia-Adresse umgestellt werden** (z. B. `service@energeia-shop.at` bzw. eine `@energeia-shop.at`-Adresse, alternativ eine `@futuremove.one`-Adresse). Der private Gmail-Absender ist nicht produktionstauglich.

**Empfehlung für den Versand:** Transaktions-E-Mails über einen Dienst mit verifizierter Domain versenden — z. B. **Resend** (vom Auftraggeber bereits im Einsatz). Dafür die gewählte Absender-Domain (energeia-shop.at bzw. futuremove.one) in Resend verifizieren (SPF/DKIM/DMARC-DNS-Einträge — unabhängig von den GitHub-Pages-Einträgen) und als Absender die Energeia-Adresse nutzen. Das verbessert die Zustellbarkeit deutlich gegenüber dem Versand „als Gmail-Konto".

---

## 9. ⭐ Zukünftiges Backend-Update: BaseLinker-Anbindung

**SFM (der Lieferant) führt Bestand und Bestellungen in BaseLinker** (`https://api.baselinker.com`). Mittelfristig soll **BaseLinker die zentrale Datenquelle** werden und das Google Sheet ersetzen — keine Doppelpflege.

**Wichtige Rahmenbedingung:** Die Website ist statisch und darf BaseLinker **nicht direkt** aufrufen (der API-Token `X-BLToken` wäre im Browser-Code sichtbar; außerdem CORS). BaseLinker muss daher **immer hinter einem serverseitigen Proxy** liegen, der den Token hält. Empfohlen: ein **Cloudflare Worker** (der Auftraggeber betreibt bereits Cloudflare-Worker). Das fügt sich nahtlos in die Schnittstelle aus Abschnitt 6 ein.

**Vorgeschlagene Architektur** (Worker als Proxy):
- **`GET /availability`** → BaseLinker `getInventoryProductsStock` (+ `getInventoryProductsList`), Ergebnis im **gleichen JSON-Format wie Abschnitt 6.1** zurückgeben. Antwort ~60 s in KV cachen (BaseLinker-Limit: 100 Requests/Minute).
- **`POST /order`** → Reservierung/Vorbestellung via `addOrder` in BaseLinker anlegen; Bestätigungs-E-Mail (Resend).
- Dadurch ist der Umstieg **nahezu drop-in**: nur `API_BASE` auf den Worker zeigen lassen.

**Produktschlüssel:** SKU/Artikelnummer (`A260…`) = BaseLinker-Produktschlüssel. „Sofort lieferbar" vs. „Vorbestellung" ergibt sich automatisch aus dem echten Bestand (Stück > 0 → lieferbar). Reservierungen reservieren/reduzieren den Bestand nativ in BaseLinker und laufen in dessen Fulfillment (Rechnung, Versandlabel).

**Zwei Phasen (mit SFM abgestimmt):**
- **Phase 1:** Live-Verfügbarkeit lesen + Reservierungen/Vorbestellungen in BaseLinker anlegen.
- **Phase 2 (sobald echte Bestellungen laufen):** **Statusänderungen und Sendungsnummern (Versendungsnummer)** zurück an die Website/den Kunden — idealerweise per **BaseLinker-Webhook** (alternativ Polling via `getOrders`). Damit erhält der Kunde automatisch Tracking + Status-Updates (bestätigt → in Bearbeitung → versandt → zugestellt).

**Von SFM anzufordern (für Phase 1):**
1. BaseLinker **API-Token** (`X-BLToken`).
2. **Inventar-/Katalog-ID** (`getInventories`).
3. Bestätigung, dass **SFM-SKUs = Artikelnummern** (`A260…`) sind, und ob **Farben eigene Produkte oder Varianten** unter einer SKU sind (bestimmt das Mapping).
4. Zielstatus/-quelle, in die Reservierungen vs. Vorbestellungen laufen sollen.

**BaseLinker-Referenz:** Endpoint `https://api.baselinker.com/connector.php`; Auth `X-BLToken`-Header; POST mit `method` + `parameters` (JSON); Limit 100/Min. Relevante Methoden: `getInventories`, `getInventoryProductsList`, `getInventoryProductsData`, `getInventoryProductsStock`, `addOrder`.

---

## 10. Anbindung an das Custom-E-Commerce-Backend — Vorgehen

1. **Datenquelle wählen:** eigenes Backend als „source of truth", oder (empfohlen, siehe Abschnitt 9) BaseLinker über einen Proxy.
2. **Zwei Endpunkte bereitstellen** gemäß Abschnitt 6 (GET-Verfügbarkeit, POST-Vorgang), inkl. CORS.
3. **`API_BASE`** in `index.html` auf den neuen Endpunkt setzen → `git commit` → `git push` (GitHub Pages veröffentlicht automatisch).
4. **SKU/Artikelnummer** als Produktschlüssel einführen (ins `MODELS`-Array aufnehmen; Quelle: SFM-Preisliste 2027).
5. **Warenkorb / Checkout / Zahlung** ergänzen: Die Reservierungs-/Vorbestell-Formulare sind der Einstiegspunkt. Die Seite ist bewusst modular gehalten, um später einen Warenkorb + Zahlungsanbieter anzuhängen.
6. **E-Mails** auf Resend (verifizierte Domain) umstellen (Abschnitt 8).
7. **Einbindung auf energeia-shop.de:** Die Seite soll außerdem auf der Startseite von energeia-shop.de eingebunden/verlinkt werden (mit Auftraggeber abstimmen).

---

## 11. Rechtliches / Firmendaten (für Impressum, E-Mails, Rechnungen)

```
Energeia Vertriebs GmbH
Aigenstraße 46, 4600 Thalheim bei Wels, Österreich
Vertreten durch: Daniel Grenzner
Tel: +43 664 5550175 · service@energeia-shop.at
Firmenbuch: FN 687378 a, Landesgericht Wels
UID: ATU83660717
```
Impressum/Datenschutz verlinken aktuell auf `energeia-shop.at/impressum` bzw. `/datenschutz`.

---

## 12. Offene Punkte / To-dos

- **Produktbilder fehlen** für **Legend, Future Sport, Urbano** (aktuell sauberer Platzhalter „Foto folgt"). Bilder vom Auftraggeber nachreichen und in `assets-v2/<modell>/` ablegen.
- **Absender-E-Mail unbedingt auf eine Energeia-Adresse umstellen** (z. B. `service@energeia-shop.at`, alternativ `@futuremove.one`) — der private Gmail-Absender ist nur eine Übergangslösung (Resend, Abschnitt 8).
- **Testdaten bereinigen** im Google Sheet (Testzeilen im Tab `Bestellungen`; Testreservierung bei Dream 7/Orange).
- **BaseLinker-Zugänge** bei SFM anfordern (Abschnitt 9) — sobald die Phase-1-Anbindung gebaut wird.
- Aufräumen im Repo möglich: `index1.html`, `index2.html`, `assets/` (Altbestand).

---

## 13. Kontakte

- **Auftraggeber / Projektleitung:** Jan Wellmann — jan.wellmann@gmail.com
- **Energeia (SFM-Vertrieb AT):** service@energeia-shop.at · Daniel Grenzner
- **Lieferant / Warenwirtschaft:** SFM (BaseLinker)

---

*Dieses Dokument liegt auch im Repository (`HANDOVER.md`) und wird mit dem Code übergeben.*
