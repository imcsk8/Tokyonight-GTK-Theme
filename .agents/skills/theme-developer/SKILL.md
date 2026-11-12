---
name: theme-developer
description: >-
  Specializes in GTK (GTK2/3/4), GNOME Shell, Cinnamon, and XFCE desktop theme development, compilation, version updates, panel styling, and release packaging.
  Use when building, editing SCSS/SASS stylesheets, updating theme compatibility for GNOME Shell versions (such as 40 through 50+), modifying top bar panel transparency, or packaging releases.
---

# GTK & GNOME Shell Theme Developer Skill

This skill provides guidelines and runbooks for developing, editing, compiling, updating, and releasing GTK and GNOME Shell themes.

## Core Concepts

GTK and GNOME Shell themes use SASS/SCSS source files compiled via `sassc`.
Theme packages usually include styles for:
- **GTK3** (`gtk-3.0/gtk.css`, `gtk-dark.css`)
- **GTK4** (`gtk-4.0/gtk.css`, `gtk-dark.css`, with libadwaita integration)
- **GTK2** (`gtk-2.0/gtkrc`)
- **GNOME Shell** (`gnome-shell/gnome-shell.css`)
- **Cinnamon, Metacity, XFWM4, Plank**

> [!NOTE]
> Theme specification is **optional**. If no specific theme name or variant is provided, use the theme present in the workspace or default to standard parameters.

---

## Workspace Structure & Key Files

A standard GTK theme repository follows this layout:
```text
themes/
├── build.sh                      # Script to compile SASS to CSS
├── install.sh                    # CLI installation script with option flags
├── gtkrc.sh                      # Generates GTK2 configuration files
├── release/
│   └── make-release.sh           # Packages theme variants into release tarballs (.tar.xz)
└── src/
    ├── main/                     # SASS entrypoints (gtk-3.0, gtk-4.0, gnome-shell, cinnamon)
    ├── sass/
    │   ├── _colors.scss          # Color definitions & functions
    │   ├── _tweaks.scss          # Feature toggles (compact, float, blackness, etc.)
    │   └── gnome-shell/          # GNOME Shell versioned widget stylesheets
    │       ├── _common.scss      # Main common imports
    │       ├── common/_panel.scss# Top bar panel styling (transparency, background, margins)
    │       └── _widgets-XX-0.scss# Version-specific widget definitions (e.g. 48-0, 50-0)
    └── assets/                   # SVG & PNG assets for controls, windows, and desktop
```

---

## Standard Development Workflows

### 1. Panel & Top Bar Customization
To modify GNOME Shell top bar panel behavior (such as background transparency):
- Open `src/sass/gnome-shell/common/_panel.scss`.
- Locate `#panel { background-color: ... }`.
- **To disable transparency for dark themes**: Replace `rgba($black, 0.85)` (or any `rgba` value with alpha < 1.0) with `$black` or `rgba($black, 1.0)`.

### 2. GNOME Shell Version Support (GNOME 50.1+ and Future Releases)
To ensure compatibility with GNOME 50.1 and future GNOME Shell releases:
1. **Create Version Widget File**: Create `src/sass/gnome-shell/_widgets-50-0.scss` importing necessary common and versioned widget components.
2. **Update Common Imports**: In `src/sass/gnome-shell/_common.scss`, set `@import 'widgets-50-0';` as default.
3. **Future-Proof Installer Logic**: In `install.sh`:
   - Parse major GNOME version: `SHELL_VERSION="$(gnome-shell --version | cut -d ' ' -f 3 | cut -d . -f -1 | sed 's/[^0-9].*//')"`.
   - Add condition for `SHELL_VERSION >= 50` -> `GS_VERSION="50-0"`.
   - Ensure fallback for missing `gnome-shell` defaults to `GS_VERSION="50-0"`.
   - In `gnome_shell_version()`, use wildcards/regex for version replacement:
     `sed -i "/\widgets/s/[0-9]\+-[0-9]\+/${GS_VERSION}/" ...`

### 3. Compiling SASS to CSS
- Ensure `sassc` is installed.
- Compile theme CSS files:
  ```bash
  cd themes && ./build.sh
  ```

### 4. Creating Release Tarballs
- Release script is located at `themes/release/make-release.sh`.
- Ensure script uses proper relative pathing (`cd "$THEME_DIR/.."`) and valid scheme tweak names (e.g., `storm`, `moon`).
- Run the release script:
  ```bash
  ./themes/release/make-release.sh
  ```
- Archives will be created in `themes/release/*.tar.xz`.

---

## Installation Guidelines

> [!IMPORTANT]
> **Always ask for approval before installing themes to the user's system.**

- **CLI Installation**:
  ```bash
  ./install.sh [OPTIONS]
  ```
- **Key Option Flags**:
  - `-n, --name [NAME]` *(Optional)*: Custom theme name.
  - `-d, --dest [DIR]` *(Optional)*: Destination directory (defaults to `~/.themes` or `~/.local/share/themes`).
  - `-t, --theme [VARIANT]` *(Optional)*: Accent colors (`default`, `purple`, `green`, `all`, etc.).
  - `-c, --color [VARIANT]` *(Optional)*: Color mode (`light`, `dark`).
  - `-l, --libadwaita` *(Optional)*: Symlinks `gtk-4.0` theme to `~/.config/gtk-4.0` for GTK4/libadwaita apps.
  - `--tweaks [TWEAKS]` *(Optional)*: Custom style variants (`moon`, `storm`, `black`, `float`, `outline`, `macos`).
