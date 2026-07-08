# Gestionale Prenotazioni — Template white-label

Web app in **Streamlit** per gestire le prenotazioni di più strutture ricettive
(hotel / B&B): vista settimanale a griglia (verde = libera, rosso = occupata),
inserimento e modifica prenotazioni con **tariffe standard derogabili con ➖ / ➕**,
calcolo incassi (giorno / settimana / mese / anno) e salvataggio **permanente**.

Il template è pronto per il deploy su **Streamlit Community Cloud**. Prima della
consegna a un cliente vanno personalizzati brand, strutture e tariffe, e vanno
configurati i **Secrets** (password di accesso + persistenza su GitHub).

---

## 1. Requisiti

Un file `requirements.txt` con:

```
streamlit
pandas
requests
```

Struttura minima del repository:

```
tuo-repository/
├── app.py
├── requirements.txt
├── README.md
└── data/
    └── prenotazioni.csv   (creato in automatico al primo salvataggio)
```

---

## 2. Deploy su Streamlit Community Cloud

1. Carica `app.py` e `requirements.txt` in un repository **GitHub**.
2. Vai su [share.streamlit.io](https://share.streamlit.io) e collega l'account GitHub.
3. **New app** → scegli repository, branch (`main`) e file principale (`app.py`).
4. **Deploy**. Al primo avvio comparirà la schermata di accesso.

> Il disco di Streamlit Cloud è **temporaneo**: si azzera ad ogni riavvio. Per non
> perdere mai le prenotazioni, configura la persistenza su GitHub (sezione 3.2).

---

## 3. Configurazione dei Secrets

Su Streamlit Cloud: apri l'app → menu **⋮** → **Settings** → **Secrets**, e incolla
il blocco in formato TOML qui sotto (sostituendo i valori segnaposto):

```toml
# --- Accesso all'app ---
password = "la_tua_password"

# --- Persistenza permanente su GitHub ---
github_token  = "IL_TUO_TOKEN_GITHUB"
github_repo   = "tuo-utente/tuo-repository"
github_branch = "main"
```

Salva: l'app si riavvia da sola con le nuove impostazioni.

### 3.1 Password di accesso

Imposta il Secret `password` con **una password a tua scelta** (il valore
`la_tua_password` qui sopra è solo un segnaposto: mettine una tua).

- Il Secret `password` ha la **precedenza** e sovrascrive la password iniziale
  del template senza toccare il codice.
- Finché non imposti questo Secret, resta attiva la **password demo** già inclusa
  nel codice, comoda solo per testare l'app appena deployata. **Prima di andare in
  produzione o di consegnare l'app a un cliente, imposta sempre una tua password
  nei Secrets.**
- La password non viene mai salvata in chiaro nell'URL: l'accesso è mantenuto da
  un token derivato (hash non reversibile), così non si deve ridigitare ad ogni
  ricaricamento.

### 3.2 Persistenza permanente su GitHub (token)

Le prenotazioni vengono scritte (commit automatico) sul file
`data/prenotazioni.csv` del tuo repository. Serve un **token GitHub** con permesso
di scrittura sui contenuti del repo.

#### a) Come generare il token

1. Su GitHub apri **Settings** (impostazioni account) → in fondo a sinistra
   **Developer settings**.
2. **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
3. Compila:
   - **Token name**: un nome riconoscibile (es. `gestionale-prenotazioni`).
   - **Expiration**: scegli una scadenza (es. 12 mesi); ricordati di rigenerarlo
     alla scadenza.
   - **Repository access**: **Only select repositories** → seleziona **solo** il
     repository che ospita l'app.
4. **Permissions** → **Repository permissions** → voce **Contents** → imposta
   **Read and write**. (Le altre voci puoi lasciarle su *No access*.)
5. **Generate token** e **copia subito** il valore mostrato (inizia con
   `github_pat_...`): GitHub lo mostra **una sola volta**.

> In alternativa puoi usare un *classic token* con lo scope **repo**, ma il
> fine-grained qui sopra è più sicuro perché limitato al singolo repository.

#### b) Come inserire il token nei Secrets

Incolla il valore copiato al posto di `IL_TUO_TOKEN_GITHUB` nel blocco della
sezione 3, e compila anche:

- `github_repo` → nel formato `utente/nome-repository` (attenzione alle
  **maiuscole/minuscole**, devono coincidere con quelle reali).
- `github_branch` → di norma `main` (opzionale: se omesso vale `main`).

#### c) Verifica

Dentro l'app apri la sezione **💾 Backup / ripristino dati** → **🔎 Diagnostica
salvataggio su GitHub** → **▶️ Verifica connessione a GitHub**. Se tutto è corretto
vedrai la conferma del permesso di scrittura. Inserisci poi una prenotazione di
esempio per verificare che venga salvata.

---

## 4. Personalizzazione (rebranding)

Tutte le modifiche si fanno in `app.py`:

- **Nome visualizzato** (`TUO NOME HOTEL / B&B`): cambia la costante `BRAND_NAME`
  in cima ad `app.py`. Aggiorna in automatico scheda browser, sidebar e testo
  della **filigrana** di sfondo. Il grande titolo colorato in intestazione va
  aggiornato a parte (cerca `TUO NOME` nel blocco della testata).
- **Filigrana di sfondo** (testo del brand in diagonale + motivo alberghiero, un
  campanello da reception a tratto sottile, su bianco): è generata via CSS, sempre
  presente dietro ogni sezione. Per renderla più o meno marcata regola i valori
  `fill-opacity` / `opacity` nella funzione `_filigrana_uris()`.
- **Nomi delle strutture** (`STRUTTURA 1/2/3`) e **icone** (🏨 🏢 🏡): nel
  dizionario `HOTELS`.
- **Camere**: sempre nel dizionario `HOTELS`, campo `rooms` (numero e tipo di
  ciascuna camera).
- **Tariffe standard**: campo `prezzi` di ogni struttura. Sono valori segnaposto:
  in fase di prenotazione ogni tariffa resta comunque derogabile con ➖ / ➕.
- **Logo/filigrana** (opzionale): metti un file `assets/watermark.png` nel repo e
  comparirà come filigrana tenue; se il file non c'è, semplicemente non viene
  mostrata.
- **Favicon** (opzionale): `assets/icon-192.png`.

---

## 5. Backup

Anche con la persistenza su GitHub attiva, conviene ogni tanto scaricare un
backup: sezione **💾 Backup / ripristino dati** → **⬇️ Scarica backup CSV**.
Da lì puoi anche **ripristinare** i dati da un CSV precedente.
