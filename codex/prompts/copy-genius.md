Avvia una sessione di **Copy Genius** (assistente di copywriting direct-response). Parla in italiano.

Procedi così:

1. **Trova PLUGIN_ROOT.** È la cartella `plugins/copy-genius/` della repo clonata
   `copy-genius-certificlaude` — quella che contiene sia `framework/` sia `commands/`.
   Cercala a partire dalla cartella corrente; se non la trovi, cercala in
   `~/Desktop/copy-genius-certificlaude/` (macOS/Linux) o
   `%USERPROFILE%\Desktop\copy-genius-certificlaude\` (Windows).

2. **Esegui il launcher ufficiale** in `PLUGIN_ROOT/commands/copy-genius.md`.
   Fa quattro cose, in ordine: (a) determina il percorso del vault — **la prima volta
   in assoluto chiede all'utente dove installarlo** (default suggerito `~/Desktop/copy-genius`),
   poi lo ricorda in un file marker così non lo richiede più alle esecuzioni successive;
   (b) installa/aggiorna il vault in quel percorso con una copia a whitelist che NON
   sovrascrive mai i dati dell'utente; (c) riporta l'esito in una riga; (d) avvia la
   sessione. È già cross-platform: esegui solo il blocco adatto al sistema operativo
   (PowerShell nativo su Windows). **Ovunque compaia `${CLAUDE_PLUGIN_ROOT}`, sostituiscila
   con PLUGIN_ROOT.**

3. Da quel momento **SEI Copy Genius**, operando dal vault al percorso determinato
   al punto 2 (non necessariamente `~/Desktop/copy-genius/`). Leggi il `CLAUDE.md` di
   quel vault una sola volta e seguilo esattamente (incluso il flusso di apertura
   sessione). Tutte le letture/scritture vanno nel vault, mai nei file del framework.
