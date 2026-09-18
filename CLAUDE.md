# Nova Parcel — project notes for Claude

Unofficial Nova Poshta parcel tracker. Fork of FakerFox/nova-parcel, published at
github.com/oleksandrkodua/nova-parcel.

- macOS: native Swift/SwiftUI in `Sources/`, built by `build.sh` with plain `swiftc` (no Xcode project).
- Windows: Electron in `windows/`.
- `Resources/Bridge.js` is shared: it runs inside the official web cabinet page and returns
  a minimal parcel snapshot to both apps.

## Commands

- macOS tests: `./test.sh` (Swift core checks + Bridge checks)
- Windows tests: `cd windows && npm test`
- Type-check the whole macOS app: `xcrun swiftc -typecheck Sources/*.swift`.
  Always run it after touching Swift — `test.sh` compiles only `ParcelCore.swift`.
  In the Claude sandbox `xcrun` fails (cache write); call `/Library/Developer/CommandLineTools/usr/bin/swiftc -typecheck -swift-version 5 -sdk /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk -target arm64-apple-macosx14.0 Sources/*.swift` instead.
- Build: `./build.sh` → `dist/Nova Parcel.app` (already strips Finder xattrs before signing).
- Install the build (the user runs this, not you — the sandbox blocks /Applications):
  `./build.sh && { killall NovaParcel 2>/dev/null; sleep 1; true; } && rm -rf "/Applications/Nova Parcel.app" && cp -R "dist/Nova Parcel.app" /Applications/ && open "/Applications/Nova Parcel.app"`
- Sync diagnostics (macOS): `log show --predicate 'subsystem == "ua.local.novaparcel"' --last 1h --info`

## Rules

- Never commit or push. Show the diff and a short summary; the user commits from VS Code.
- User-facing strings are Ukrainian. Code comments are English, only where the "why" is non-obvious.
- UI changes: first a static HTML mockup in `mockups/` (current vs proposed, masked TTNs),
  wait for approval, then edit Swift.
- Bridge.js may return only a minimal snapshot. Diagnostics are counts only —
  never TTNs, names, phones, addresses or descriptions.
- The app is unofficial: style may follow novaposhta.ua, but no Nova Poshta logo, no
  "NovaPoshta" brand font, and the "НЕОФІЦІЙНИЙ ВІДЖЕТ" label stays visible.

## Invariants — do not revert

- Manual parcels (`isManual`) survive sign-in and account switch.
- Delivered parcels are capped at 50 per tab (`ParcelMerge.deliveredLimit`, `DELIVERED_LIMIT`).
- Bridge.js aborts on a list page only if *no* row has a number (`list.every`), so one
  unnumbered draft row can't break the whole sync.
- In demo mode only the settings are disabled; "Готово" (Esc) and "Завершити роботу" stay enabled.
- The app lives in the Dock (`.regular`, no `LSUIElement`); closing the window keeps it running.

## Open issue

A parcel in the "waybill created, not yet handed over" state (StatusCode 1) didn't appear,
while the phone app showed it; it appeared once it moved to StatusCode 5. The diagnostic
counters were added to find out whether the cabinet list omits such rows or they arrive
and get dropped. Check the sync log the next time a parcel is in that state.
