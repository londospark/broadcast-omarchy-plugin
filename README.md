# Broadcast — Omarchy plugin

Bar widget for [broadcast](https://github.com/londospark/broadcast), AI noise
suppression for PipeWire. Shows filter health at a glance and gives you a
panel to switch backends (DeepFilterNet / NVIDIA Maxine), route individual
apps and windows through the filter, and manage input/output devices —
without leaving the bar.

This repo is just the plugin (QML + manifest). The actual noise-suppression
engine (`broadcast-ctl`, `broadcast-gui`, and the DSP backends) lives in the
main [broadcast](https://github.com/londospark/broadcast) repo and is
mirrored here automatically whenever it changes.

## Install

```sh
omarchy plugin add https://github.com/londospark/broadcast-omarchy-plugin.git --enable
```

This installs the bar widget, but the widget only draws a UI around
`broadcast-ctl` — it needs that installed separately first. Grab a release
binary, `.deb`, or `.rpm` from the
[broadcast releases page](https://github.com/londospark/broadcast/releases),
or see the main repo's README for building from source.

## Updating

```sh
omarchy plugin update io.github.londospark.broadcast
```

## License

GPL-3.0-or-later, same as the main [broadcast](https://github.com/londospark/broadcast) repo.
