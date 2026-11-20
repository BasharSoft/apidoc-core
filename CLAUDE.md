# CLAUDE.md

BasharSoft fork of the open-source apiDoc parser library (`apidoc-core` `0.8.3`, MIT). Node.js engine behind the `apidoc` CLI: finds source files, extracts `@api...` comment blocks, and returns the JSON the apidoc template renders.
For Wuzzuf it matters only indirectly: `apidoc` depends on it, and wuzzuf-platform runs `apidoc` at Docker build time for its old API docs. No Wuzzuf-specific code; `package.json` still names upstream.
Live branch: `master` (wuzzuf-platform's lock files pin commit `27a8561`). 2017-era stack.

## Commands

```bash
npm install
npm test                                     # jshint lib/ test/ + mocha test/**/*_test.js
npm run jshint                               # lint only (rules in package.json "jshintConfig")
npx mocha test/parser_api_param_test.js      # one test file
```

Not run on its own; `apidoc` calls `require('apidoc-core').parse(...)`.

## Architecture

| Path | What |
|---|---|
| `lib/index.js` | Entry. `parse(options)`, `getSpecificationVersion()` (spec `0.3.0`). Holds the registry maps `filters`, `languages`, `parsers`, `workers` (tag/extension -> module path) |
| `lib/parser.js` | `Parser` (hub): reads files (`iconv-lite`), finds comment blocks per language, runs element parsers |
| `lib/parsers/` | One parser per tag: `api.js`, `api_define.js`, `api_param.js` (`parse()`, `getGroup()`), `api_success.js`, `api_use.js`, … |
| `lib/workers/` | Post-processing with `preProcess`/`postProcess`: `api_use.js` (resolve `@apiUse`/`@apiDefine`), `*_title.js`, `*_structure.js`, `api_group.js`, `api_sample_request.js`, … |
| `lib/filters/` | Drop duplicate fields in error/header/param/success lists (`postFilter`) |
| `lib/languages/` | Comment syntax per extension; `default.js` (C-style) handles PHP |
| `lib/errors/` | `WorkerError`, `ParameterError`, `ParserError`, `FileError` |
| `lib/utils/find_files.js` | `FindFiles`: walks input folders with `klaw-sync`, applies include/exclude regex |
| `lib/plugin_loader.js` | `PluginLoader`: requires any installed `apidoc-plugin-*` package |
| `lib/worker.js`, `lib/filter.js` | Run the workers and filters over parsed blocks |
| `test/` | mocha + should: `apidoc_test.js`, `parser_api_description_test.js`, `parser_api_param_test.js`, `util_unindent_test.js` |

Flow: `parse(options)` -> `FindFiles` -> `Parser` (language regex per file, element parsers per tag) -> `Worker` (`preProcess`, then `postProcess`) -> `Filter` -> `{ data, project }` JSON back to apidoc.

## Conventions

- New tag: add `lib/parsers/<api_tag>.js` exporting `parse` (plus `path`/`method` like the others) and register it in the `parsers` map in `lib/index.js`. Same pattern for `workers`, `filters`, `languages`.
- Throw the typed errors from `lib/errors/`, not bare `Error`.
- Lint with the `jshintConfig` in `package.json`; tests live in `test/*_test.js`.
- `hooks.md` lists the hook points plugins can use.

## Gotchas

- wuzzuf-platform's `yarn.lock` (used by its Dockerfile) installs this commit from the personal fork `SherifMedhat/apidoc-core` over retired `git://`; its `package-lock.json` uses `BasharSoft/apidoc-core`. Pushing here doesn't reach the build until those locks change.
- `apidoc` depends on this repo by git URL without a commit, so a fresh install without a lock takes `master` HEAD. Keep `master` working.
- Old dependency ranges (`lodash ~4.17.4`, `semver ~5.3.0`, `fs-extra` 3, mocha 3, jshint 2.9).
- `PluginLoader` requires every `apidoc-plugin-*` in `node_modules` at parse time.
- `.travis.yml` is upstream's; no CI runs for this fork.

## Connections

- Direct consumer: [apidoc](https://github.com/BasharSoft/apidoc/blob/master/docs/wiki/README.md) (npm git dependency).
- Indirect: [wuzzuf-platform](https://github.com/BasharSoft/wuzzuf-platform/blob/master/docs/wiki/README.md) build (`gulp old-api-doc` -> `public/docs`).

## Deploy

Not deployed or published to npm. Push to `master` on `https://github.com/BasharSoft/apidoc-core.git`; consumers pick it up through their lock files.

## More

- Wiki: [docs/wiki/README.md](docs/wiki/README.md); [known-issues.md](docs/wiki/known-issues.md)
- Code map: [docs/wiki/code-map/index.md](docs/wiki/code-map/index.md)
- Graph: `graphify explain "Parser" --graph ../graphify/repos/apidoc-core/graph.json`
