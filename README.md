# Trip Regolarità 2.0 — contachilometri GPS con media

App web installabile (PWA) per Android che usa il GPS del telefono per misurare lo spostamento
e la **velocità media di viaggio**.

## Novità della 2.0.3

- **Traccia GPX più fedele**: salvataggio automatico sul telefono, tratti separati dove manca il segnale, scarto dei salti GPS impossibili, ultimo punto sempre incluso.

## Novità della 2.0

- **Velocità media di viaggio** in km/h, calcolata **escludendo soste e pause**.
- **Velocità attuale** in km/h, aggiornata in tempo reale.
- **Ingranaggio in alto a destra** con le impostazioni: puoi mostrare o nascondere la media e scegliere la
  **lingua** (italiano predefinita o inglese). Le scelte vengono ricordate sul telefono e la media continua a
  essere calcolata anche quando è nascosta.
- Tutte le funzioni della 1.0 restano invariate.

## Cosa fa

- **Distanza parziale** — in **metri**, grande, in alto, con tasto **RESET PARZIALE**.
- **Distanza totale** — in chilometri, campo separato, con tasto **RESET TOTALE**: azzera totale, parziale e
  media, imposta **N° Vignetta a 1** e mette in **pausa** la registrazione, senza conferma.
- **Velocità media di viaggio** — distanza totale diviso il tempo trascorso dal tasto **AVVIA**
  al **RESET TOTALE**; include anche soste, fermi e pause.
- **Velocità attuale** — stima su una finestra di alcuni secondi, filtrata per non seguire il rumore del GPS.
- **Impostazioni** — tocca l'ingranaggio in alto a destra per mostrare/nascondere media e velocità attuale
  e per scegliere la **lingua** dell'interfaccia (italiano/inglese, italiano predefinita). I numeri seguono
  il formato della lingua scelta (0,00 km / 0.00 km).
- **Salvataggio percorso (opzionale)** — attiva **"Salvataggio percorso?"** nelle impostazioni: con **AVVIA**
  parte anche la registrazione della traccia, che **prosegue anche dopo RESET TOTALE** (il reset azzera solo
  i conteggi del viaggio, non la traccia) e resta attiva anche in pausa. Per salvarla apri le impostazioni e
  premi **ESPORTA ORA**: il file **GPX** finisce nella cartella Download e la registrazione della traccia si
  chiude lì (al prossimo **AVVIA** ne parte una nuova). La traccia viene conservata sul telefono (si ripristina se l'app viene chiusa dal sistema) finché non premi **ESPORTA ORA**. Dopo una chiusura non esportata, riaprendo l'app la traccia recuperata resta ferma ed è esportabile dalle impostazioni; il prossimo **AVVIA** la **sostituisce** con una traccia nuova. Se il GPS resta senza segnale per più di 20 s si apre un nuovo tratto (<trkseg>) invece di una linea retta, i salti GPS impossibili (> 250 km/h) vengono scartati e l'ultimo punto valido entra sempre nel file.
  Per rivederla su **Google Maps**: apri *mymaps.google.com* → *Crea mappa* → *Importa* e scegli il file
  (il GPX si apre anche in molte altre app di mappe e navigatori).
- **N° Vignetta** — conta quante volte è stato premuto **RESET PARZIALE**, parte da **1** e si riporta a 1
  con **RESET TOTALE**.
- **AVVIA / PAUSA** sempre visibile in basso (barra fissa).
- **All'avvio si riparte da zero**: 0 m, 0,00 km, media 0,0 km/h e N° Vignetta 1
  (nessun dato precedente viene recuperato).
- **Schermo sempre accesso** durante la registrazione (Wake Lock, dove supportato).
- Funziona **offline** dopo il primo caricamento (service worker).
- **Modalità demo**: aggiungi `?demo=1` (o `#demo`) all'indirizzo per simulare uno spostamento senza muoverti.

## Test sul telefono (5 minuti)

1. Doppio click su **`avvia.cmd`** sul PC: avvia il server locale (porta **8010**, per non interferire con la 1.0) e un tunnel HTTPS Cloudflare.
2. Nella finestra nera compare un link tipo `https://xxxx-xxxx.trycloudflare.com`.
3. Apri quel link con **Chrome su Android**.
4. Consenti l'accesso alla posizione quando richiesto.
5. Menu ⋮ di Chrome → **Installa app** (o *Aggiungi a schermata Home*): ora parte a schermo intero come un'app.
6. Premi **AVVIA** e cammina o guida.

> Il link del tunnel è temporaneo: cambia a ogni avvio e funziona solo se il PC è acceso e la finestra resta aperta.
> Un indirizzo `http://192.168.x.x` **non** funziona: Android blocca il GPS se la connessione non è HTTPS.

### Alternativa senza internet — USB

```powershell
# una volta sola: installa platform-tools e attiva il debug USB sul telefono
adb reverse tcp:8010 tcp:8010
```

Poi apri `http://localhost:8010` sul telefono: `localhost` è considerato sicuro, quindi il GPS funziona.

### Alternativa permanente — hosting

- Trascina la cartella su **app.netlify.com/drop** (gratis, HTTPS incluso), oppure usa GitHub Pages.
- Con l'app online puoi generare un **APK** con [PWABuilder](https://www.pwabuilder.com/) (Trusted Web Activity),
  installabile da file senza Play Store.
- In alternativa si può rifare l'app in Kotlin nativo e compilare l'APK con Android Studio.

## Come vengono calcolate distanza e velocità

- **Distanza**: formula di **Haversine** tra due posizioni GPS consecutive.
- Un rilevamento viene usato solo se la sua accuratezza è migliore di **±30 m** (costante `MAX_ACC`).
- Uno spostamento viene sommato solo se supera `max(2,5 m, 50% dell'accuratezza)`: evita che il "rumore"
  del GPS gonfi la distanza da fermo.
- **Tempo di viaggio**: calcolato dal tasto **AVVIA** fino al **RESET TOTALE**, include soste, fermi e pause.
- **Media di viaggio** = `distanza totale ÷ tempo trascorso (AVVIA → RESET TOTALE)`.
- **Velocità attuale**: distanza tra la posizione più vecchia (circa 3 secondi fa) e quella attuale, divisa
  per il tempo trascorso; il valore viene smussato e portato a 0 quando il movimento è trascurabile.

## File

| File | Ruolo |
|---|---|
| `index.html` | tutta l'app (UI, GPS, calcoli, media) |
| `manifest.webmanifest` | identità dell'app installabile (nome, icone, colori) |
| `sw.js` | service worker, funzionamento offline |
| `icon-*.png` | icone app |
| `serve.js` | server statico minimale (`node serve.js 8010`) |
| `avvia.cmd` | avvia server + tunnel HTTPS |
| `tools/cloudflared.exe` | binario del tunnel (portabile, nessuna installazione) |
| `make-icons.ps1` | rigenera le icone |

## Limiti noti

- Android sospende il GPS quando il browser va in background: **tieni l'app in primo piano** durante il tragitto.
- Precisione tipica all'aperto: 3–10 m, molto peggiore tra edifici alti o in galleria.
- Distanza calcolata in linea retta tra i punti: su percorsi molto tortuosi la stima è leggermente inferiore al reale.
- La velocità attuale è una stima filtrata: in caso di segnale scarso può oscillare o restare a 0.
- Non c'è mappa: aggiungerne una richiede connessione per le tile.
