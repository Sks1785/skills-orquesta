---
name: verify
description: Verifica davvero che una modifica funzioni prima di considerarla conclusa. Da usare quando si creano o si modificano app (Flutter/Android), configurazioni di server, Docker, Frigate, Home Assistant, n8n o qualsiasi progetto di sicurezza e automazione domestica. Impone controlli reali e prove invece di affermare che «dovrebbe funzionare».
---

# Skill: verify (verificare)

## Principio
Non sei il verificatore per intuizione: sei il verificatore per PROVE.
Non dire mai «è risolto», «dovrebbe funzionare» o «ho trovato il problema» senza aver eseguito un controllo reale e visto il suo risultato.
Se non puoi eseguire qualcosa, dillo e segna quel punto come NON VERIFICATO.

## Protocollo fisso (sempre gli stessi 6 passi, in quest'ordine)

1. **Identificare il progetto e il suo profilo.** Decidi quale si applica (vedi «Profili» sotto). Se non è chiaro, chiedi una volta sola.
2. **Leggere prima di affermare.** Apri i file e la configurazione reali coinvolti. Non dare per scontato il contenuto. Se stai per dire «il guasto è in X», mostra prima di aver letto X.
3. **Definire il criterio di successo.** Scrivi in una o due righe cosa significa «funziona» per questa modifica (es.: «l'app compila e la schermata delle impostazioni si apre senza errori»).
4. **Eseguire i controlli del profilo.** Usa comandi reali (non inventati). Se un comando fallisce, includilo così com'è nel rapporto.
5. **Provare il caso che prima falliva.** Riproduci il problema originale e conferma che non si verifica più. Se non riesci a riprodurlo, dillo.
6. **Riferire con il formato obbligatorio** (vedi sotto).

## Formato obbligatorio del rapporto

```
VERIFICA
Progetto: <nome/tipo>
Criterio di successo: <una riga>

Controlli eseguiti:
- [OK] <comando o prova> → <risultato rilevante>
- [ERRORE] <comando o prova> → <errore>
- [NON VERIFICATO] <cosa manca e perché>

Cosa ho letto: <file/configurazioni esaminati>
Conclusione: VERIFICATO / PARZIALE / NON VERIFICATO
Rischi residui: <elenco breve o «nessuno noto»>
Prossimo passo consigliato: <una riga>
```

Regole del formato:
- Segna VERIFICATO solo se TUTTI i controlli critici sono OK.
- Cita gli errori in modo letterale e breve; non riassumerli a memoria.
- Separa ciò che è stato controllato da ciò che è stato supposto. Il supposto va come «suppongo», mai come fatto.

## Profilo A: app (Flutter / Android)

Controlli, dal meno al più costoso (fermati al primo che fallisce):
1. `flutter analyze` → nessun errore (elenca gli avvisi).
2. `flutter test` → tutti i test passano. Se non ci sono test per ciò che è cambiato, dillo e proponine uno.
3. `flutter build apk --debug` (o `./gradlew assembleDebug` in Android nativo) → compila.
4. Se è disponibile un emulatore/dispositivo: `flutter run` oppure installare l'APK e provare il flusso interessato.
5. Esaminare `adb logcat` (filtrando per il pacchetto dell'app) cercando eccezioni all'apertura della schermata modificata.

Extra secondo il caso:
- Modifiche alle dipendenze: `flutter pub get` senza conflitti; controlla la versione minima dell'SDK.
- Modifiche a permessi o notifiche: esamina `AndroidManifest.xml` e prova su un dispositivo con Android recente.
- Prima di pubblicare su Google Play: build di release firmata e verifica che `versionCode` sia stato incrementato.
- Se la modifica riguarda l'archiviazione dei dati: prova con dati esistenti (migrazione), non solo con un'installazione pulita.

## Profilo B: sicurezza e automazione domestica

Regola d'oro: **prima solo lettura**. Non modificare la configurazione in produzione senza (a) backup, (b) conferma esplicita dell'utente e (c) piano di ripristino.

Controlli tipici (adattali all'installazione reale; non dare per scontato che un comando esista, verificalo):
- **Docker / docker-compose:** `docker compose config` (valida la sintassi), `docker compose ps` (stato), `docker logs --tail 100 <contenitore>` (errori recenti). Esamina percorsi dei volumi e porte prima di avviare qualcosa.
- **Frigate:** valida il file di configurazione con lo strumento di Frigate stesso, se esiste; verifica nei log che ogni telecamera si connetta; conferma che il rilevamento sia attivo e che le registrazioni vengano scritte su disco. Controlla l'uso della CPU se il rilevamento è su CPU.
- **Telecamere:** conferma che il flusso RTSP risponda (ad esempio con `ffprobe` sull'URL, senza mostrare le credenziali nel rapporto). Esamina il budget PoE dello switch (somma dei consumi delle telecamere rispetto alla potenza totale).
- **Home Assistant:** esegui il controllo della configurazione prima di riavviare; poi esamina il registro per errori delle integrazioni.
- **n8n o altre automazioni:** esegui il flusso in modalità di prova con dati di esempio e controlla l'uscita di ogni nodo; conferma fuso orario e credenziali.
- **NAS, rete e UPS (gruppo di continuità):** controlla la connettività (`ping`, porte aperte), lo spazio libero su disco e lo stato dell'UPS (ad esempio con `upsc` se usa NUT). Dopo un'interruzione di corrente simulata o reale, conferma che lo spegnimento ordinato funzioni.
- **Sicurezza:** non esporre telecamere, pannelli o il NAS a Internet senza autenticazione forte e, meglio, una VPN. Esamina le porte aperte e gli avvisi del sistema stesso. Non includere mai password, token o URL con credenziali nel rapporto.

Per qualsiasi modifica di rete o firewall: controlla prima di applicare che non resterai senza accesso (tieni una via di ritorno).

## Cosa fare quando non si può verificare
- Di' esattamente cosa non hai potuto controllare e perché (nessun accesso al dispositivo, nessuna rete, nessun permesso).
- Proponi il comando o la prova concreta che l'utente può eseguire e quale risultato aspettarsi.
- Segna la conclusione come PARZIALE o NON VERIFICATO. Non colmare le lacune con supposizioni.

## Antipattern (vietati)
- Dire «ora funziona» senza aver eseguito nulla.
- Indicare la causa di un guasto senza aver letto il codice o il log pertinente.
- Inventare nomi di comandi, opzioni, file o porte. Se non sei sicuro, verifica o chiedi.
- Cambiare più cose insieme: una modifica, una verifica.
- Nascondere dal rapporto i controlli falliti.

## Manutenzione di questa skill
Ogni volta che un errore sfugge (qualcosa di «verificato» che poi fallisce), aggiungi qui un nuovo controllo che l'avrebbe rilevato. Questo elenco migliora con gli errori reali.
