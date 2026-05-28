# sddm-adart-matugener

<img width="2030" height="1200" alt="image" src="https://github.com/user-attachments/assets/24bd99f1-ce8d-4dd4-955a-ac8d85c18f25" />



A Matugen-wired SDDM theme based totally, fully, and with both hands on
[`sddm-astronaut-theme`](https://github.com/Keyitdev/sddm-astronaut-theme).

The clock, login layout, input styling, icons, fonts, system buttons, virtual keyboard
handling, and general “yes, this looks like an actual login screen” part come from
Astronaut. I added ADArt-specific wiring around it:

- Matugen-generated `theme.conf`
- mutable background handling
- a runtime directory outside `/usr/share`
- install and preview scripts
- a default ADArt background: `theme/Backgrounds/adart-woman-reading.png`

In other words: this is Astronaut with plumbing. Glamorous, like most plumbing.

> [!IMPORTANT]
> ## Credit
>
> This project is derived from `sddm-astronaut-theme`.
> I keep the upstream license and credit intact. I do not pretend the pretty QML grew here
> naturally after rainfall.

## Why

The original Astronaut theme already has the thing I want:

- a big clock
- nice date formatting
- username/password inputs with icons
- session controls
- system buttons
- virtual keyboard support
- a good full-screen composition

So this project does **not** try to reimplement that. I tried. It worked like a
rectangle wearing a helmet.

Instead, this project keeps the Astronaut layout and changes how the mutable parts
are managed.

## What this project changes

Astronaut expects this in `theme.conf`:

```ini
Background="Backgrounds/astronaut.png"
```

and the path must be relative to the theme directory.

I keep that rule and use:

```ini
Background="Backgrounds/adart-current.png"
```

`adart-current.png` is a symlink:

```text
/usr/share/sddm/themes/sddm-adart-matugener/Backgrounds/adart-current.png
  -> /usr/local/share/sddm-adart-matugener/background.png
```

So SDDM sees a relative theme background, while my wallpaper/Matugen scripts update
the real mutable file outside `/usr/share`.

## Directory layout

Repository:

```text
sddm-adart-matugener/
├── matugen/
│   ├── matugen.config.toml
│   └── sddm-adart-matugener-theme.conf
├── scripts/
│   ├── install
│   └── preview-theme
└── theme/
    ├── Assets/
    ├── Backgrounds/
    │   ├── adart-current.png -> /usr/local/share/sddm-adart-matugener/background.png
    │   └── adart-woman-reading.png
    ├── Components/
    ├── Fonts/
    ├── LICENSE
    ├── Main.qml
    ├── metadata.desktop
    └── theme.conf -> /usr/local/share/sddm-adart-matugener/theme.conf
```

Installed static theme:

```text
/usr/share/sddm/themes/sddm-adart-matugener/
```

Mutable runtime files:

```text
/usr/local/share/sddm-adart-matugener/
├── background.png
└── theme.conf
```

## Install

From the repository root:

```bash
chmod +x scripts/install scripts/preview-theme
./scripts/install
```

The install script does these things:

1. creates the runtime directory:

   ```text
   /usr/local/share/sddm-adart-matugener/
   ```

2. copies the default background:

   ```text
   theme/Backgrounds/adart-woman-reading.png
   ```

   to:

   ```text
   /usr/local/share/sddm-adart-matugener/background.png
   ```

3. creates an initial fallback `theme.conf` if one does not exist yet:

   ```text
   /usr/local/share/sddm-adart-matugener/theme.conf
   ```

4. installs the static theme to:

   ```text
   /usr/share/sddm/themes/sddm-adart-matugener
   ```

5. ensures the installed symlinks exist:

   ```text
   Backgrounds/adart-current.png -> /usr/local/share/sddm-adart-matugener/background.png
   theme.conf -> /usr/local/share/sddm-adart-matugener/theme.conf
   ```

6. enables the SDDM theme by writing:

   ```text
   /etc/sddm.conf.d/20-sddm-adart-matugener.conf
   ```

With:

```ini
[Theme]
Current=sddm-adart-matugener
```

## Install without enabling

```bash
./scripts/install --no-enable
```

## Force runtime files to be overwritten

Normally, `install` keeps existing runtime files so Matugen does not get stomped on
like a small, colorful bug.

To overwrite them:

```bash
./scripts/install --force-runtime
```

## Preview

```bash
./scripts/preview-theme
```

or directly:

```bash
sddm-greeter-qt6 --test-mode --theme /usr/share/sddm/themes/sddm-adart-matugener
```

The preview can show this warning:

```text
Socket error: "QLocalSocket::connectToServer: Invalid name"
```

In test mode, that is usually harmless. The greeter is not running inside a real
SDDM login session, so some socket/session parts are fake. The software is surprised
by the conditions it agreed to run under. Classic.

## Matugen

The Matugen template should generate Astronaut-compatible `theme.conf` keys.

Example Matugen config:

```toml
[templates.sddm-adart-matugener]
input_path = '~/.config/matugen/templates/sddm-adart-matugener-theme.conf'
output_path = '/usr/local/share/sddm-adart-matugener/theme.conf'
post_hook = ''
```

If Matugen cannot write directly to `/usr/local/share/sddm-adart-matugener`, make sure
the directory is user-owned:

```bash
sudo install -d -m 0755 /usr/local/share/sddm-adart-matugener
sudo chown "$USER:$USER" /usr/local/share/sddm-adart-matugener
```

## Background updates

The runtime background is:

```text
/usr/local/share/sddm-adart-matugener/background.png
```

My wallpaper script can update it by copying the generated Hyprlock background there:

```bash
install -m 0644 ~/.cache/hyprlock-current.png \
  /usr/local/share/sddm-adart-matugener/background.png
```

The theme reads it through the relative symlink:

```text
Backgrounds/adart-current.png
```

because Astronaut expects a relative background path. I obey. Not happily, but I
obey.

## Useful checks

Check SDDM theme config:

```bash
cat /etc/sddm.conf.d/20-sddm-adart-matugener.conf
```

Check installed theme symlinks:

```bash
ls -l /usr/share/sddm/themes/sddm-adart-matugener/theme.conf
ls -l /usr/share/sddm/themes/sddm-adart-matugener/Backgrounds/adart-current.png
```

Check runtime files:

```bash
ls -l /usr/local/share/sddm-adart-matugener/
```

Preview:

```bash
scripts/preview-theme
```

## License and attribution

This project is based on `sddm-astronaut-theme`, and I want that to be obvious rather than hidden in the basement.

I keep the upstream license file with the theme and preserve attribution in this README.
The ADArt-specific parts are the Matugen/runtime wiring and helper scripts.

## Name

`sddm-adart-matugener`

Because “Matugen-powered SDDM Astronaut derivative with mutable background symlink
plumbing” would be more accurate, but less merciful.

