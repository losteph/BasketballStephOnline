# 🏀 Basketball Steph - Portale Tornei Online

Portale web statico e reattivo per la visualizzazione in tempo reale di risultati, classifiche, calendari, tabelloni playoff a eliminazione diretta e statistiche individuali (stile Card FUT) dell'ecosistema Basket Steph.

Il sito opera in modalità sola lettura (read-only): legge i dati generati ed esportati dall'applicazione mobile/PWA tramite file JSON statici posizionati nella cartella `data/`. 

## 📁 Struttura del Progetto

```
repo/
│
├── index.html              # Portale principale 
├── app.js                  # Motore JavaScript 
├── style.css               # Stili dark theme 
│
├── data/                   # Cartella dati del torneo
│   ├── player.json         # Database anagrafico e attributi 
│   ├── teams.json          # Squadre
│   ├── match.json          # Storico delle partite giocate 
│   └── tournaments.json    # Configurazione torneo
│
└── README.md
```

---

## ⚙️ Come Funziona il Portale

- **Caricamento Unificato:** All'avvio, il portale esegue una sola lettura asincrona dei 4 file JSON posizionati in data/.Switch Istantaneo (Zero-Lag): La barra di navigazione superiore ospita i pulsanti per passare al volo tra i vari tornei/gironi (es. Girone A, Girone B, Playoff). 

- **Classifica Basket:** Vittoria (+2 pt), Sconfitta (0 pt). Risoluzione automatica della parità prima tramite scontro diretto, poi tramite Differenza Canestri (DC = PF - PS) e infine Punti Fatti (PF).

- **Pagelline:** Calcolo automatico dei voti individuali basato su canestri (+3, +2, +1), assist, rimbalzi, buone azioni difensive, falli ed errori.

- **Auto-Avanzamento Playoff:** Nella modalità eliminazione diretta, il tabellone controlla i match giocati e sposta automaticamente le squadre vincitrici (o perdenti nelle finaline) al turno successivo.


---

## 🛠️ Guida Passo Passo: Gestione del Torneo

#### **Passo 1:** Configurare Giocatori e Squadre

Esporta i file player.json e teams.json dall'app Basket Steph (oppure compilali manualmente nel percorso data/).

Assicurati che ogni squadra in teams.json contenga gli ID dei giocatori referenziati in player.json.   

#### **Passo 2:** Configurare lo Scheduler (data/tournaments.json)

Questo file è il cervello del portale: definisce quali e quanti gironi esistono e come si sviluppa l'albero dei playoff.

#### **Passo 3:** Inserire le Partite Giocate (data/match.json)

Man mano che le partite vengono disputate sul campo e salvate sull'app Basket Steph, usa il tasto Esporta JSON nello Storico Partite per generare il file match.json aggiornato.

Copia il file scaricato dentro la cartella data/match.json del portale web.

Assicurati che il campo "tournamentId" presente in ogni partita coincida esattamente con l'attributo "id" specificato nel blocco del torneo in tournaments.json.

La pagina web aggiornerà classifiche, tabellini e qualificazioni all'istante.

---

## 📖 Guida Dettagliata allo Scheduler (data/tournaments.json)

Il file tournaments.json permette di alternare due tipologie di competizione: round_robin (fase a gironi/campionato all'italiana) e knockout (fase a eliminazione diretta).

Struttura Globale del file JSON

```json
{
  "activeTournamentId": "girone_a",
  "tournaments": [
    { /* Blocco Girone A */ },
    { /* Blocco Girone B */ },
    { /* Blocco Playoff */ }
  ]
}
```

`activeTournamentId:` Definisce quale modalità/girone mostrare di default al primo caricamento della pagina.

Esempio Pratico 1: Girone all'Italiana (round_robin) Utilizzato per campionati, gironi di qualificazione a 4, 6 o 8 squadre.

```json
{
  "id": "girone_a",
  "name": "GIRONE A",
  "type": "round_robin",
  "teamIds": ["t1", "t2", "t3", "t4"],
  "rounds": [
    {
      "roundName": "1ª Giornata",
      "matches": [
        {
          "homeName": "Vipers",
          "awayName": "Red Bullz",
          "scheduledDate": "16/10/2026 18:00",
          "locationName": "Playground Steph - Campo A"
        },
        {
          "homeName": "Sharks",
          "awayName": "Thunder",
          "scheduledDate": "16/10/2026 19:00",
          "locationName": "Playground Steph - Campo A"
        }
      ]
    },
    {
      "roundName": "2ª Giornata",
      "matches": [
        {
          "homeName": "Vipers",
          "awayName": "Sharks",
          "scheduledDate": "17/10/2026 18:00",
          "locationName": "Playground Steph - Campo A"
        },
        {
          "homeName": "Red Bullz",
          "awayName": "Thunder",
          "scheduledDate": "17/10/2026 19:00",
          "locationName": "Playground Steph - Campo A"
        }
      ]
    }
  ]
}
```

Perché è strutturato così:teamIds: Filtra la classifica e le rose della sezione Squadre mostrando solo le 4 squadre di questo girone, anche se nel file globale teams.json ce ne sono 8 o più.rounds: Organizza il calendario in comode giornate a blocchi separati.scheduledDate e locationName: Finché la partita non è presente in match.json, la card rimarrà nello stato DA GIOCARE mostrando data, orario e campo. Appena inserisci il risultato in match.json, il motore lo aggancerà automaticamente trasformando la card in FINALE con il tabellino interattivo.Esempio Pratico 2: Tabellone Playoff (knockout)Utilizzato per tabelloni a eliminazione diretta (Final Four, Quarti, Semifinali, Finali). Sfrutta il sistema di nodi concatenati con auto-avanzamento.
```json
{
  "id": "playoff",
  "name": "PLAYOFF FINALI",
  "type": "knockout",
  "stages": [
    {
      "stageName": "Semifinali",
      "isSemi": true,
      "matches": [
        {
          "matchCode": "SF1",
          "label": "Semifinale 1",
          "homeName": "Vipers",
          "awayName": "Kings",
          "scheduledDate": "19/10/2026 18:30",
          "locationName": "Centrale"
        },
        {
          "matchCode": "SF2",
          "label": "Semifinale 2",
          "homeName": "Phoenix",
          "awayName": "Red Bullz",
          "scheduledDate": "19/10/2026 19:30",
          "locationName": "Centrale"
        }
      ]
    },
    {
      "stageName": "Finale 3°/4° Posto",
      "isThirdPlace": true,
      "matches": [
        {
          "matchCode": "F3P",
          "label": "Finalina 3° Posto",
          "homeFromLoser": "SF1",
          "awayFromLoser": "SF2",
          "scheduledDate": "20/10/2026 18:00",
          "locationName": "Centrale"
        }
      ]
    },
    {
      "stageName": "Finalissima 1° Posto",
      "isFinal": true,
      "matches": [
        {
          "matchCode": "FIN",
          "label": "Finalissima",
          "homeFromWinner": "SF1",
          "awayFromWinner": "SF2",
          "scheduledDate": "20/10/2026 19:30",
          "locationName": "Centrale"
        }
      ]
    }
  ]
}
```

Come impostare i collegamenti tra turni:matchCode: È la chiave univoca della partita (es. SF1, SF2). Serve ai turni successivi come punto di riferimento.homeFromWinner / awayFromWinner: Indica da quale partita precedente deve arrivare la squadra vincitrice. Non appena SF1 viene giocata, il vincitore prende automaticamente il posto nel blocco FIN.homeFromLoser / awayFromLoser: Indica che lo slot è riservato alla squadra sconfitta (ideale per finaline 3°/4° posto).Flag Grafici:isSemi: true: Applica il badge azzurro e la cornice dedicata alle semifinali.isThirdPlace: true: Applica il badge rosso per la finalina.isFinal: true: Applica il tema oro. Al termine della partita, il vincitore viene incoronato nel Banner Campione Torneo in cima al tabellone con trofeo e animazione.Esempio Pratico 3: Come Estendere a Quarti di Finale (8 Squadre Knockout)Per creare un torneo a eliminazione diretta partendo dai Quarti di Finale, basta aggiungere lo stage dei quarti prima delle semifinali:
```json
{
  "stageName": "Quarti di Finale",
  "matches": [
    { "matchCode": "QF1", "label": "Quarto 1", "homeName": "Vipers", "awayName": "Hawks", "scheduledDate": "18/10/2026 15:00" },
    { "matchCode": "QF2", "label": "Quarto 2", "homeName": "Red Bullz", "awayName": "Black Panthers", "scheduledDate": "18/10/2026 16:00" },
    { "matchCode": "QF3", "label": "Quarto 3", "homeName": "Sharks", "awayName": "Kings", "scheduledDate": "18/10/2026 17:00" },
    { "matchCode": "QF4", "label": "Quarto 4", "homeName": "Thunder", "awayName": "Phoenix", "scheduledDate": "18/10/2026 18:00" }
  ]
},
{
  "stageName": "Semifinali",
  "isSemi": true,
  "matches": [
    { "matchCode": "SF1", "label": "Semifinale 1", "homeFromWinner": "QF1", "awayFromWinner": "QF2", "scheduledDate": "19/10/2026 18:00" },
    { "matchCode": "SF2", "label": "Semifinale 2", "homeFromWinner": "QF3", "awayFromWinner": "QF4", "scheduledDate": "19/10/2026 19:00" }
  ]
}
```
Il motore risolverà automaticamente l'intero flusso: QF1 e QF2 alimenteranno SF1, mentre QF3 e QF4 alimenteranno SF2, portando i rispettivi vincitori fino alla Finalissima senza alcuna riga di codice extra.