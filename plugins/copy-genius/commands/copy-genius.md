---
description: Activate Copy Genius — a direct-response copywriting assistant. Installs/updates the vault on first run, then starts the session.
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep]
---

# Copy Genius — launcher (install · update · run)

You are launching a Copy Genius session. This command is self-installing and self-updating, **cross-platform (macOS / Linux / Windows)**. Run the phases below **in order**, then hand off to the orchestrator.

## Key locations

- **Framework source (read-only, lives in the plugin)**: `${CLAUDE_PLUGIN_ROOT}/framework/`
- **Working vault (the student's, lives on their machine)**: a folder **the student chooses**, resolved in Phase 1 below. Suggested default if they don't have a preference:
  - macOS/Linux: `$HOME/Desktop/copy-genius`
  - Windows: `%USERPROFILE%\Desktop\copy-genius`
- **Vault-path marker (remembers the student's choice across runs)**:
  - macOS/Linux: `$HOME/.copy-genius/vault-path.txt`
  - Windows: `%USERPROFILE%\.copy-genius\vault-path.txt`

The framework is what the author ships and updates. The vault is where the student works — their brands, swipe, notebook, and feedback accumulate there and must **never** be overwritten by an update, and its location must **never** be silently reassigned once chosen.

## The invariant (read this — it governs everything below)

Copy Genius is installed by a **whitelist copy**: the install copies ONLY the known framework paths into the vault. It never copies over — and therefore never touches — the student's data. This is safer than an exclude list: a path that isn't in the framework whitelist is, by definition, left exactly as the student left it.

**FRAMEWORK paths** (copied on install AND refreshed on every update):
`CLAUDE.md`, `index.md`, `VERSION`, `core/conventions.md`, `core/strategic-frameworks/` (all), `core/writing/writing-principles.md`, `core/writing/emotional-intelligence.md`, `skills/` (all), `format-specialists/` (all), `section-specialists/` (all), `brands/_template/` (all).

**USER-DATA paths** (seeded once on first install, then NEVER overwritten):
`brands/` (every brand folder except `_template/`), `swipe/`, `strategy-notebook.md`, `raw/`, `core/feedback-rules.md`, `core/writing/banned-phrases-user.md`, `monitoraggio/` (the competitive-ads archive written by `ad-scraping`), `skills/ad-scraping/node_modules/` + `skills/ad-scraping/archive-root.txt` (installed dependencies and local archive override — the framework copy never deletes, so they survive).

Two files inside those paths are framework rules that must reach an already-installed vault: `core/writing/banned-phrases-user.md` and `swipe/full-text-rules.md`. They are seeded when MISSING (never overwritten if present) — see the update block below.

---

## Phase 1 — Determine the vault path (ask once, remember forever)

**First, detect the operating system.** Then run the matching block below to check whether the vault path is already known — **do not ask the user anything yet.**

### macOS / Linux (and Git Bash on Windows) — POSIX shell

```bash
MARKER="$HOME/.copy-genius/vault-path.txt"
DEFAULT_VAULT="$HOME/Desktop/copy-genius"

if [ -f "$MARKER" ] && [ -s "$MARKER" ]; then
  echo "COPYGENIUS_VAULT=$(cat "$MARKER")"
elif [ -f "$DEFAULT_VAULT/CLAUDE.md" ]; then
  # Pre-existing install from before this feature — adopt it silently, don't re-ask.
  mkdir -p "$(dirname "$MARKER")"
  printf '%s' "$DEFAULT_VAULT" > "$MARKER"
  echo "COPYGENIUS_VAULT=$DEFAULT_VAULT"
else
  echo "COPYGENIUS_VAULT_UNSET default=$DEFAULT_VAULT"
fi
```

### Windows (native PowerShell)

```powershell
$Marker = "$env:USERPROFILE\.copy-genius\vault-path.txt"
$DefaultVault = "$env:USERPROFILE\Desktop\copy-genius"

if ((Test-Path $Marker) -and ((Get-Content $Marker -Raw).Trim().Length -gt 0)) {
  "COPYGENIUS_VAULT=$((Get-Content $Marker -Raw).Trim())"
} elseif (Test-Path "$DefaultVault\CLAUDE.md") {
  New-Item -ItemType Directory -Force -Path (Split-Path $Marker) | Out-Null
  Set-Content -Path $Marker -Value $DefaultVault -NoNewline
  "COPYGENIUS_VAULT=$DefaultVault"
} else {
  "COPYGENIUS_VAULT_UNSET default=$DefaultVault"
}
```

**Read the result:**

- `COPYGENIUS_VAULT=<path>` → the vault path is already known (either from a previous run, or adopted from an existing pre-feature install). **Do not ask anything.** Use `<path>` as `VAULT` and go straight to Phase 2.
- `COPYGENIUS_VAULT_UNSET default=<path>` → this is a genuinely first-ever install on this machine. **Ask the student**, in chat, where they want the Copy Genius vault installed. Mention the suggested default (`<path>` from the output, translated to the right OS syntax) and that they can press enter / just confirm to accept it, or type a different folder. **Wait for their answer before continuing** — do not proceed with a default silently.
  - Resolve whatever they answer to an absolute path (expand `~` or `%USERPROFILE%`; if they gave a relative path, resolve it against their home directory). If they gave no answer / confirmed, use the suggested default.
  - Save the resolved path so this question is **never asked again**:

    macOS/Linux:
    ```bash
    VAULT="<resolved absolute path>"
    mkdir -p "$(dirname "$HOME/.copy-genius/vault-path.txt")"
    printf '%s' "$VAULT" > "$HOME/.copy-genius/vault-path.txt"
    ```

    Windows:
    ```powershell
    $Vault = "<resolved absolute path>"
    New-Item -ItemType Directory -Force -Path (Split-Path "$env:USERPROFILE\.copy-genius\vault-path.txt") | Out-Null
    Set-Content -Path "$env:USERPROFILE\.copy-genius\vault-path.txt" -Value $Vault -NoNewline
    ```
  - Use that same resolved path as `VAULT` for Phase 2.

## Phase 2 — Install or update the vault

Both blocks implement the exact same whitelist logic; they differ only in shell. Both are idempotent: first run installs (framework + empty user scaffolds); later runs refresh only the framework and leave all user data untouched; same version = no-op. Both read `VAULT` fresh from the marker file, so they work regardless of how Phase 1 resolved it.

**Write the script to a temp file first, then run the file — do not paste the whole script as one inline Bash/PowerShell command.** This script is long and uses command substitutions (`$(...)`); some Claude Code environments cap how large a single inline command can be once it contains those (observed limit: ~965 bytes), and this script is well over that. Writing it to a file sidesteps the limit because the file's own content has no such cap, and the command that *runs* the file is short and substitution-free.

### macOS / Linux (and Git Bash on Windows) — POSIX shell

1. Use the **Write** tool to save the following content to `/tmp/copygenius-install.sh` (use `$TMPDIR/copygenius-install.sh` instead if `/tmp` isn't writable):

   ```bash
   SRC="${CLAUDE_PLUGIN_ROOT}/framework"
   VAULT="$(cat "$HOME/.copy-genius/vault-path.txt")"

   copy_framework() {
     mkdir -p "$VAULT/core/strategic-frameworks" "$VAULT/core/writing" "$VAULT/skills" \
              "$VAULT/format-specialists" "$VAULT/section-specialists" "$VAULT/brands/_template"
     cp -f  "$SRC/CLAUDE.md" "$SRC/index.md" "$SRC/VERSION" "$VAULT/"
     cp -f  "$SRC/core/conventions.md" "$VAULT/core/"
     cp -Rf "$SRC/core/strategic-frameworks/." "$VAULT/core/strategic-frameworks/"
     cp -f  "$SRC/core/writing/writing-principles.md" "$SRC/core/writing/emotional-intelligence.md" "$VAULT/core/writing/"
     cp -Rf "$SRC/skills/." "$VAULT/skills/"
     cp -Rf "$SRC/format-specialists/." "$VAULT/format-specialists/"
     cp -Rf "$SRC/section-specialists/." "$VAULT/section-specialists/"
     cp -Rf "$SRC/brands/_template/." "$VAULT/brands/_template/"
   }
   seed_userdata() {   # first install only — never overwrites an existing file
     mkdir -p "$VAULT/raw" "$VAULT/swipe" "$VAULT/core/writing"
     cp -Rf "$SRC/swipe/." "$VAULT/swipe/"
     cp -f  "$SRC/strategy-notebook.md" "$VAULT/"
     cp -Rf "$SRC/raw/." "$VAULT/raw/"
     cp -f  "$SRC/core/feedback-rules.md" "$VAULT/core/"
     cp -f  "$SRC/core/writing/banned-phrases-user.md" "$VAULT/core/writing/"
   }

   if [ ! -f "$VAULT/CLAUDE.md" ]; then
     mkdir -p "$VAULT"; copy_framework; seed_userdata
     echo "COPYGENIUS_RESULT=INSTALLED version=$(cat "$VAULT/VERSION" 2>/dev/null)"
   else
     PLUGIN_V="$(cat "$SRC/VERSION" 2>/dev/null)"; VAULT_V="$(cat "$VAULT/VERSION" 2>/dev/null)"
     # seed any user-data file that is MISSING (older vault) without overwriting existing ones
     [ -f "$VAULT/core/feedback-rules.md" ]            || cp -f "$SRC/core/feedback-rules.md" "$VAULT/core/"
     [ -f "$VAULT/core/writing/banned-phrases-user.md" ] || { mkdir -p "$VAULT/core/writing"; cp -f "$SRC/core/writing/banned-phrases-user.md" "$VAULT/core/writing/"; }
     [ -f "$VAULT/swipe/full-text-rules.md" ]           || { mkdir -p "$VAULT/swipe"; cp -f "$SRC/swipe/full-text-rules.md" "$VAULT/swipe/"; }
     if [ "$PLUGIN_V" != "$VAULT_V" ]; then
       copy_framework
       echo "COPYGENIUS_RESULT=UPDATED from=${VAULT_V:-unknown} to=${PLUGIN_V}"
     else
       echo "COPYGENIUS_RESULT=UPTODATE version=${VAULT_V}"
     fi
   fi
   ```

2. Then run it with a short, substitution-free command: `bash /tmp/copygenius-install.sh` (adjust the path if you used `$TMPDIR`).

### Windows (native PowerShell)

1. Use the **Write** tool to save the following content to `%TEMP%\copygenius-install.ps1`:

   ```powershell
   $SRC   = "$env:CLAUDE_PLUGIN_ROOT\framework"
   $VAULT = (Get-Content "$env:USERPROFILE\.copy-genius\vault-path.txt" -Raw).Trim()

   function Copy-Framework {
     New-Item -ItemType Directory -Force -Path "$VAULT\core\strategic-frameworks","$VAULT\core\writing","$VAULT\skills","$VAULT\format-specialists","$VAULT\section-specialists","$VAULT\brands\_template" | Out-Null
     Copy-Item -Force "$SRC\CLAUDE.md","$SRC\index.md","$SRC\VERSION" "$VAULT\"
     Copy-Item -Force "$SRC\core\conventions.md" "$VAULT\core\"
     Copy-Item -Recurse -Force "$SRC\core\strategic-frameworks\*" "$VAULT\core\strategic-frameworks\"
     Copy-Item -Force "$SRC\core\writing\writing-principles.md","$SRC\core\writing\emotional-intelligence.md" "$VAULT\core\writing\"
     Copy-Item -Recurse -Force "$SRC\skills\*" "$VAULT\skills\"
     Copy-Item -Recurse -Force "$SRC\format-specialists\*" "$VAULT\format-specialists\"
     Copy-Item -Recurse -Force "$SRC\section-specialists\*" "$VAULT\section-specialists\"
     Copy-Item -Recurse -Force "$SRC\brands\_template\*" "$VAULT\brands\_template\"
   }
   function Seed-Userdata {   # first install only
     New-Item -ItemType Directory -Force -Path "$VAULT\raw","$VAULT\swipe","$VAULT\core\writing" | Out-Null
     Copy-Item -Recurse -Force "$SRC\swipe\*" "$VAULT\swipe\"
     Copy-Item -Force "$SRC\strategy-notebook.md" "$VAULT\"
     Copy-Item -Recurse -Force "$SRC\raw\*" "$VAULT\raw\"
     Copy-Item -Force "$SRC\core\feedback-rules.md" "$VAULT\core\"
     Copy-Item -Force "$SRC\core\writing\banned-phrases-user.md" "$VAULT\core\writing\"
   }

   if (-not (Test-Path "$VAULT\CLAUDE.md")) {
     New-Item -ItemType Directory -Force -Path $VAULT | Out-Null
     Copy-Framework; Seed-Userdata
     "COPYGENIUS_RESULT=INSTALLED version=$(Get-Content "$VAULT\VERSION")"
   } else {
     $PLUGIN_V = Get-Content "$SRC\VERSION" -ErrorAction SilentlyContinue
     $VAULT_V  = Get-Content "$VAULT\VERSION" -ErrorAction SilentlyContinue
     if (-not (Test-Path "$VAULT\core\feedback-rules.md"))            { Copy-Item -Force "$SRC\core\feedback-rules.md" "$VAULT\core\" }
     if (-not (Test-Path "$VAULT\core\writing\banned-phrases-user.md")) { New-Item -ItemType Directory -Force -Path "$VAULT\core\writing" | Out-Null; Copy-Item -Force "$SRC\core\writing\banned-phrases-user.md" "$VAULT\core\writing\" }
     if (-not (Test-Path "$VAULT\swipe\full-text-rules.md"))            { New-Item -ItemType Directory -Force -Path "$VAULT\swipe" | Out-Null; Copy-Item -Force "$SRC\swipe\full-text-rules.md" "$VAULT\swipe\" }
     if ($PLUGIN_V -ne $VAULT_V) { Copy-Framework; "COPYGENIUS_RESULT=UPDATED from=$VAULT_V to=$PLUGIN_V" }
     else { "COPYGENIUS_RESULT=UPTODATE version=$VAULT_V" }
   }
   ```

2. Then run it with a short, substitution-free command: `powershell -NoProfile -ExecutionPolicy Bypass -File "$env:TEMP\copygenius-install.ps1"`

**If neither block fits the environment** (unknown shell): apply the invariant by hand — copy ONLY the FRAMEWORK paths listed above from `${CLAUDE_PLUGIN_ROOT}/framework/` into the vault at the path recorded in the marker file; on a first run also copy the USER-DATA scaffolds; on an update never write to a USER-DATA path that already exists. Compare the two `VERSION` files to decide install vs update vs no-op.

## Phase 3 — Report the result (one line, in the user's language)

Read the `COPYGENIUS_RESULT=` line:

- `INSTALLED` → "Copy Genius installato in `<VAULT>`. Pronto." (use the actual resolved vault path, not a hardcoded one)
- `UPDATED from=X to=Y` → "Copy Genius aggiornato (X → Y). I tuoi brand, swipe, note e feedback sono intatti."
- `UPTODATE` → say nothing about it; just proceed.

Keep it to one line. Do not dump the file list.

## Phase 4 — Start the session

Now read `<VAULT>/CLAUDE.md` (the vault copy, at the path resolved in Phase 1 — **not** the plugin copy, and **not** necessarily `~/Desktop/copy-genius`). It is your operating manual for the entire session — identity, architecture, routing, language, and session behavior all live there. Read it ONCE now, then follow it exactly, including its session-open flow (§11). Do not re-read it later in the session.

From this point on you ARE Copy Genius, operating out of the vault at `<VAULT>`. All reads and writes during the session target the vault, never the plugin framework source.
