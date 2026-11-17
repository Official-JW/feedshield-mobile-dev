# Feed Shield mobile

Phone-site shell for Feed Shield. Version 0.1.8.

This is an unwired dev preview. It has no public host, and it must not be published to the marketing domain or copied into `website/`.

The shell is installable to a phone home screen (PWA: manifest, icons, and a service worker via `vite-plugin-pwa`). It does not compute feeds and speeds yet.

The shell opens behind a 4-digit access-code gate. The client posts the entered code to `POST /check` and never fetches a code list. On this box, `vite` and `vite preview` mock that same contract from `../ACCESS_CODE.txt` (chmod 600, not part of the client bundle). The live tester uses a Cloudflare Worker at `VITE_GATE_WORKER_URL`. The Worker compares against the `ACCESS_CODES` secret and never returns the list.


## What is wired

- Scan, Add, and Library screens with bottom tab navigation
- Scan (v0.1.5): live camera (getUserMedia) with a file-input fallback, downscaled to 1600 px JPEG and posted to the Worker `POST /scan` with the session code in `X-Access-Code`. The Worker reads it with two Workers AI vision models, matches R1-R5 (spec 18) against a Worker-only catalog index (identification fields only), and drops the photo. The phone shows a confirm card (Confirm, then Save to library) or up to 5 candidates, plus Type SKU and Build params. Server down shows "Scanning is offline".
- v0.1.6: spec 18 v1.1 R4 (a one-character neighbor blocks only if it also fits what was read; with neighbors, both readers must agree on the EDP at the spec 20 L4 bar), an overall wrong-code cap across all IPs that pauses the gate, units on the "Read from photo" rows, coating codes spelled out (UN shows as Uncoated), and "Check catalog" for typed SKUs (Worker `POST /match`, user taps the row to confirm).
- v0.1.7: overall cap set by Direction to 20 wrong codes in 15 minutes, which pauses the gate for 60 minutes; sign-in pause message updated.
- v0.1.8: one-question card. When the candidates that fit the read differ in only one field, the card asks about that field (for example "Square, or 0.010 in corner radius?"); the answer opens the normal confirm card, with a "Show all candidates" link. TF: `node --experimental-strip-types scripts/tf-one-question.mjs`.
- Type SKU and Build params entry, with validation
- Tool library in IndexedDB (`idb`), with view, delete, Export JSON, and Import JSON (schema version 1, merge by id)

## Knowledge-base pickers

Maker, workpiece material, coating, and tool material lists are generated from the Feed Shield knowledge base. They are not hand-typed.

```
npm run generate-kb
```

writes `src/kb/lists.json` and records the source file for each list. Nickel superalloy grades named in KB19 maker rows (`rows[].material_row_as_printed`) are merged into that picker group. 03 and 07 were not in `/workspace/knowledge-base/` on this box; the script looks in known mirrors and notes the path it used. 07 was read but has no picker lists of its own.

17-4 PH uses the mill-typical HRC states from KB 19. The condition picker shows the assumed HRC, and the HRC field is editable.

## Engine adapter

The engine core is not imported in this shell. Recommendations go through a typed adapter:

- Interface: `EngineAdapter` in `src/engine/adapter.ts`
- Method: `recs(input): Promise<RecsResult>`
- Default implementation: `NotConnectedEngine`, which returns `status: "not-connected"` and no numbers

To plug in the real engine later, implement `EngineAdapter` and replace `defaultEngine` in `src/App.tsx`. Do not hard-code SFM, IPT, or any other cutting value. If the adapter is not connected, the recs card shows "Engine not connected" and dashes. Chart leads; the engine column is labeled exactly `Feed Shield - F/S`. Units belong in the field label column.

The later `packages/engine` move is a separate, byte-identical land. This shell does not perform that move.

## Scripts

```
npm install
npm run generate-kb
npm run typecheck
npm run build
npm run preview
```

`npm run build` typechecks and then builds. Preview is local only.

## Layout

Phone-first at 390px width. Tap targets are at least 44px. UI labels are plain ASCII.
