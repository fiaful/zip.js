# Review checklist

Each item below starts with a trigger that can be read from a diff. When reviewing a change, check
only the items whose trigger matches. An item says what to check and why it exists. "CI" marks the
checks a workflow already enforces, the others rely on the reviewer.

## Generated files

- **A file under `lib/` changes.** Run `npm run build-dev`, then `npm run build`, and commit what
  they regenerate. The tests run the committed inline worker files (`lib/core/web-worker-inline-*.js`)
  and not the sources directly: without `build-dev`, a green suite says nothing about a worker
  change. CI ("Verify build output is up to date").
- **`index.d.ts` changes**, doc comments included. Run `npm run build`, which regenerates
  `index.d.cts`, and `npm run build-doc-markdown`, which regenerates `docs/`. CI ("Verify build
  output is up to date", "Verify markdown doc is up to date"). The local tests do not see a stale
  `index.d.cts`.

## Workers

- **A property is added to, renamed in or removed from a message exchanged with a worker** (codec
  options, result fields, error fields). Its name is listed in `worker-message-property-names.js`,
  kept in alphabetical order, then `npm run build`. The main bundle and the worker bundle are
  minified separately, and an unlisted name is mangled differently on each side: the worker reads
  `undefined`. The tests show it in Deno, Bun and browsers only, never in Node.js, which runs the
  codecs on the main thread.

## Tests

- **A new test file.** It is registered in `tests/tests-data.js`, or in the `files` list of
  `tests/types/tsconfig.json` for a type test. `tests/README.md` gives the procedure. CI ("Verify
  the tests are registered").
- **A test instantiates the WebAssembly module.** It is registered with `features: ["wasmBuild"]`,
  or it fails on the old browsers of the test matrix that the other lanes do not cover.
- **A test fails now and then.** It is not retried, skipped or given a delay. The cause is found.

## Library code

- **A timer or a wait is added** (`setTimeout`, a message channel hop, "wait until idle"). Not in
  library code. When two cases can only be told apart by timing, the distinction is dropped and the
  simpler contract is documented.
- **Code shaped by a Node.js bug or a `Buffer` quirk.** Prefer a report upstream, or one
  normalization at the API boundary (wrap the input into a plain `Uint8Array` on entry), over
  defensive code spread through the library. An unavoidable workaround says which bug it works
  around, so that it can be removed.
- **The CRC-32 check of the reader is mentioned.** The option is `checkCrc32`. Its deprecated alias
  is not used in code, tests, documentation or examples.

## Code style

- **Any code change.** `npm run lint` clean. CI ("Verify no code issue").
- **A comment is added.** The code is written without comments, the names and the structure carry
  the meaning. A comment is kept only where the code cannot be made clear without it.
