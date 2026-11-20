# apidoc-core Wiki

`apidoc-core` is BasharSoft's **fork of the open-source apiDoc parser library** (version `0.8.3`, original author Peter Rottmann, MIT). It is the Node.js engine behind the `apidoc` command-line tool: it finds source files, extracts `@api...` comment blocks, and turns them into the JSON that the `apidoc` HTML template renders.

**How it serves Wuzzuf:** only indirectly. It is a dependency of the `apidoc` fork, which `wuzzuf-platform` runs at Docker build time to generate its old API docs (`public/docs`). Both of `wuzzuf-platform`'s lock files resolve `apidoc-core` to **this exact commit** (`27a8561`): `package-lock.json` from `BasharSoft/apidoc-core`, `yarn.lock` from the personal fork `SherifMedhat/apidoc-core`. Nothing else in wuzzuf-home uses it. `basharjobs` mentions apidoc-core only inside its own vendored copy of an older apidoc (`.build-wuzzuf/api_doc_source`).

No Wuzzuf-specific code was found: `package.json` still names the upstream repo and the parsers are the standard apiDoc set. Without git history, small changes from upstream 0.8.3 cannot be ruled out.

This wiki was written from the source code (`master` at `27a8561`). The existing README/docs weren't used. The [code map](code-map/index.md) was generated with graphify.

> **Read [Known issues](known-issues.md) first.** This is unmaintained 2017-era code pulled into a production image build from a git URL, partly through a personal GitHub account.

## What is inside

| Path | What |
|---|---|
| `lib/index.js` | Entry: `parse(options)`, spec version `0.3.0`, wires parsers, workers, filters, languages |
| `lib/parser.js` | Reads files (with `iconv-lite` for encodings), finds comment blocks per language, runs element parsers |
| `lib/parsers/` | One parser per tag: `@api`, `@apiDefine`, `@apiDeprecated`, `@apiDescription`, `@apiError(Example)`, `@apiExample`, `@apiGroup`, `@apiHeader(Example)`, `@apiName`, `@apiParam(Example)`, `@apiPermission`, `@apiSampleRequest`, `@apiSuccess(Example)`, `@apiUse`, `@apiVersion` |
| `lib/workers/` | Post-processing: resolve `@apiUse`/`@apiDefine`, group titles, permissions, sample request URLs, structures |
| `lib/filters/` | Remove duplicate fields in error/header/param/success lists |
| `lib/languages/` | Comment syntax for clj, coffee, erl, ex, lua, pm, py, rb, and a default (C-style, used for PHP) |
| `lib/utils/find_files.js` | Walks input folders with `klaw-sync`, applies include/exclude regex filters |
| `lib/plugin_loader.js` | Loads `apidoc-plugin-*` packages |
| `test/` | mocha tests for the parser, `@apiDescription`, `@apiParam`, `unindent` |

How it would run: `npm install`, then `npm test` (jshint + mocha). It is not run on its own; `apidoc` calls `require('apidoc-core').parse(...)`.

## Status

**LIBRARY-USED-BY-apidoc (and so by `wuzzuf-platform`'s build).** Last push 2021-05-11. No CI for the fork (`.travis.yml` is upstream's).

## Related repos

| Repo | Role |
|---|---|
| `apidoc` | The only direct consumer (`package.json` dependency) |
| `wuzzuf-platform` | Pins this commit in `package-lock.json` and `yarn.lock` |
