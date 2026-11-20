# Known issues

These were found while writing this wiki from the code, as of `master` at `27a8561`. Items marked ✓ were checked by opening the exact line.

**No secrets are committed** in this repo.

| # | Issue | Where | Notes |
|---|---|---|---|
| 1 | ✓ **Installed from a personal fork in production builds** | `wuzzuf-platform/yarn.lock:72-74` | `git://github.com/SherifMedhat/apidoc-core.git#27a8561`. Same commit as here, but the source of truth is a personal account and the `git://` protocol is retired on GitHub |
| 2 | **Unmaintained dependencies** | `package.json` | `fs-extra` 3, `glob` 7, `semver` ~5.3, `lodash` ~4.17.4 (old patch levels with known advisories for lodash), `iconv-lite` 0.4; dev tools `mocha` 3, `jshint` 2.9 |
| 3 | **Plugin loading runs arbitrary packages** | `lib/plugin_loader.js` | Any installed `apidoc-plugin-*` module is required at parse time. Low risk in a build container, but it is code execution from `node_modules` |
| 4 | **Fork with no visible changes** | whole repo | `package.json` still points to upstream; keeping a fork adds maintenance without a clear reason. Upstream apidoc/apidoc-core could replace it |
