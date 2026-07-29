## Context

Maintainer scripts that version and ship this action package itself (dogfooded via `.github/workflows/version-or-publish.yml` and `release-pr.yml`).

## Tech

- TypeScript run with `node --experimental-strip-types`
- `@actions/exec` for git/changeset CLI
- Reads root `package.json` via import attributes

## Architecture

- `bump.ts` — `changeset version`, then rewrite README `changesets/action@vN` pin to major line
- `release.ts` — skip if tag exists; detach HEAD; force-add `dist`; `changeset tag`; force-push to `refs/heads/vN`
- `release-pr.ts` — detach; force-add `dist`; force-push to `pr-release` (snapshot builds from `/release-pr` comments)

## Patterns

- DO run `yarn build` before `yarn release` / `yarn release:pr` so `dist/` exists (gitignored in the working tree).
- DO keep README action ref updates inside `bump.ts` so version PRs stay consistent.
- DON'T push release-line branches from feature work — only the release workflows/scripts should force-push `vN` / `pr-release`.
- DO treat tag existence check in `release.ts` as the idempotency gate before rewriting remote history.

## Key Files

- `scripts/bump.ts`
- `scripts/release.ts`
- `scripts/release-pr.ts`
- `.github/workflows/version-or-publish.yml`
- `.github/workflows/release-pr.yml`
