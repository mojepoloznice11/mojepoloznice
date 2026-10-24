
# Navodila: Hosting PWA aplikacije (Netlify + GitHub Pages) — brezplačno, brez baze

Pripravil bom vse potrebne datoteke in korak-za-korakom navodila za deploy. Dodam tudi `manifest.json` in service worker, da bo prava PWA izkušnja na iPhone (offline delovanje, boljša ikona, brez Safari URL vrstice).

## Pregled, kaj bomo naredili

1. Dodal manifest.json + izboljšan PWA setup v tvoj HTML
2. Dodal preprost service worker za offline delovanje
3. Dal navodila za 2 možnosti deploya (izberi eno):
   - **Netlify Drop** — najhitreje, brez računa, brez git-a
   - **GitHub Pages** — malo več dela, a lažje za kasnejše posodobitve

Najprej pripravim datoteke, nato napišem navodila.

Pripravljam datoteke. Tukaj je vse, kar potrebuješ.

---

## 📁 Datoteka 1: `index.html` (posodobljena verzija)

Tvoja obstoječa datoteka je že skoraj pripravljena za PWA. Dodam samo povezavo na `manifest.json` in registracijo service workerja. Spodaj je **cela posodobljena datoteka** — zamenjaj svojo z njo:

```html
<!DOCTYPE html>
<html lang="sl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Moje Položnice</title>

  <!-- PWA & Mobile Settings -->
  <meta name="theme-color" content="#2563eb">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="Položnice">
  <link rel="manifest" href="manifest.json">
  <link rel="apple-touch-icon" href="icon-192.png">
  <link rel="icon" href="icon-192.png">

  <style>
    /* ... CELOTEN OBSTOJEČI <style> BLOK OSTANE NESPREMENJEN ... */
  </style>
</head>
<body>

  <!-- ... CELOTEN OBSTOJEČI <body> OSTANE NESPREMENJEN ... -->

  <script>
    // ... CELOTEN OBSTOJEČI JS KODA OSTANE NESPREMENJENA ...

    // DODAJ TOLE NA SAM KONEC SCRIPT BLOKA (za offline delovanje):
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('sw.js').catch(err => {
          console.log('Service worker registration failed:', err);
        });
      });
    }
  </script>
</body>
</html>
```

**Pomembno:** Spremenila sta se samo `<head>` del (manifest + ikona povezavi) in dodani so 6 vrstic JS na koncu. Vsa logika za položnice ostane 100% enaka.

---

## 📁 Datoteka 2: `manifest.json` (nova datoteka)

Ustvari novo datoteko z imenom `manifest.json` v isti mapi kot `index.html`:

```json
{
  "name": "Moje Položnice",
  "short_name": "Položnice",
  "description": "Evidenca plačanih in prihajajočih položnic",
  "start_url": "./index.html",
  "scope": "./",
  "display": "standalone",
  "background_color": "#f8fafc",
  "theme_color": "#2563eb",
  "orientation": "portrait",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

---

## 📁 Datoteka 3: `sw.js` (service worker — za offline delovanje)

Nova datoteka `sw.js`, prav tako v isti mapi:

```javascript
const CACHE_NAME = 'poloznice-v1';
const urlsToCache = [
  './',
  './index.html',
  './manifest.json'
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(urlsToCache))
  );
  self.skipWaiting();
});

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(keys.filter((k) => k !== CACHE_NAME).map((k) => caches.delete(k)))
    )
  );
  self.clients.claim();
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => response || fetch(event.request))
  );
});
```

---

## 📁 Ikoni (icon-192.png in icon-512.png)

Potrebuješ dve PNG sliki (kvadratni, z ikono 🧾 ali karkoli želiš). Najlažja pot:

1. Pojdi na **[favicon.io/emoji-favicons](https://favicon.io/emoji-favicons/)**
2. Izberi emoji 🧾 (receipt)
3. Prenesi paket → notri bo `android-chrome-192x192.png` in `android-chrome-512x512.png`
4. Preimenuj ju v `icon-192.png` in `icon-512.png`

*(Če te je lena, lahko za začetek manifest pusti brez icons – aplikacija bo delovala, ikona na home screenu pa bo samo screenshot strani namesto lepe ikone. Priporočam pa, da narediš ta korak za lepši izgled.)*

---

## 🚀 Navodila za deploy — Opcija A: Netlify Drop (najhitreje, 2 minuti, brez računa)

1. Pripravi mapo na računalniku, npr. `polo znice-app/`, in vanjo daj vse 4 datoteke:
   - `index.html`
   - `manifest.json`
   - `sw.js`
   - `icon-192.png`, `icon-512.png`
2. Pojdi v brskalniku na **https://app.netlify.com/drop**
3. Povleci celo mapo (drag & drop) v okno na strani
4. Netlify v nekaj sekundah objavi stran in ti da URL, npr. `https://wonderful-biscuit-12345.netlify.app`
5. Ta URL odpri na **iPhone v Safariju**
6. Klikni ikono **Deli** (kvadrat s puščico navzgor) → **"Add to Home Screen"** (Dodaj na začetni zaslon)
7. Potrdi ime (npr. "Položnice") → Dodaj

Zdaj imaš ikono na home screenu, ki se odpre fullscreen (brez Safari vrstice), deluje tudi offline (zaradi service workerja), in podatki se shranjujejo lokalno v ta "app" na tvojem iPhone.

**Opomba:** Pri brezplačnem Netlify Drop (brez računa) stran ostane živa, a jo težje kasneje posodabljaš (vsakič znova povlečeš mapo gor in dobiš isti/nov URL, odvisno od nastavitve). Če želiš lažje posodabljanje → naredi račun na Netlify (brezplačen) ali uporabi Opcijo B spodaj.

---

## 🚀 Navodila za deploy — Opcija B: GitHub Pages (malo več dela, lažje posodabljanje)

Boljše, če misliš občasno dodajati funkcionalnosti vnaprej.

1. Ustvari brezplačen račun na **github.com** (če ga še nimaš)
2. Ustvari nov repozitorij, npr. `polo znice` (lahko ga nastaviš na **Private**, da ni javno viden)
3. Naloži vanj vse 4-5 datotek (index.html, manifest.json, sw.js, ikoni) — preko spletnega vmesnika: "Add file" → "Upload files"
4. Pojdi v **Settings** repozitorija → **Pages** (levo v meniju)
5. Pod "Branch" izberi `main` in mapo `/ (root)` → Save
6. Po ~1 minuti ti GitHub pokaže URL, nekaj kot:
   `https://tvoje-uporabnisko-ime.github.io/polo znice/`
7. Ta URL odpri na iPhone v Safariju → enako kot zgoraj: Deli → Add to Home Screen

**Prednost GitHub Pages:** ko hočeš kasneje kaj spremeniti v kodi, samo urediš datoteko na GitHubu (ali naložiš novo verzijo) in stran se avtomatsko posodobi — brez ponovnega drag&drop-a.

---

## ⚠️ Pomembne omejitve, ki jih moraš vedeti

1. **Podatki so vezani na ta Safari/iPhone.** Če zbrišeš aplikacijo iz home screena ali počistiš Safari podatke (Settings → Safari → Clear History and Website Data), **izgubiš vse vnesene položnice**. To ni vezano na internet/strežnik — vse živi samo lokalno v localStorage tvojega telefona.
2. **Brez sinhronizacije.** Če bi kdaj hotel odpreti isto evidenco na računalniku ali drugem telefonu, tam ne bo istih podatkov — vsaka naprava ima svoj lokalni localStorage.
3. **Priporočam občasni backup.** Lahko dodam v aplikacijo gumb "Izvozi podatke" (prenese JSON datoteko vseh položnic) in "Uvozi podatke" — to je preprosta dodatna funkcija, če jo želiš, lahko to dodam v kodo zdaj.

---

### Želiš, da:
1. **Dodam funkcijo Export/Import (backup v JSON datoteko)** direktno v `index.html`, da ne izgubiš podatkov, če se kaj zalomi? *(priporočam, 10 min dela)*
2. Pripravim generirani ikoni (192px/512px) namesto da jih sam iščeš na favicon.io?
3. Ali greva kar na backend/Supabase verzijo, če veš da boš res rabil dostop iz več naprav?

Povej, pa nadaljujem.



Dobro vprašanje — tukaj je pomembno razčistiti nesporazum, ker **GitHub Pages sam po sebi tega ne omogoča**.

## Zakaj GitHub Pages ni "baza"

GitHub Pages je **statični hosting** — servira samo datoteke (HTML, CSS, JS, JSON) bralcu, točno tako kot bi jih prebral iz mape na disku. Nima:
- strežniške logike (ne more sprejemati zahtev za pisanje/spreminjanje datotek)
- avtentikacije ali API-ja za zapisovanje

Tvoj brskalnik lahko JSON datoteko iz GitHub Pages samo **prebere** (GET zahteva), ne more pa vanjo **zapisati** nazaj na GitHub. Ko bi na iPhone odkljukal položnico kot plačano, bi se ta sprememba zgodila samo lokalno v tvojem brskalniku (localStorage) — nazaj na GitHub se nikoli ne zapiše. Druga naprava bi še vedno videla staro, prvotno JSON datoteko.

Torej: **ne, sama ta kombinacija ti ne da sinhronizacije med napravami.**

## Kako bi dejansko lahko uporabil GitHub kot "bazo" (obstaja trik, a z omejitvami)

Obstaja pot preko **GitHub API-ja** (ne GitHub Pages, ampak GitHub REST API), kjer app z uporabo **Personal Access Tokena** dejansko lahko piše nazaj v repozitorij (commit nove JSON datoteke). To ljudje včasih uporabljajo kot "poor man's database". Ampak:

- Moral bi token shraniti nekje v aplikaciji/brskalniku → varnostno tveganje (kdorkoli z dostopom do telefona/kode lahko vidi token in spreminja tvoj GitHub repo)
- Vsaka sprememba (vsak klik na "plačano") = en git commit → počasno, in po nekaj mesecih uporabe imaš repo z tisoči commitov
- Rate limiti GitHub API-ja (5000 zahtev/uro za avtenticirane zahteve) — za osebno rabo dovolj, a je "hack", ne prava rešitev
- Potreben je dodaten JS za branje/pisanje preko API-ja (fetch z Authorization header)

**Tehnično izvedljivo, a ni priporočljivo** — je bolj "trik" kot prava rešitev, in ima varnostne luknje pri shranjevanju tokena v client-side kodi.

## Kaj dejansko rabiš za pravo sinhronizacijo med napravami

Potrebuješ **pravi backend z bazo**, ki sprejema zahteve za branje IN pisanje. Najboljše brezplačne opcije za tvoj primer:

| Rešitev | Kaj je | Zastonj tier | Zahtevnost |
|---|---|---|---|
| **Supabase** | Postgres baza + avtomatski API | Da, velikodušen | Nizka-srednja |
| **Firebase (Firestore)** | NoSQL baza od Googla | Da, velikodušen | Nizka-srednja |
| **JSONBin.io / similar** | Preprost "JSON kot servis" | Da, omejen | Zelo nizka |

Vse te rešitve so narejene **ravno za tvoj primer** — majhna osebna app, ki jo odpiraš iz več naprav in rabi obstojne podatke. Namesto `localStorage.setItem(...)` bi v kodi imel `fetch('https://tvoj-projekt.supabase.co/...', {method: 'POST', ...})`.

## Povzetek

- **GitHub Pages sam** = samo hosting statične strani, brez pravega shranjevanja → ni "baza"
- **GitHub kot baza preko API-ja** = tehnično mogoče, a hack z varnostnimi in praktičnimi pomanjkljivostmi
- **Za resnično sinhronizacijo med napravami** rabiš pravo rešitev kot Supabase/Firebase — in to je naslednji logični korak, če ugotoviš, da ti samo-na-enem-telefonu-localStorage ne zadostuje

Če se odločiš, da želiš dostop iz več naprav (telefon + računalnik npr.), je **Supabase** moje priporočilo — brezplačen, enostaven za tvoj primer uporabe, in lahko ti pripravim celotno kodo, ko se odločiš.
