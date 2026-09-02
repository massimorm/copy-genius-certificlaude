# Copy Genius

Assistente di copywriting direct-response per Claude Code. Ti porta da brand + pubblico + offerta fino al copy finito (ads, advertorial, landing, VSL, email, upsell, libri), passando per ricerca di mercato e funnel brief.

Si attiva con il comando `/copy-genius` dentro Claude Code.

---

## Installazione (studenti)

Ti servono solo **Claude Code** installato ([download](https://www.anthropic.com/claude-code)) e un account Anthropic attivo. Non ti serve un account GitHub.

Apri Claude Code e digita questi **due comandi**, uno alla volta:

```
/plugin marketplace add copynerdai/copy-genius-certificlaude
```

```
/plugin install copy-genius@copynerd
```

Poi **riavvia Claude Code** (chiudi e riapri) e digita:

```
/copy-genius
```

Al primo avvio, Copy Genius ti chiede **dove vuoi installare il vault** (la cartella dove vivranno i tuoi brand, swipe e note) — premi invio per usare il default proposto (`~/Desktop/copy-genius/`), oppure indica un altro percorso. La scelta viene ricordata: alle esecuzioni successive non te lo chiede più, installa e aggiorna sempre nello stesso posto.

Fatto. Nessun file da spostare a mano.

> **Obsidian (opzionale)**: per navigare il vault visivamente, apri la cartella del vault (quella che hai scelto o confermato al primo avvio) come vault in [Obsidian](https://obsidian.md) ("Apri cartella come vault"). Parti da `index.md`.

---

## Come si usa

Apri Claude Code, digita `/copy-genius`, e il sistema parte. Da quel momento puoi:

- creare un nuovo brand (ti guida l'orchestratore)
- scrivere copy (landing, email, ad, VSL, libri) per i tuoi brand
- fare ricerca di mercato
- analizzare swipe e distillare note di strategia

Tutto il tuo lavoro — brand, swipe, note, feedback — vive nella cartella del vault (quella scelta al primo avvio) **sul tuo computer**. Resta privato e locale: non viene mai caricato da nessuna parte.

---

## Aggiornamenti

Quando esce una nuova versione, aggiorni con un comando dentro Claude Code:

```
/plugin update copy-genius@copynerd
```

Al `/copy-genius` successivo, Copy Genius rinfresca il framework e **lascia intatti i tuoi brand, swipe, note e feedback**. Non devi salvare niente da parte: il tuo lavoro è al sicuro per costruzione.

---

## Cosa NON viene mai toccato da un aggiornamento

Il tuo lavoro. In dettaglio, queste cartelle/file dentro la cartella del vault sono tuoi e protetti:

| Protetto (tuo) | Aggiornato (framework) |
|---|---|
| `brands/` (i tuoi brand) | `CLAUDE.md`, `core/strategic-frameworks/`, `core/writing/` |
| `swipe/` (il tuo swipe file) | `skills/`, `format-specialists/`, `section-specialists/` |
| `strategy-notebook.md` | `brands/_template/`, `index.md` |
| `core/feedback-rules.md` (regole globali) | |
| `core/writing/banned-phrases-user.md` (frasi bandite) | |

---

## Problemi?

- **`/copy-genius` non compare** dopo l'install → hai riavviato Claude Code? Chiudi e riapri.
- **Errore sul marketplace** → ricontrolla di aver scritto esattamente `copynerdai/copy-genius-certificlaude`.
- **Al primo `/copy-genius` chiede dove installare il vault e poi il permesso di scrivere in quella cartella** → è normale. Rispondi con un percorso (o premi invio per il default `~/Desktop/copy-genius/`) e dai **Allow / Sì**. Te lo chiede una sola volta: dalla seconda esecuzione in poi installa/aggiorna sempre nello stesso posto senza richiederlo.
- **Ho sbagliato a digitare il percorso al primo avvio, come lo cambio?** → cancella il file marker (macOS/Linux: `~/.copy-genius/vault-path.txt`; Windows: `%USERPROFILE%\.copy-genius\vault-path.txt`) e rilancia `/copy-genius`: te lo richiederà da capo. Se vuoi anche spostare i dati già creati, sposta prima manualmente la cartella del vecchio vault nel nuovo percorso.
- Altri dubbi → contatta il canale di supporto del corso.

### Windows — piano B (solo se l'auto-installazione non parte)

Se su Windows il primo `/copy-genius` non riesce a creare la cartella da solo, puoi installare il vault a mano in 1 minuto, da Esplora File (usa qui il percorso di default; se ne hai scelto uno diverso, sostituiscilo):

1. Vai su **https://github.com/copynerdai/copy-genius-certificlaude** → pulsante verde **Code** → **Download ZIP**.
2. Estrai lo ZIP (tasto destro → Estrai tutto).
3. Dentro la cartella estratta apri: `plugins\copy-genius\framework\`
4. Seleziona **tutto il contenuto** di quella cartella `framework` (Ctrl+A) e **copialo** (Ctrl+C).
5. Sul **Desktop** crea una cartella chiamata esattamente `copy-genius`, entraci e **incolla** (Ctrl+V).
6. Torna in Claude Code e digita `/copy-genius` → trova il vault già pronto e parte.

Path finale corretto: `C:\Users\<tuo-utente>\Desktop\copy-genius\` con dentro un file `CLAUDE.md`.
