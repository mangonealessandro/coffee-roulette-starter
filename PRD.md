# Coffee Roulette - PRD

## Executive Summary

**Problem:** I colleghi che lavorano in un'azienda tendono a interagire sempre con le stesse persone, creando silos informali che riducono la coesione del team e limitano lo scambio di idee tra reparti.

**Solution:** Coffee Roulette è uno strumento interno che abbina automaticamente i colleghi per incontri informali a cadenza settimanale, con la possibilità di richiedere un match istantaneo on-demand.

**Value:** Aumentare la connessione interpersonale all'interno dell'azienda, favorire la cultura aziendale e rompere i silos comunicativi, senza richiedere sforzo organizzativo da parte dei dipendenti.

---

## Background & Context

### Situazione attuale

Oggi i dipendenti organizzano i propri incontri informali in modo autonomo, tendendo a interagire solo con colleghi già conosciuti. Non esiste uno strumento dedicato che faciliti connessioni casuali tra persone di team o reparti diversi.

### Perché ora

Con la crescita del team e l'incremento del lavoro ibrido/remoto, le occasioni informali di incontro si sono ridotte drasticamente. Serve un meccanismo strutturato che ricrei queste connessioni casuali in modo scalabile.

### Cosa succede se non lo risolviamo

- Onboarding più lento per i nuovi assunti
- Meno collaborazione cross-team
- Riduzione del senso di appartenenza aziendale

---

## Goals & Success Metrics

### Business Goals

- Aumentare le connessioni cross-team del 50% nei primi 3 mesi
- Raggiungere un tasso di partecipazione attiva del 70% dei dipendenti entro il primo mese

### KPIs

| Metrica | Baseline | Target | Misurazione |
|---|---|---|---|
| Dipendenti iscritti | 0 | 80% del team | Conteggio utenti attivi |
| Match completati (confermati) | 0 | 60% dei match proposti | Feedback in-app |
| Match settimanali per utente | 0 | 1 a settimana | Log di sistema |
| Retention settimanale | 0% | 65% dopo 4 settimane | Utenti che mantengono iscrizione |

### Criteri di successo utente

- Un utente può iscriversi e ricevere il primo match in meno di 2 minuti
- Un utente può richiedere un match on-demand in meno di 30 secondi
- Un utente può gestire le proprie preferenze (disponibilità, pausa) senza supporto

---

## User Personas

### Marco, Sviluppatore Backend

- **Contesto:** Lavora in un team tecnico di 8 persone, prevalentemente remote. Conosce bene i colleghi diretti ma ha poca visibilità sugli altri reparti.
- **Pain Points:**
  - Non sa con chi parlare fuori dal suo team
  - Trova imbarazzante avvicinarsi a sconosciuti senza un pretesto
  - Dimentica di "socializzare" quando è immerso nel lavoro
- **Goals:** Conoscere persone nuove in azienda, allargare la sua rete interna senza sforzo
- **Tech Level:** Expert

---

### Giulia, HR Manager

- **Contesto:** Gestisce l'onboarding e la cultura aziendale. Cerca strumenti che aumentino la coesione senza richiedere eventi formali costosi.
- **Pain Points:**
  - Gli eventi aziendali sono costosi e poco scalabili
  - I nuovi assunti faticano a integrarsi al di là del proprio team
  - Non ha visibilità su quanto i colleghi si connettano tra loro
- **Goals:** Migliorare la cultura aziendale, supportare l'onboarding, avere metriche di engagement
- **Tech Level:** Intermediate

---

### Luca, Nuovo Assunto (1 mese in azienda)

- **Contesto:** Ha appena iniziato. Conosce solo il suo manager e i colleghi diretti. Vorrebbe espandere la rete interna ma non sa da dove cominciare.
- **Pain Points:**
  - Non sa chi sono le persone degli altri team
  - Teme di disturbare colleghi senior se li contatta direttamente
  - Mancano occasioni strutturate per incontri informali
- **Goals:** Conoscere persone oltre il suo team, sentirsi parte dell'azienda più rapidamente
- **Tech Level:** Intermediate

---

## User Stories

### Priorità Alta (MVP)

1. **Come dipendente**, voglio iscrivermi a Coffee Roulette con il mio account aziendale, così da partecipare agli abbinamenti.

2. **Come dipendente**, voglio ricevere ogni settimana una notifica con il nome e il profilo del collega con cui sono stato abbinato, così da poter organizzare un caffè.

3. **Come dipendente**, voglio poter confermare o rifiutare un match proposto, così da avere il controllo sui miei impegni.

4. **Come dipendente**, voglio poter richiedere un match on-demand istantaneo con un collega disponibile, così da non dover aspettare il ciclo settimanale.

5. **Come dipendente**, voglio poter mettere in pausa la mia partecipazione per un periodo (es. ferie, periodo intenso), così da non ricevere match quando non sono disponibile.

### Priorità Media (Post-MVP)

6. **Come dipendente**, voglio vedere un profilo base del collega abbinato (nome, team, ruolo), così da avere un punto di partenza per la conversazione.

7. **Come dipendente**, voglio poter segnalare che un caffè è avvenuto, così da contribuire alle metriche aziendali.

8. **Come HR Manager**, voglio avere una dashboard con le statistiche di utilizzo aggregate (match effettuati, partecipazione per team, retention), così da misurare l'impatto del programma.

9. **Come dipendente**, voglio ricevere un suggerimento di conversazione o un'icebreaker question quando ricevo un match, così da rompere il ghiaccio più facilmente.

10. **Come amministratore**, voglio poter gestire manualmente gli utenti (aggiungere, rimuovere, sospendere), così da mantenere il database aggiornato.

---

## Solution Overview

Coffee Roulette è una web application interna che:

1. **Gestisce un pool di dipendenti** iscritti volontariamente al programma
2. **Genera abbinamenti casuali settimanali** evitando ripetizioni recenti tra le stesse coppie
3. **Notifica i partecipanti** dell'abbinamento con le informazioni del collega
4. **Permette match on-demand** per chi vuole un incontro extra fuori dal ciclo settimanale
5. **Offre controllo agli utenti** sulla propria partecipazione (pausa, conferma/rifiuto match)

---

## Core User Flow

### Flow Principale: Match Settimanale

```
1. Dipendente si registra → crea profilo (nome, team, ruolo, foto opzionale)
2. Sistema ogni lunedì → genera abbinamenti dal pool degli iscritti attivi
3. Dipendente riceve notifica → "Sei stato abbinato con [Nome], [Ruolo] del team [Team]"
4. Dipendente visualizza match → vede profilo del collega
5. Dipendente conferma interesse → sistema registra la conferma
6. I due colleghi si organizzano autonomamente → caffè avviene fuori dall'app
7. Dipendente segna il caffè come avvenuto (opzionale) → sistema aggiorna metriche
```

### Flow Secondario: Match On-Demand

```
1. Dipendente accede alla sezione "Match Istantaneo"
2. Dipendente richiede un abbinamento ora
3. Sistema cerca un collega disponibile nel pool (escludendo match recenti)
4. Sistema propone un collega → dipendente accetta o chiede un altro suggerimento
5. L'altro collega riceve una notifica di richiesta match
6. L'altro collega accetta o declina
7. Se accetta → entrambi ricevono i dettagli reciproci
```

### Edge Cases

- **Nessun collega disponibile per match on-demand:** Il sistema informa l'utente e suggerisce di riprovare più tardi
- **Pool di utenti dispari per match settimanale:** Un utente rimane senza match quella settimana (ruota a settimana successiva)
- **Utente in pausa:** Escluso dagli abbinamenti fino a riattivazione manuale
- **Stessa coppia abbinata di recente:** Sistema evita re-abbinamento per almeno N settimane configurabili

---

## Functional Requirements

### Must-Have (MVP)

- [ ] **Registrazione utente:** I dipendenti si iscrivono con email aziendale e creano un profilo base (nome, cognome, team, ruolo)
  - Why critical: Prerequisito per qualsiasi funzionalità

- [ ] **Login / Autenticazione:** Accesso sicuro tramite email aziendale + password (o link magico)
  - Why critical: Necessario per proteggere i dati degli utenti

- [ ] **Pool di partecipazione:** L'utente può attivare/disattivare la propria partecipazione settimanale
  - Why critical: Partecipazione deve essere volontaria e controllabile

- [ ] **Match settimanale automatico:** Il sistema genera abbinamenti casuali ogni settimana per tutti gli utenti attivi, evitando coppie già abbinate di recente
  - Why critical: Funzionalità core del prodotto

- [ ] **Visualizzazione match:** L'utente vede il profilo del collega abbinato (nome, team, ruolo)
  - Why critical: Senza questa info il match non ha valore

- [ ] **Conferma/Rifiuto match:** L'utente può accettare o declinare il match proposto
  - Why critical: Rispetto dell'autonomia dell'utente

- [ ] **Match on-demand:** L'utente può richiedere un abbinamento istantaneo fuori dal ciclo settimanale
  - Why critical: Secondo caso d'uso principale dichiarato

- [ ] **Pausa partecipazione:** L'utente può mettere in pausa gli abbinamenti per un periodo definito
  - Why critical: Senza questa feature, gli utenti si disiscriveranno invece di fare pausa

- [ ] **Notifiche in-app:** L'utente riceve notifiche per nuovi match e richieste on-demand
  - Why critical: Il valore del prodotto dipende dall'awareness del match

### Should-Have (Post-MVP)

- [ ] **Profilo arricchito:** Foto profilo, breve bio, interessi
  - Impact: Alto — migliora qualità delle conversazioni
  - Effort: Piccolo

- [ ] **Segnalazione caffè avvenuto:** L'utente conferma che l'incontro ha avuto luogo
  - Impact: Alto — abilita metriche di completamento
  - Effort: Piccolo

- [ ] **Dashboard HR / Admin:** Statistiche aggregate su match effettuati, partecipazione per team, trend settimanali
  - Impact: Alto per Giulia (HR)
  - Effort: Medio

- [ ] **Icebreaker questions:** Una domanda casuale suggerita all'atto del match
  - Impact: Medio — riduce attrito nella conversazione
  - Effort: Piccolo

- [ ] **Storico match personale:** L'utente può vedere i colleghi con cui è stato abbinato in passato
  - Impact: Medio
  - Effort: Piccolo

- [ ] **Notifiche email:** Recap settimanale del match via email aziendale
  - Impact: Alto per adoption
  - Effort: Medio

### Won't-Have (Out of Scope)

- ❌ **Integrazione calendario (Google/Outlook):** Aggiunge complessità di integrazione non necessaria per l'MVP
- ❌ **Chat in-app:** Non è uno strumento di messaggistica; i colleghi si organizzano con i propri strumenti aziendali esistenti
- ❌ **Matching per interessi/skill:** L'abbinamento casuale è intenzionale per favorire connessioni inattese
- ❌ **App mobile nativa:** La web app responsive è sufficiente per il caso d'uso
- ❌ **SSO / OAuth enterprise:** Rimandato a post-MVP per ridurre complessità iniziale
- ❌ **Feedback e rating dei match:** Rischia di creare imbarazzo tra colleghi

---

## Non-Functional Requirements

### Performance
- La pagina principale deve caricarsi in meno di 2 secondi
- Il matching on-demand deve produrre un risultato in meno di 3 secondi
- Il sistema deve supportare fino a 500 utenti simultanei senza degradazione

### Sicurezza
- Autenticazione obbligatoria per accedere a qualsiasi dato
- I dati personali degli utenti sono visibili solo agli utenti autenticati
- Nessun dato sensibile esposto nelle API pubbliche
- Password hashate con algoritmo sicuro (bcrypt o equivalente)

### Affidabilità
- Il job di matching settimanale deve essere eseguito con garanzia di almeno-una-volta
- In caso di errore nel matching, il sistema deve notificare l'amministratore

### Usabilità
- L'interfaccia deve essere utilizzabile senza training su desktop e mobile browser
- Tutte le azioni principali raggiungibili in massimo 3 click dalla homepage

### Scalabilità
- L'architettura deve supportare la crescita da 50 a 500 utenti senza refactoring

---

## Open Questions

| # | Domanda | Impatto | Deadline decisione |
|---|---|---|---|
| 1 | Chi gestisce gli account? Auto-registrazione libera o invito da HR? | Alto — impatta registrazione e sicurezza | Prima dello sviluppo |
| 2 | Quante settimane devono passare prima che la stessa coppia possa essere ri-abbinata? | Medio — impatta algoritmo di matching | Prima del matching |
| 3 | Il rifiuto di un match genera un nuovo abbinamento automatico o l'utente rimane senza match quella settimana? | Medio — UX del rifiuto | Prima dello sviluppo |
| 4 | Le statistiche della dashboard HR devono essere anonime o nominative? | Alto — privacy e compliance | Prima della dashboard |
| 5 | Esiste già un sistema di notifiche/email aziendale da integrare, o usiamo un servizio esterno? | Medio — impatta notifiche email post-MVP | Prima delle notifiche email |

---

## Dependencies & Risks

### Dipendenze

- **Accesso ai dati dei dipendenti:** Serve una lista aggiornata di team e ruoli (CSV, API HR, o input manuale)
- **Dominio/Hosting interno:** L'app deve essere accessibile dalla rete aziendale o VPN

### Rischi

| Rischio | Probabilità | Impatto | Mitigazione |
|---|---|---|---|
| Bassa adoption iniziale | Alta | Alto | Comunicazione interna, champion per team |
| Utenti si dimenticano di organizzare il caffè | Alta | Medio | Reminder automatici, facilità di uso |
| Pool troppo piccolo per matching significativo | Medio | Alto | Minimo 10-15 utenti per avviare il programma |
| Privacy concerns sui dati personali | Bassa | Alto | Partecipazione volontaria, profilo minimale |
| Job settimanale fallisce silenziosamente | Bassa | Alto | Monitoring + alerting sul job di matching |

---

*Documento creato il 2026-03-04*
*Versione: 1.0*
