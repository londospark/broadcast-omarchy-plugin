# Broadcast — Omarchy plugin

Bar widget for [broadcast](https://github.com/londospark/broadcast), AI noise
suppression for PipeWire. Shows filter health at a glance and gives you a
panel to switch backends (DeepFilterNet / NVIDIA Maxine), route individual
apps and windows through the filter, and manage input/output devices —
without leaving the bar.

This repo is just the plugin (QML + manifest) — a UI shell around the
`broadcast-ctl` command. It's mirrored automatically from
[londospark/broadcast](https://github.com/londospark/broadcast)'s
`omarchy-plugin/` folder whenever that changes, so it's always in sync with
the engine, but it can't install that engine for you: Omarchy's plugin
mechanism only ever clones QML files, it has no install-script hook. You
need all four steps below, in order, for the bar icon to do anything.

## 1. Install the plugin

```sh
omarchy plugin add https://github.com/londospark/broadcast-omarchy-plugin.git --enable
```

The bar icon will appear now, but it'll show red/unavailable until the rest
of this is done.

## 2. Install broadcast-ctl (and optionally broadcast-gui)

Pick whichever matches your distro:

```sh
# Arch / CachyOS / Manjaro (AUR)
paru -S broadcast-bin        # pre-built binaries
# or: paru -S broadcast-git  # build from source

# Ubuntu / Debian / Mint / Pop!_OS
gh release download --repo londospark/broadcast -p '*.deb' --dir /tmp/
sudo dpkg -i /tmp/broadcast-ctl_*.deb /tmp/broadcast-gui_*.deb
sudo apt-get install -f     # resolve any missing dependencies

# Fedora / openSUSE
gh release download --repo londospark/broadcast -p '*.rpm' --dir /tmp/
sudo rpm -i /tmp/broadcast-ctl-*.rpm /tmp/broadcast-gui-*.rpm

# Any distro, no package manager
gh release download --repo londospark/broadcast -p 'broadcast-ctl' -p 'broadcast-gui' --dir ~/.local/bin/
chmod +x ~/.local/bin/broadcast-ctl ~/.local/bin/broadcast-gui
```

(`broadcast-gui` is optional — the bar panel covers the same controls — but
worth having for the standalone window.)

## 3. Install a noise-suppression backend

The widget needs at least one LADSPA backend loaded to do anything.

**DeepFilterNet** — free, CPU-based, works on any hardware:

```sh
# Arch / CachyOS
paru -S libdeep_filter_ladspa-git ladspa
```

On other distros there's no package yet — build the LADSPA plugin from
[DeepFilterNet](https://github.com/Rikorose/DeepFilterNet)'s own instructions
and make sure `ladspa` (the LADSPA SDK/host tools) is installed too.

**NVIDIA Maxine** — optional, better quality on an RTX GPU, needs a (free)
[NGC API key](https://org.ngc.nvidia.com/setup/api-key):

```sh
git clone https://github.com/londospark/broadcast.git /tmp/broadcast
NGC_API_KEY=your_key_here bash /tmp/broadcast/scripts/install-maxine-sdk.sh
```

This has to build against the SDK, so it needs the full `broadcast` repo
cloned — there's no standalone Maxine installer. You can delete `/tmp/broadcast`
once it finishes; it installs the built plugin to `~/.local/lib/ladspa/`.

## 4. Write and load the PipeWire filter chain config

```sh
broadcast-ctl install-config --apply
```

This writes the filter chain config for whichever backend you just installed
and restarts PipeWire, PipeWire-Pulse, and WirePlumber to pick it up.

## 5. Verify

```sh
broadcast-ctl status
```

Should report `Filters: loaded`. If it does, the bar icon should already be
showing healthy — click it to toggle filtering on and route individual apps.

## Updating the plugin itself

```sh
omarchy plugin update io.github.londospark.broadcast
```

(This only updates the bar widget. Update `broadcast-ctl`/`broadcast-gui`
the same way you installed them in step 2.)

## License

GPL-3.0-or-later, same as the main [broadcast](https://github.com/londospark/broadcast) repo.
