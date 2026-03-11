# CLAUDE.md — AI Assistant Guide for dot-emacs

This is the personal Emacs configuration of John Wiegley. It is a large,
mature, and carefully curated configuration spanning thousands of lines of
Emacs Lisp.

---

## Repository Overview

| File/Directory | Purpose |
|---|---|
| `init.el` | Main Emacs init file (~3,883 lines); bootstraps all packages |
| `settings.el` | Emacs `custom-set-variables` dump (auto-managed by Custom) |
| `dot-org.el` | Org-mode configuration (loaded by Gnus/standalone Org sessions) |
| `dot-gnus.el` | Gnus email/news reader configuration |
| `org-settings.el` | Org-mode `custom-set-variables` |
| `gnus-settings.el` | Gnus `custom-set-variables` |
| `abbrevs.el` | Global and mode-specific abbreviation tables |
| `lisp/` | Custom Emacs Lisp packages and git submodules |
| `snippets/` | YASnippet snippet files organized by major mode |
| `doc/EmacsWiki/` | Archived EmacsWiki documentation (read-only reference) |
| `Makefile` | Byte-compilation build system |

---

## Key Conventions

### Package Management: `use-package`

Every third-party package is declared with `use-package`. There are 443+
`use-package` declarations in `init.el`. Standard patterns:

```elisp
(use-package some-package
  :defer t                        ; lazy-load (preferred)
  :demand t                       ; load immediately (rare, explicit)
  :diminish                       ; hide from mode-line
  :bind (("C-c x" . some-command) ; key bindings
         (:map some-mode-map
               ("C-c y" . other-command)))
  :hook (some-mode . some-function) ; mode hooks
  :preface                        ; evaluated at compile AND load time
  (defun my-helper () ...)
  :init                           ; runs before package loads
  (setq some-var t)
  :config                         ; runs after package loads
  (some-mode 1))
```

Key `use-package` behaviors to be aware of:
- `:defer t` is used pervasively; avoid `:demand t` unless truly necessary
- `:preface` is for helper functions needed at both compile and load time
- `:disabled t` is used to temporarily disable packages without deleting them
- `use-package-expand-minimally t` is set in production (non-debug) mode

### Compilation Contexts

The init file heavily uses these macros:

- `eval-and-compile` — code needed at both byte-compile and runtime
- `eval-when-compile` — code needed only at byte-compile time
- `(require 'use-package)` is inside `eval-and-compile` at the top level

### Nix Integration

Dependencies are managed via **Nix**, not ELPA/MELPA directly. Key patterns:

- `emacs-environment` is read from the `NIX_MYENV_NAME` environment variable
- `nix-read-environment` reads Nix build inputs to extend `load-path`
- Nix profile paths like `~/.nix-profile/share/emacs/site-lisp/` appear as
  `:load-path` values
- The Nix overlay is at: https://github.com/jwiegley/nix-config/blob/master/overlays/10-emacs.nix
- Do **not** add ELPA/MELPA `package.el` install calls; all packages come from Nix

### Multi-Environment Support

The config supports running two simultaneous Emacs instances:

- Standard (`emacs-environment` matches `emacs26*`) → data in `~/.emacs.d/data/`
- Alternate (`emacsHEAD`) → data in `~/.emacs.d/data-alt/`

The `alternate-emacs` boolean constant controls environment-specific behavior.

### Custom Prefix Keymaps

Prefix maps are defined at the top of `init.el` and used throughout:

| Prefix | Map name |
|---|---|
| `C-,` | `my-ctrl-comma-map` |
| `<C-m>` | `my-ctrl-m-map` |
| `C-h e` | `my-ctrl-h-e-map` |
| `C-h x` | `my-ctrl-h-x-map` |
| `C-c b` | `my-ctrl-c-b-map` |
| `C-c e` | `my-ctrl-c-e-map` |
| `C-c m` | `my-ctrl-c-m-map` |
| `C-c w` | `my-ctrl-c-w-map` |
| `C-c y` | `my-ctrl-c-y-map` |

When adding new key bindings, prefer these prefix maps over bare global bindings.

### Startup Performance

The file applies several startup optimizations that are reversed after init:

```elisp
;; Applied at top of init.el (before loading anything)
(setq gc-cons-threshold 402653184   ; ~384 MB
      gc-cons-percentage 0.6
      file-name-handler-alist nil)

;; Restored in after-init-hook
(setq gc-cons-threshold 800000
      gc-cons-percentage 0.1)
```

Do not change these without understanding their performance implications.

---

## File Structure of `init.el`

The file is organized into clearly marked sections:

1. **Functions** (line 21) — core helpers (`emacs-path`, `lookup-password`, window config stack)
2. **Environment** (line 54) — load-path, `use-package` bootstrap, Nix integration
3. **Settings** (line 110) — loads `settings.el`, handles data directory variants
4. **Libraries** (line 158) — deferred utility library declarations (dash, s, f, etc.)
5. **Keymaps** (line 218) — prefix keymap definitions
6. **Packages** (line 245) — the bulk of the file; ~443 `use-package` declarations, alphabetical
7. **Layout** (line 3797) — frame/window sizing functions (`emacs-min`, `emacs-max`)
8. **Finalization** (line 3862) — startup timing, `startup` interactive function

---

## The `lisp/` Directory

Custom and vendored Elisp files:

**Submodules** (declared in `.gitmodules`):
- `lisp/use-package` — the `use-package` macro itself
- `lisp/alert` — notification library
- `lisp/async` — async processing
- `lisp/initsplit` — split custom settings across files
- `lisp/haskell-config` — Haskell IDE support
- `lisp/git-annex` — git-annex integration
- `lisp/erc-yank` — ERC paste helper
- `lisp/chess` — chess client
- `lisp/git-undo` — undo git changes
- `lisp/nix-update` — Nix expression updater
- `lisp/emacs-pl` — Perl integration

**Personal Elisp files** in `lisp/`:
- `personal.el` — miscellaneous personal commands
- `org-smart-capture.el` — smart Org capture
- `org-balance.el` — Org workload balancing
- `paredit-ext.el` — ParEdit extensions
- `erc-alert.el`, `erc-macros.el` — ERC enhancements
- `esh-toggle.el`, `sh-toggle.el` — shell/eshell toggles
- `persian-johnw.el` — Persian language input
- `prover.el` — proof assistant utilities
- `coq-lookup.el`, `merlin.el` — Coq/OCaml support

---

## The `snippets/` Directory

YASnippet snippets, organized by major mode directory:

- `snippets/org-mode/` — Org capture/agenda templates
- `snippets/latex-mode/` — LaTeX fragments
- `snippets/c-mode/`, `snippets/c++-mode/`, `snippets/cc-mode/`
- `snippets/haskell-mode/`
- `snippets/python-mode/`
- `snippets/emacs-lisp-mode/`
- `snippets/coq-mode/`
- `snippets/nix-mode/`
- And more...

---

## Build System

```bash
make          # Byte-compile init.el, dot-org.el, dot-gnus.el + all lisp/
make compile  # Byte-compile all .el files under lisp/
make clean    # Remove all .elc files
make speed    # Benchmark startup time
```

The Makefile discovers subdirectories under `lisp/` automatically (excluding
`.git`, `doc`, `test`, `tests`, `obsolete`).

**Ignored build artifacts** (see `.gitignore`):
- `*.elc` — byte-compiled files
- `elpa/` — package.el packages (not used; managed by Nix)
- `data*/` — runtime data directories
- `var/`, `eshell/`, `server/`, `logs/` — runtime state

---

## Settings Files

`settings.el` and `*-settings.el` files are generated by Emacs's `Custom`
system. **Do not hand-edit** the `custom-set-variables` block unless you
understand exactly what you are doing. Changes made via `M-x customize` will
be written back here automatically.

The exception is `abbrevs.el`, which is hand-maintained and has a file-local
variable that runs `check-parens` and `quietly-read-abbrev-file` on save.

---

## Org-mode Configuration

`dot-org.el` is a standalone file loaded directly by Org-mode (not through
`init.el`). It:
- Loads `org-settings.el` for custom variables
- Defines custom agenda views (hotlist, projects, next actions, etc.)
- Configures capture templates, refile targets, TODO keywords
- Sets agenda files: `~/Documents/tasks/todo.txt`, `Bahai.txt`, `BAE.txt`

Custom agenda commands of note:
- `h` — Current Hotlist (HOT projects)
- `H` — Non-Hot Projects
- `n` — Project Next Actions
- `A` — Priority #A tasks
- `b` — Priority #A and #B tasks
- `r` — Uncategorized inbox items
- `w` — Waiting/delegated tasks

---

## Gnus Configuration

`dot-gnus.el` is loaded directly by Gnus. It:
- Loads `gnus-settings.el` for custom variables
- Defines `switch-to-gnus` for jumping to Gnus from anywhere
- Configures IMAP/SMTP connections using `auth-source` / `pass`
- Integrates with `bbdb` for contact management

---

## Guidelines for AI Assistants

1. **Preserve `use-package` structure** — always use `use-package` for package
   configuration; do not use raw `require` + `setq` outside of it.

2. **Use `:defer t` by default** — only load packages immediately with
   `:demand t` when there is a clear reason.

3. **Never add `package-install` calls** — all packages are Nix-managed.

4. **Keep sections alphabetical** — the Packages section in `init.el` is
   alphabetically ordered by package name. Insert new `use-package` declarations
   in the correct alphabetical position.

5. **Do not modify `settings.el` or `*-settings.el` directly** — these are
   managed by Emacs Custom. If a variable needs changing, change it in the
   appropriate `:config` or `:init` block in `init.el`.

6. **Respect `eval-and-compile` boundaries** — code in `eval-and-compile`
   blocks is needed at byte-compile time. Don't move such code outside.

7. **Use `bind-key` not `global-set-key`** — the `bind-key` package (part of
   `use-package`) is used throughout and provides better introspection.

8. **Byte-compile to verify** — run `make` or `make compile` to confirm that
   changes byte-compile without warnings or errors.

9. **Test startup time** — use `make speed` to check that startup performance
   has not regressed after changes.

10. **The `lisp/` submodules are upstream packages** — do not modify code
    inside `lisp/alert`, `lisp/use-package`, etc. Those are external projects.
    Custom forks live as separate repos.
