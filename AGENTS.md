## Purpose

GitHub Action that automates Changesets releases: opens or updates a version PR when changesets exist, and optionally publishes packages to npm and creates GitHub releases when none remain.

## Project Snapshot

- Tech: TypeScript (ESM), Node >= 20, Yarn 1, Rollup+esbuild bundle, Vitest
- Type: single-package GitHub Action (`@changesets/action`); remote may be a LiveViewTech fork of upstream `changesets/action`
- Runtime entry: `action.yml` → `dist/index.js` (built from `src/index.ts`)
- Subguides:
  - `src/AGENTS.md` — action runtime
  - `scripts/AGENTS.md` — self-release tooling

## Commands

```bash
yarn install --frozen-lockfile
yarn build
yarn typecheck
yarn test
yarn test:watch
yarn bump          # version + pin README action ref
yarn release       # publish dist to vN branch
yarn release:pr    # snapshot push to pr-release
yarn changeset
```

CI runs `yarn typecheck` and `yarn test` via `.github/workflows/ci.yml` (setup: `.github/actions/ci-setup`).

## Conventions

- Branch for this intelligence refresh work: follow repo PR norms; Changesets release PRs use `changeset-release/<base>`.
- ESM only (`"type": "module"`); import local modules with `.ts` extensions (see `src/index.ts`).
- `tsconfig.json`: `strict`, `verbatimModuleSyntax`, `noEmit` (typecheck only; ship via Rollup).
- Do not commit `dist/` on feature branches — it is gitignored; release scripts force-add it onto release refs.
- Action inputs/outputs are defined only in `action.yml`; keep README examples aligned when changing them.

## Directory Map

- `src/` → see `src/AGENTS.md`
- `scripts/` → see `scripts/AGENTS.md`
- `__fixtures__/` — disposable Changesets workspaces for Vitest (`fixturez` copies)
- `.github/` — CI, dogfood `version-or-publish`, `/release-pr` comment workflow
- `action.yml` — marketplace/action interface
- `types/` — ambient typings (e.g. `fixturez`)

## Gotchas

- **Force-push**: `git-cli` mode force-pushes the version branch (`src/git.ts`); release scripts force-push `vN` / `pr-release` (`scripts/release.ts`, `scripts/release-pr.ts`).
- **Secrets on disk**: action writes `GITHUB_TOKEN` to `~/.netrc` and may write `NPM_TOKEN` into `~/.npmrc` (`src/index.ts`).
- **Publish detection**: `runPublish` regex-matches `New tag:` lines in publish script stdout (`src/run.ts`); custom publish output can skip tags/releases.
- **Clean checkout has no `dist/`**: consumers should use release branches/tags that contain the bundle, not raw `main`.
