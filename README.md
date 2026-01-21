
# OrpheusDL (FZF Fork)

A modified version of [OrpheusDL](https://github.com/OrfiTeam/OrpheusDL) integrated with **fzf** for a rapid, interactive command-line experience. Designed to be driven by the **`tdl`** smart wrapper.

### ✨ Key Features

* **Interactive UI:** Replaces the standard selection menu with a searchable, sortable **fzf** table.
* **Rich Previews:** "Summary Card" view in the preview pane shows full details (Artist, Year, Quality, Explicit status) without cluttering the main list.
* **Context Aware:** Automatically adjusts column layouts for Tracks, Albums, or Artists.
* **Self-Healing:** The companion shell script automatically detects broken Python environments and rebuilds them on the fly.

---

### 📦 Requirements

* **Python 3.x**
* **[fzf](https://github.com/junegunn/fzf)** (Required for the interactive menu)
* **Fish Shell** (For the `tdl` wrapper)

### 🛠️ Setup

* **Clone this Repository** (or replace your existing `orpheus.py` with the one provided here).
or
* **Install the Wrapper:** Copy the Tidal fish wrapper `tdl` function below into your Fish configuration (usually `~/.config/fish/functions/tdl.fish`).

> **Note:** The `tdl` wrapper manages the installation at `~/Music/OrpheusDL` by default. It will automatically clone the repository and the Tidal module if they are missing, and create the venv there.

---

### 🚀 Usage

The **`tdl`** command is your single point of entry. It handles activation, execution, and updating.

#### 1. Search (Smart Detection)

Search for tracks by default, or specify a type.

```fish
# Default (Track Search)
tdl "call me karizma"

# Specific Types
tdl album "pink floyd"
tdl artist "the fray"
tdl playlist "lofi hip hop"

```

#### 2. Direct Download

Paste a Tidal URL to download immediately.

```fish
tdl "https://tidal.com/browse/track/123456"

```

#### 3. Updates

Updates the core repository, the Tidal module, and Python dependencies.

```fish
tdl update

```

---

### ⌨️ Controls (FZF)

Once the search results appear:

* **Type** to filter results instantly.
* **Up/Down** to navigate.
* **Enter** to select and download.
* **ESC** to cancel.

**Column Legend:**

* `#`: Index
* `TRACK`: Track Title (Truncated)
* `YEAR`: Release Year
* `LENGTH`: Duration (MM:SS)
* `[E]`: Explicit Tag
* `QUAL`: Quality (HiFi, Mast, DA=Dolby Atmos)

---

### 🐟 The Wrapper Script (`tdl`)

```fish
function tdl --description "OrpheusDL Tidal Wrapper"
    set -l tdl_base "$HOME/Music"
    set -l tdl_dir "$tdl_base/OrpheusDL"
    set -l venv_dir "$tdl_dir/.venv"
    set -l tidal_mod_dir "$tdl_dir/modules/tidal"
    set -l py "$venv_dir/bin/python"

    # --- Phase 1: Main Repo Check ---
    if not test -d "$tdl_dir"
        echo "OrpheusDL missing. Cloning..."
        mkdir -p "$tdl_base"
        git clone https://github.com/OrfiTeam/OrpheusDL.git "$tdl_dir"
    end

    # --- Phase 2: Venv Check ---
    if not test -f "$py"
        echo "Venv broken or missing. Rebuilding..."
        rm -rf "$venv_dir"
        python -m venv "$venv_dir"
        "$venv_dir/bin/pip" install -U pip
        "$venv_dir/bin/pip" install -r "$tdl_dir/requirements.txt"
    end

    # --- Phase 3: Tidal Module Check ---
    if not test -d "$tidal_mod_dir"
        echo "Tidal module missing. Installing..."
        builtin cd "$tdl_dir"
        git clone --recurse-submodules https://github.com/Dniel97/orpheusdl-tidal.git "modules/tidal"
        # Force init settings
        "$py" orpheus.py --help > /dev/null 2>&1
    end

    # --- Phase 4: Execution Logic ---
    if test (count $argv) -eq 0
        echo "Usage: tdl [url | update | (album/artist/track) search_term]"
        return 1
    end
    
    builtin cd "$tdl_dir"
    set -l first "$argv[1]"

    # UPDATE
    if test "$first" = "update"
        echo "Updating OrpheusDL and Tidal..."
        git pull
        git -C "modules/tidal" pull
        "$venv_dir/bin/pip" install -r requirements.txt
        echo "Done."
        return 0
    end

    # URL
    if string match -q "http*" "$first"
        "$py" orpheus.py "$argv"
        return 0
    end

    # SEARCH
    if contains "$first" album artist playlist track
        set -l mode $argv[1]
        set -l query $argv[2..-1]
        "$py" orpheus.py search tidal "$mode" "$query"
    else
        # Default to track search
        set -l query $argv
        "$py" orpheus.py search tidal track "$query"
    end
end

```

