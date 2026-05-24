# 📝 Text Editors

> A **text editor** is one of the most essential tools on any Linux system. Whether editing config files, writing scripts, or developing software, knowing at least one terminal-based editor is a critical skill — because GUI editors aren't always available, especially on servers.

---

## 1. Overview of Linux Text Editors

### 1.1 Categories

| Category | Examples | Best For |
|---|---|---|
| **Terminal / CLI** | vi, vim, nano, emacs, ed | Servers, SSH, scripting |
| **GUI** | gedit, kate, mousepad, xed | Desktop environments |
| **Programmer / IDE** | VS Code, Sublime Text, Neovim | Development workflows |

### 1.2 Which Editor to Learn?

| Editor | Learning Curve | Power | Availability |
|---|---|---|---|
| **nano** | Easy | Basic | Most Linux distros |
| **vi/vim** | Steep | Very High | Every Linux/Unix system |
| **emacs** | Steep | Very High | Widely available |
| **Neovim** | Steep | Very High | Modern vim replacement |
| **gedit** | Easy | Moderate | GNOME desktops |

!!! tip "Minimum Requirement"
    Every Linux user should know basic `vi` or `vim` — it's on every Unix/Linux system, including minimal installs, containers, and rescue environments. `nano` is a great starting point for beginners.

---

## 2. vi and vim

### 2.1 History and Variants

| Editor | Description |
|---|---|
| **ed** | Original Unix line editor (1969) |
| **ex** | Extended ed; line-oriented |
| **vi** | Visual Interface built on ex (1976) |
| **vim** | Vi IMproved — the modern standard (1991) |
| **neovim** | Modern vim rewrite with Lua config (2014) |
| **gvim** | GUI version of vim |

```bash
which vi vim nvim          # Check what's installed
vi --version               # Version info
vim --version              # Full feature list
nvim --version             # Neovim version
```

### 2.2 Opening Files

```bash
vim file.txt               # Open or create file
vim +10 file.txt           # Open at line 10
vim +/pattern file.txt     # Open at first match of pattern
vim file1 file2 file3      # Open multiple files
vim -R file.txt            # Read-only mode
vim -d file1 file2         # Diff mode (vimdiff)
vimdiff file1 file2        # Side-by-side diff
view file.txt              # Read-only vim
```

### 2.3 The Three Modes

Vim is a **modal editor** — keys do different things depending on the active mode.

```
┌─────────────────────────────────────────────────────┐
│                    NORMAL MODE                       │
│           (default — navigation & commands)          │
│                                                      │
│   Press i/a/o → INSERT      Press : → COMMAND-LINE  │
│   Press v/V/Ctrl+V → VISUAL                         │
└─────────────────────────────────────────────────────┘

  INSERT MODE          VISUAL MODE        COMMAND-LINE MODE
  (typing text)        (selecting text)   (: commands)
  Press Esc to         Press Esc to       Press Enter to
  return to Normal     return to Normal   execute, Esc to cancel
```

| Mode | Enter With | Purpose |
|---|---|---|
| **Normal** | `Esc` (always returns here) | Navigation, commands |
| **Insert** | `i`, `a`, `o`, `I`, `A`, `O` | Typing and editing text |
| **Visual** | `v`, `V`, `Ctrl+V` | Selecting text |
| **Command-line** | `:` | Save, quit, search, settings |
| **Replace** | `R` | Overwrite text |

### 2.4 Entering and Exiting Insert Mode

| Key | Action |
|---|---|
| `i` | Insert before cursor |
| `I` | Insert at beginning of line |
| `a` | Append after cursor |
| `A` | Append at end of line |
| `o` | Open new line below and insert |
| `O` | Open new line above and insert |
| `s` | Delete character and insert |
| `S` | Delete line and insert |
| `R` | Enter Replace mode |
| `Esc` | Return to Normal mode |

### 2.5 Saving and Quitting

```
:w              Write (save) the file
:w filename     Save as a new filename
:wq             Write and quit
:wq!            Force write and quit (override read-only)
:q              Quit (only if no unsaved changes)
:q!             Quit without saving (discard changes)
:x              Write only if changed, then quit (like :wq but smarter)
ZZ              Write and quit (Normal mode shortcut for :wq)
ZQ              Quit without saving (Normal mode shortcut for :q!)
:e!             Reload file from disk (discard changes)
:w !sudo tee %  Save file that requires root (forgot to open with sudo)
```

### 2.6 Navigation — Normal Mode

**Basic movement:**

| Key | Movement |
|---|---|
| `h` | Left one character |
| `l` | Right one character |
| `j` | Down one line |
| `k` | Up one line |
| `0` | Beginning of line |
| `^` | First non-whitespace of line |
| `$` | End of line |
| `w` | Next word (start) |
| `W` | Next WORD (whitespace-delimited) |
| `b` | Previous word (start) |
| `B` | Previous WORD |
| `e` | End of current/next word |
| `E` | End of current/next WORD |

**Screen movement:**

| Key | Movement |
|---|---|
| `gg` | Go to first line |
| `G` | Go to last line |
| `5G` or `:5` | Go to line 5 |
| `Ctrl+F` | Page forward (down) |
| `Ctrl+B` | Page backward (up) |
| `Ctrl+D` | Half page down |
| `Ctrl+U` | Half page up |
| `H` | Move to top of screen |
| `M` | Move to middle of screen |
| `L` | Move to bottom of screen |
| `zz` | Center screen on cursor |
| `zt` | Scroll cursor to top |
| `zb` | Scroll cursor to bottom |

**Character search on line:**

| Key | Action |
|---|---|
| `f{char}` | Move to next `char` on line |
| `F{char}` | Move to previous `char` on line |
| `t{char}` | Move to just before next `char` |
| `T{char}` | Move to just after previous `char` |
| `;` | Repeat last f/F/t/T forward |
| `,` | Repeat last f/F/t/T backward |

**Jumping:**

| Key | Action |
|---|---|
| `%` | Jump to matching bracket `()`, `[]`, `{}` |
| `*` | Search for word under cursor (forward) |
| `#` | Search for word under cursor (backward) |
| `''` | Jump back to last position |
| `Ctrl+O` | Jump to previous location in jump list |
| `Ctrl+I` | Jump to next location in jump list |

### 2.7 Editing — Normal Mode

**Deleting:**

| Key | Action |
|---|---|
| `x` | Delete character under cursor |
| `X` | Delete character before cursor |
| `dw` | Delete word forward |
| `db` | Delete word backward |
| `dd` | Delete entire line |
| `D` | Delete from cursor to end of line |
| `d0` | Delete from cursor to beginning of line |
| `d$` | Delete from cursor to end of line (same as D) |
| `5dd` | Delete 5 lines |
| `dG` | Delete from cursor to end of file |
| `dgg` | Delete from cursor to start of file |

**Changing (delete + enter insert):**

| Key | Action |
|---|---|
| `cw` | Change word |
| `cc` | Change entire line |
| `C` | Change from cursor to end of line |
| `c$` | Change from cursor to end of line |
| `ci"` | Change inside double quotes |
| `ci(` | Change inside parentheses |
| `ca{` | Change around curly braces (includes braces) |

**Copying (yanking) and Pasting:**

| Key | Action |
|---|---|
| `yy` | Yank (copy) current line |
| `yw` | Yank word |
| `y$` | Yank to end of line |
| `5yy` | Yank 5 lines |
| `yG` | Yank to end of file |
| `p` | Paste after cursor |
| `P` | Paste before cursor |

**Undo and Redo:**

| Key | Action |
|---|---|
| `u` | Undo last change |
| `U` | Undo all changes on current line |
| `Ctrl+R` | Redo |
| `.` | Repeat last change |

**Other editing:**

| Key | Action |
|---|---|
| `r{char}` | Replace single character |
| `~` | Toggle case of character |
| `g~w` | Toggle case of word |
| `gUw` | Uppercase word |
| `guw` | Lowercase word |
| `>>` | Indent line right |
| `<<` | Indent line left |
| `==` | Auto-indent line |
| `=G` | Auto-indent to end of file |
| `J` | Join current line with next |
| `gJ` | Join without adding space |

### 2.8 Search and Replace

**Searching:**

```
/pattern        Search forward for pattern
?pattern        Search backward for pattern
n               Next match (same direction)
N               Previous match (reverse direction)
*               Search for word under cursor (forward)
#               Search for word under cursor (backward)
:noh            Clear search highlighting
```

**Search and Replace (command-line mode):**

```
:s/old/new/         Replace first occurrence on current line
:s/old/new/g        Replace all on current line
:%s/old/new/g       Replace all in entire file
:%s/old/new/gc      Replace all with confirmation prompt
:%s/old/new/gi      Case-insensitive replace
:5,10s/old/new/g    Replace in lines 5 to 10
:'<,'>s/old/new/g   Replace in visual selection
```

**Pattern examples:**

```
:%s/\bword\b/new/g     Whole word match
:%s/^/    /g           Indent all lines by 4 spaces
:%s/\s\+$//            Remove trailing whitespace
:%s/\r//g              Remove Windows carriage returns (^M)
```

### 2.9 Visual Mode

| Key | Action |
|---|---|
| `v` | Character visual mode |
| `V` | Line visual mode |
| `Ctrl+V` | Block/column visual mode |
| `o` | Move to other end of selection |
| `d` | Delete selection |
| `y` | Yank (copy) selection |
| `c` | Change selection |
| `>` | Indent selection |
| `<` | Unindent selection |
| `~` | Toggle case of selection |
| `u` | Lowercase selection |
| `U` | Uppercase selection |
| `:` | Enter command on selection |

### 2.10 Multiple Files and Windows

```
:e filename         Open another file
:ls or :buffers     List open buffers
:bn                 Next buffer
:bp                 Previous buffer
:b3                 Switch to buffer 3
:bd                 Delete (close) current buffer

# Splits
:sp filename        Horizontal split
:vsp filename       Vertical split
Ctrl+W h/j/k/l      Move between splits
Ctrl+W =            Equalize split sizes
Ctrl+W _            Maximize current split height
Ctrl+W |            Maximize current split width
Ctrl+W q            Close split

# Tabs
:tabnew file        Open in new tab
:tabn               Next tab
:tabp               Previous tab
:tabc               Close tab
gt                  Next tab (normal mode)
gT                  Previous tab (normal mode)
```

### 2.11 Marks and Registers

**Marks:**

```
ma          Set mark 'a' at current position
'a          Jump to line of mark 'a'
`a          Jump to exact position of mark 'a'
''          Jump to position before last jump
:marks      List all marks
```

**Registers:**

```
"ayy        Yank line into register 'a'
"ap         Paste from register 'a'
:reg        Show all registers and their content
"+yy        Yank to system clipboard (if compiled with +clipboard)
"+p         Paste from system clipboard
"0          Contains last yank
"-          Small delete register
".          Last inserted text
"%          Current filename
```

### 2.12 Macros

```
qa          Start recording macro into register 'a'
q           Stop recording
@a          Play macro 'a'
@@          Repeat last macro
5@a         Play macro 'a' 5 times
```

### 2.13 Useful Command-Line Commands

```
:set number         Show line numbers
:set nonumber       Hide line numbers
:set relativenumber Relative line numbers
:set hlsearch       Highlight search matches
:set ignorecase     Case-insensitive search
:set smartcase      Smart case (case-sensitive if uppercase used)
:set autoindent     Auto-indent new lines
:set expandtab      Use spaces instead of tabs
:set tabstop=4      Tab width = 4 spaces
:set shiftwidth=4   Indent width = 4 spaces
:set wrap           Wrap long lines
:set nowrap         No line wrapping
:set syntax=python  Force syntax highlighting language
:set list           Show invisible characters
:set spell          Enable spell checking
:syntax on          Enable syntax highlighting
:syntax off         Disable syntax highlighting
:colorscheme desert Change color scheme
:colorscheme <Tab>  Tab-complete available schemes

:!command           Run a shell command
:r !command         Insert output of shell command
:r filename         Insert contents of file at cursor
:sort               Sort selected lines
:%!sort             Sort entire file using external sort
```

### 2.14 `.vimrc` Configuration

The `~/.vimrc` file is vim's personal configuration file, loaded at startup.

```vim
" ~/.vimrc — example configuration

" Basic settings
set nocompatible          " Use Vim features (not pure vi)
syntax on                 " Syntax highlighting
set number                " Line numbers
set relativenumber        " Relative line numbers
set cursorline            " Highlight current line
set wrap                  " Wrap long lines
set showcmd               " Show command in bottom bar
set showmatch             " Highlight matching brackets
set wildmenu              " Enhanced command completion

" Indentation
set autoindent
set expandtab             " Spaces instead of tabs
set tabstop=4
set shiftwidth=4
set softtabstop=4

" Search
set hlsearch              " Highlight matches
set incsearch             " Incremental search
set ignorecase
set smartcase

" File handling
set encoding=utf-8
set fileencoding=utf-8
set nobackup
set noswapfile

" UI
set laststatus=2          " Always show status line
set ruler                 " Show cursor position
set scrolloff=8           " Keep 8 lines above/below cursor

" Key mappings
nnoremap <leader>w :w<CR>          " Save with leader+w
nnoremap <C-h> <C-w>h              " Navigate splits with Ctrl+hjkl
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l

" The leader key (default is \)
let mapleader = ","
```

---

## 3. nano

### 3.1 Overview

`nano` is a simple, beginner-friendly terminal text editor. Its keyboard shortcuts are always displayed at the bottom of the screen, making it easy to discover commands without memorization.

```bash
nano file.txt              # Open or create file
nano +10 file.txt          # Open at line 10
nano -v file.txt           # Read-only (view) mode
nano -l file.txt           # Show line numbers
nano -m file.txt           # Enable mouse support
nano -B file.txt           # Create backup before editing
nano -i file.txt           # Auto-indent new lines
nano -w file.txt           # Disable line wrapping
```

### 3.2 nano Key Shortcuts

!!! info "Notation"
    `^` means `Ctrl`. `M-` means `Alt` (Meta key).

**File operations:**

| Shortcut | Action |
|---|---|
| `^S` | Save file |
| `^O` | Write Out (save as) — prompts for filename |
| `^X` | Exit (prompts to save if modified) |
| `^R` | Read/insert a file |
| `^T` | Invoke spell check (if available) |

**Navigation:**

| Shortcut | Action |
|---|---|
| `^F` / `→` | Forward one character |
| `^B` / `←` | Backward one character |
| `^N` / `↓` | Next line |
| `^P` / `↑` | Previous line |
| `^A` / `Home` | Beginning of line |
| `^E` / `End` | End of line |
| `^V` / `PgDn` | Next page |
| `^Y` / `PgUp` | Previous page |
| `M-\` | First line of file |
| `M-/` | Last line of file |
| `^_` | Go to specific line number |
| `M-G` | Go to specific line and column |

**Editing:**

| Shortcut | Action |
|---|---|
| `^D` | Delete character under cursor |
| `^H` / `Backspace` | Delete character before cursor |
| `M-D` | Delete word forward |
| `M-Backspace` | Delete word backward |
| `^K` | Cut (kill) current line |
| `M-6` | Copy current line |
| `^U` | Paste (uncut) |
| `M-U` | Undo |
| `M-E` | Redo |

**Search and Replace:**

| Shortcut | Action |
|---|---|
| `^W` | Search (Where is) |
| `M-W` | Search next (repeat) |
| `^\` | Search and Replace |
| `^C` | Cancel / current position info |

**Selection (marking):**

| Shortcut | Action |
|---|---|
| `M-A` | Set/unset mark (start selection) |
| `^K` | Cut marked region |
| `M-6` | Copy marked region |

**Display:**

| Shortcut | Action |
|---|---|
| `M-N` | Toggle line numbers |
| `M-P` | Toggle whitespace display |
| `M-X` | Toggle help bar |
| `^L` | Refresh / center screen |

### 3.3 nano Configuration (`~/.nanorc`)

```bash
# ~/.nanorc — example configuration

set linenumbers           # Show line numbers
set mouse                 # Enable mouse
set autoindent            # Auto-indent
set tabsize 4             # Tab width
set tabstospaces          # Convert tabs to spaces
set softwrap              # Soft line wrapping
set smooth                # Smooth scrolling
set historylog            # Save search history

# Syntax highlighting (include distro files)
include "/usr/share/nano/*.nanorc"
```

---

## 4. emacs

### 4.1 Overview

GNU Emacs is an extremely powerful, extensible editor — described as an "operating system disguised as a text editor." It uses **Emacs Lisp (Elisp)** for configuration and extensions.

```bash
emacs file.txt             # Open in GUI (if X available)
emacs -nw file.txt         # No Window — force terminal mode
emacs -nw -q file.txt      # Skip loading init file
emacs --batch              # Non-interactive (scripting)
```

### 4.2 Emacs Key Notation

| Notation | Meaning |
|---|---|
| `C-x` | Hold Ctrl, press x |
| `M-x` | Hold Alt (Meta), press x |
| `C-x C-s` | Ctrl+x, then Ctrl+s (two-chord sequence) |
| `SPC` | Spacebar |
| `RET` | Enter/Return key |

### 4.3 Essential Emacs Shortcuts

**File and session:**

| Shortcut | Action |
|---|---|
| `C-x C-f` | Open (find) file |
| `C-x C-s` | Save file |
| `C-x C-w` | Save as (write to new name) |
| `C-x C-c` | Quit emacs |
| `C-z` | Suspend emacs (resume with `fg`) |
| `C-x C-r` | Open file read-only |

**Navigation:**

| Shortcut | Action |
|---|---|
| `C-f` / `→` | Forward character |
| `C-b` / `←` | Backward character |
| `C-n` / `↓` | Next line |
| `C-p` / `↑` | Previous line |
| `C-a` / `Home` | Beginning of line |
| `C-e` / `End` | End of line |
| `M-f` | Forward word |
| `M-b` | Backward word |
| `C-v` | Scroll down (next page) |
| `M-v` | Scroll up (previous page) |
| `M-<` | Beginning of buffer |
| `M->` | End of buffer |
| `C-l` | Center screen on cursor |
| `M-g g` | Go to line number |

**Editing:**

| Shortcut | Action |
|---|---|
| `C-d` | Delete character forward |
| `DEL` | Delete character backward |
| `M-d` | Delete word forward |
| `M-DEL` | Delete word backward |
| `C-k` | Kill (cut) to end of line |
| `C-w` | Kill (cut) region |
| `M-w` | Copy region |
| `C-y` | Yank (paste) |
| `M-y` | Yank previous kill (cycle kill ring) |
| `C-/` or `C-_` | Undo |
| `C-x u` | Undo |
| `C-x C-u` | Uppercase region |
| `C-x C-l` | Lowercase region |

**Search:**

| Shortcut | Action |
|---|---|
| `C-s` | Incremental search forward |
| `C-r` | Incremental search backward |
| `M-%` | Query replace (search & replace) |
| `C-M-s` | Regex search forward |
| `C-M-%` | Regex replace |

**Windows and buffers:**

| Shortcut | Action |
|---|---|
| `C-x 2` | Split horizontally |
| `C-x 3` | Split vertically |
| `C-x 1` | Close other windows |
| `C-x 0` | Close this window |
| `C-x o` | Switch to other window |
| `C-x b` | Switch buffer |
| `C-x C-b` | List all buffers |
| `C-x k` | Kill (close) buffer |

**Mark and region:**

| Shortcut | Action |
|---|---|
| `C-SPC` | Set mark (start selection) |
| `C-x C-x` | Exchange point and mark |
| `M-h` | Mark paragraph |
| `C-x h` | Mark entire buffer |

**Commands:**

| Shortcut | Action |
|---|---|
| `M-x` | Execute named command |
| `M-x shell` | Open shell in emacs |
| `M-x term` | Full terminal emulator |
| `M-x compile` | Run compile command |
| `M-x replace-string` | Simple string replace |
| `C-g` | Cancel current command (like Esc in vim) |

### 4.4 emacs Configuration (`~/.emacs` or `~/.emacs.d/init.el`)

```elisp
;; ~/.emacs — basic configuration

;; UI
(setq inhibit-startup-message t)    ; No splash screen
(tool-bar-mode -1)                  ; No toolbar
(menu-bar-mode -1)                  ; No menu bar
(scroll-bar-mode -1)                ; No scrollbar
(global-linum-mode t)               ; Line numbers

;; Indentation
(setq-default indent-tabs-mode nil) ; Use spaces
(setq-default tab-width 4)
(setq indent-line-function 'insert-relative-tab)

;; Search
(setq case-fold-search nil)         ; Case-sensitive search

;; Backups
(setq make-backup-files nil)        ; No backup files
(setq auto-save-default nil)        ; No auto-save

;; Syntax highlighting
(global-font-lock-mode t)

;; Key bindings
(global-set-key (kbd "C-x C-b") 'ibuffer)   ; Better buffer list
```

---

## 5. ed — The Line Editor

`ed` is the original Unix text editor and the ancestor of `vi`, `sed`, and many others. It is still available on virtually every Unix/Linux system and is useful for scripting.

```bash
ed file.txt                # Open file in ed
```

### 5.1 ed Commands

```
p           Print current line
,p          Print all lines
1p          Print line 1
$p          Print last line
1,5p        Print lines 1 to 5

a           Append text after current line (end with . on own line)
i           Insert text before current line
c           Change current line
d           Delete current line
1,5d        Delete lines 1–5

/pattern/p  Print line matching pattern
,s/old/new/ Replace first on all lines
,s/old/new/g Replace all on all lines

w           Write (save) file
w file.txt  Write to filename
q           Quit
wq          Write and quit
Q           Quit without saving
```

---

## 6. Other Editors

### 6.1 gedit (GNOME)

The default GNOME text editor — clean, GUI-based, good for desktop users.

```bash
gedit file.txt             # Open file
gedit &                    # Open in background (keep terminal)
```

**Features:** Syntax highlighting, plugins, tabs, search/replace, undo history.

### 6.2 kate (KDE)

KDE's advanced text editor — powerful with IDE-like features.

```bash
kate file.txt &            # Open file
```

**Features:** Multi-document interface, terminal panel, split view, Vi input mode, sessions.

### 6.3 Neovim

A modern, community-driven rewrite of Vim with first-class Lua configuration, async plugins, and a built-in LSP client.

```bash
nvim file.txt              # Open file
nvim --headless            # Non-interactive (scripting)
```

Key improvements over vim: Lua config (`init.lua`), built-in LSP, Tree-sitter parsing, async everything, built-in terminal.

### 6.4 Comparison Table

| Feature | nano | vim | emacs | neovim |
|---|---|---|---|---|
| Learning curve | Easy | Steep | Steep | Steep |
| Modal editing | No | Yes | No | Yes |
| Extensibility | Low | High | Very High | Very High |
| Config language | nanorc | Vimscript | Elisp | Lua / Vimscript |
| Startup speed | Fast | Fast | Slow | Fast |
| Plugin ecosystem | Minimal | Large | Large | Large + modern |
| LSP support | No | Plugin | Plugin | Built-in |
| Available everywhere | Usually | Always | Often | Newer installs |

---

## 7. Editing Files Without an Editor

Sometimes you need to make quick edits from scripts or the command line without opening an interactive editor.

### 7.1 `sed` — Stream Editor

```bash
sed 's/old/new/' file.txt              # Replace first occurrence per line (stdout)
sed 's/old/new/g' file.txt             # Replace all occurrences
sed -i 's/old/new/g' file.txt          # Edit file in-place
sed -i.bak 's/old/new/g' file.txt      # In-place with backup
sed -n '5,10p' file.txt                # Print lines 5–10
sed '5,10d' file.txt                   # Delete lines 5–10
sed '/pattern/d' file.txt              # Delete lines matching pattern
sed -n '/pattern/p' file.txt           # Print only matching lines
sed '1i\first line' file.txt           # Insert line before line 1
sed '$a\last line' file.txt            # Append line after last line
```

### 7.2 `awk` for Inline Edits

```bash
awk '{gsub(/old/, "new"); print}' file.txt     # Replace all
awk 'NR==5{$0="replacement line"} {print}' file.txt  # Replace line 5
```

### 7.3 `printf` and `echo` Redirection

```bash
echo "new content" > file.txt          # Overwrite
echo "appended line" >> file.txt       # Append
printf "line1\nline2\n" > file.txt     # Multi-line write
```

### 7.4 `tee` for Simultaneous Write

```bash
echo "config=value" | tee config.txt          # Write and display
echo "config=value" | tee -a config.txt        # Append and display
echo "config=value" | sudo tee /etc/config.conf  # Write as root via pipe
```

---

## 8. Quick Reference Cheat Sheet

```bash
# ─── Open Files ──────────────────────────────────────────────────────
vim file.txt               # Open in vim
vim +10 file.txt           # Open at line 10
vim -R file.txt            # Read-only
nano file.txt              # Open in nano
emacs -nw file.txt         # Open in emacs (terminal)

# ─── vim: Save and Quit ──────────────────────────────────────────────
:w                         # Save
:q                         # Quit
:wq or ZZ                  # Save and quit
:q!  or ZQ                 # Quit without saving
:w !sudo tee %             # Save as root

# ─── vim: Navigation ─────────────────────────────────────────────────
gg / G                     # First / last line
:50                        # Go to line 50
Ctrl+F / Ctrl+B            # Page down / up
w / b                      # Next / previous word
0 / $                      # Start / end of line
%                          # Jump to matching bracket

# ─── vim: Editing ────────────────────────────────────────────────────
i / a / o                  # Insert before / after / new line below
dd / yy / p                # Delete / copy / paste line
dw / yw                    # Delete / copy word
u / Ctrl+R                 # Undo / redo
.                          # Repeat last change
>>  / <<                   # Indent / unindent

# ─── vim: Search and Replace ─────────────────────────────────────────
/pattern                   # Search forward
n / N                      # Next / previous match
:%s/old/new/g              # Replace all in file
:%s/old/new/gc             # Replace with confirmation

# ─── vim: Settings ───────────────────────────────────────────────────
:set number                # Show line numbers
:set expandtab ts=4 sw=4   # Spaces, width 4
:syntax on                 # Enable syntax highlight
:noh                       # Clear search highlight

# ─── nano Shortcuts ──────────────────────────────────────────────────
^S                         # Save
^O                         # Save as
^X                         # Exit
^W                         # Search
^\                         # Search and replace
^K                         # Cut line
^U                         # Paste
^_                         # Go to line
M-U / M-E                  # Undo / redo

# ─── emacs Shortcuts ─────────────────────────────────────────────────
C-x C-f                    # Open file
C-x C-s                    # Save
C-x C-c                    # Quit
C-s / C-r                  # Search forward / backward
M-%                        # Search and replace
C-k                        # Kill (cut) line
C-y                        # Yank (paste)
C-/                        # Undo
C-g                        # Cancel command
C-x 2 / C-x 3             # Split window horiz / vert

# ─── Scriptable Editing ──────────────────────────────────────────────
sed -i 's/old/new/g' file  # In-place replace
sed -n '5,10p' file        # Print lines 5–10
echo "text" >> file        # Append line
echo "text" | sudo tee /etc/file  # Write as root via pipe
```