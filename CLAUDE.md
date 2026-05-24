# modem-sim

Browser-based dialup modem handshake simulator. The entire app is a single static file: `public/index.html` (HTML + CSS + inline JS, no build step). Served as static files; `bin/deploy` publishes `public/`.

## Keep the README in sync

`README.md` is the reference doc for the handshake phases, line impairments, protocol specs, and accuracy notes. When you change simulation *behavior* — phase logic, the channel model, impairments, rate negotiation, or the accuracy caveats — update the matching section of `README.md` in the same change. The reference-data tables (frequencies, ITU standards) rarely change; behavior descriptions do.
