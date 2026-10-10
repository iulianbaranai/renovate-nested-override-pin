# Renovate reads a nested npm override pin from the hoisted package

Minimal reproduction for a [Renovate](https://github.com/renovatebot/renovate) discussion. See the
[minimal reproduction instructions](https://github.com/renovatebot/renovate/blob/main/docs/development/minimal-reproductions.md).

`package.json` pins `brace-expansion` to `1.1.21` for `minimatch@3.1.5` only, through a nested
`overrides` entry, while the project itself depends on `brace-expansion` `5.0.12`.
`package-lock.json` comes from `npm install --package-lock-only` with npm 11.14.0 and holds both copies:

```json
"node_modules/brace-expansion": { "version": "5.0.12" },
"node_modules/minimatch/node_modules/brace-expansion": { "version": "1.1.21" }
```

## Current behavior

Renovate 44.149.2 reads the locked version of the nested pin from the hoisted entry
`node_modules/brace-expansion`, so the pin gets `lockedVersion: "5.0.12"` and `currentVersion: "5.0.12"`
although its value is `1.1.21`.

- With `renovate.json` as committed (`config:recommended` and `rangeStrategy: "bump"`): the update
  `1.1.21 -> 5.0.12` is classified as `updateType: "patch"`, `isBreaking: false`, in `bucket: "major"`.
  Rules for minor and patch updates (grouping, automerge) then apply to a major jump of the pin.
- With `config:recommended` alone: Renovate proposes no update for the pin at all (`updates: []`, no skip
  reason), because it believes the pin is already at 5.0.12.

minimatch 3 cannot use brace-expansion 5: every pattern with braces throws `TypeError: expand is not a function`.

## Expected behavior

`lockedVersion: "1.1.21"`, read from `node_modules/minimatch/node_modules/brace-expansion`. The update
`1.1.21 -> 5.0.12` is a `major` with `isBreaking: true`, under both configurations.

## How to run

```bash
docker run --rm -v "$PWD:/usr/src/app" -w /usr/src/app \
  -e RENOVATE_PLATFORM=local -e LOG_LEVEL=debug -e LOG_FORMAT=json \
  ghcr.io/renovatebot/renovate:44.149.2
```

In the `"packageFiles with updates"` record, look at the dependency with `"depType": "overrides"` and
`"managerData": { "parents": ["minimatch@3.1.5"] }`.

## Link to the Renovate issue or Discussion

Added once the discussion is open.
