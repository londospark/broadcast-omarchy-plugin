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
# Arch / CachyOS / Manjaro (AUR) — pick one of each pair
paru -S broadcast-ctl-bin broadcast-gui-bin   # pre-built, no compiling
# or
paru -S broadcast-ctl-git broadcast-gui-git   # always builds latest main

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
worth having for the standalone window. Note: `broadcast-bin` on the AUR,
without the `-ctl`/`-gui` in the name, is a different, unrelated project —
don't install that one expecting this.)

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

**NVIDIA Maxine** — optional, better quality on an RTX GPU (Turing/20-series
or newer). Every [broadcast release](https://github.com/londospark/broadcast/releases)
bundles the runtime for desktop RTX generations (Turing through Blackwell),
so most people don't need an NGC account at all:

```sh
curl -fsSL -o install-maxine-runtime.sh \
  https://raw.githubusercontent.com/londospark/broadcast/63fa9b060aef4cac9e38d70db926bd9899f67801/scripts/install-maxine-runtime.sh
bash install-maxine-runtime.sh
```

(Needs the `gh` CLI on PATH — the script pulls the runtime and plugin
straight from the latest `broadcast` release. The commit above is pinned
the same way as the NGC path below; worth a read before running, same as
any script off the internet.)

If your GPU predates Turing, or isn't a desktop/workstation RTX part the
bundle covers, build against NVIDIA's SDK directly with your own (free)
[NGC API key](https://org.ngc.nvidia.com/setup/api-key) instead:

```sh
git clone https://github.com/londospark/broadcast.git /tmp/broadcast
git -C /tmp/broadcast checkout --detach 63fa9b060aef4cac9e38d70db926bd9899f67801
NGC_API_KEY=your_key_here bash /tmp/broadcast/scripts/install-maxine-sdk.sh
```

The commit above is pinned to the exact `broadcast` snapshot this plugin was
last synced against — kept up to date automatically on every sync, so it
always matches what was actually reviewed rather than a moving branch. This
has to build against the SDK, so it needs the full `broadcast` repo cloned —
there's no standalone Maxine installer. You can delete `/tmp/broadcast` once
it finishes; either path installs the plugin to `~/.local/lib/ladspa/`.

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

## Removing

```sh
omarchy plugin remove io.github.londospark.broadcast
```

That only removes the bar widget. To undo the rest of the setup:

```sh
# Drop the filter chain configs and restart PipeWire
rm ~/.config/pipewire/pipewire.conf.d/50-{deepfilter,maxine}-{input,output}.conf \
   ~/.config/pipewire/pipewire.conf.d/50-broadcast-defaults.conf
systemctl --user restart pipewire pipewire-pulse wireplumber

# Uninstall the CLI/GUI, matching however you installed them in step 2
paru -R broadcast-ctl-bin broadcast-gui-bin           # AUR (or -git)
sudo dpkg -r broadcast-ctl broadcast-gui              # .deb
sudo rpm -e broadcast-ctl broadcast-gui               # .rpm
rm ~/.local/bin/broadcast-ctl ~/.local/bin/broadcast-gui   # manual binaries

# Uninstall a backend, e.g. DeepFilterNet on Arch
paru -R libdeep_filter_ladspa-git
# Maxine's SDK lives under ~/.local/share/nvidia-maxine-sdk and the
# built plugin under ~/.local/lib/ladspa/ — rm -rf both if you installed it
```

## License and dependencies

The plugin itself is GPL-3.0-or-later, same as the main
[broadcast](https://github.com/londospark/broadcast) repo it drives. It runs
unsandboxed inside Omarchy's shell process and shells out to `broadcast-ctl`
(also GPL-3.0-or-later) — no user configuration is touched until you
explicitly run `broadcast-ctl install-config --apply` yourself in step 4.

`broadcast-ctl` in turn depends on PipeWire and one of two LADSPA noise-
suppression backends: [DeepFilterNet](https://github.com/Rikorose/DeepFilterNet)
(MIT/Apache-2.0) or NVIDIA's proprietary [Maxine Audio Effects SDK](https://developer.nvidia.com/maxine) —
see step 3. The Maxine runtime bundled in `broadcast` releases is
redistributed under [NVIDIA's Maxine SDK license](https://developer.nvidia.com/downloads/maxine-sdk-license),
whose distribution supplement permits this for any SDK portion other than
its audio/video data samples; NVIDIA's own EULA still applies to that
runtime once installed.
