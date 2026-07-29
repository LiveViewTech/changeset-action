## Context

Action runtime: read Changesets state, open/update version PRs, run publish scripts, push tags, create GitHub Releases.

## Tech

- TypeScript ESM, Node 20 action runtime (`action.yml`)
- `@actions/core|exec|github`, Octokit + `@octokit/plugin-throttling`
- `@changesets/read`, `@changesets/pre`, `@changesets/ghcommit`
- `@manypkg/get-packages`, `semver`, `unified`/remark for changelog slicing
- Tests: Vitest + `fixturez` + mocks of `@actions/github` / `./git.ts`

## Architecture

1. `index.ts` — require `GITHUB_TOKEN`, resolve `cwd`/`commitMode`, setup git user + `~/.netrc`, `readChangesetState()`, then switch:
   - no changesets + `publish` input → configure `~/.npmrc`, `runPublish`
   - changesets with releases → `runVersion`
   - all-empty changesets → no PR
2. `run.ts` — `runVersion` on `changeset-release/<branch>`; `runPublish` execs publish script and parses stdout for tags/releases
3. `git.ts` — `git-cli` (force-push) vs `github-api` (`commitChangesFromRepo`, signed commits)
4. `octokit.ts` — throttled Octokit (retry rate limits up to 2)
5. `utils.ts` — changelog AST extract, changed-package detection

## Patterns

- DO keep version/publish branching in `index.ts` switch; extend cases there, not ad-hoc in helpers.
- DO honor both commit modes when touching git (`git.ts` octokit branch vs CLI).
- DO use `getOptionalInput` / `core.getBooleanInput` for action inputs (`index.ts`).
- DON'T assume publish succeeded unless stdout matches `New tag:` patterns (`run.ts`).
- DON'T truncate PR bodies ad-hoc — `getVersionPrBody` already caps at 60k (`run.ts`).
- DO copy fixtures with `fixturez` and symlink root `node_modules` in tests (`run.test.ts`).

## Key Files

- `src/index.ts` — orchestration and outputs
- `src/run.ts` — version PR + publish/release
- `src/git.ts` — commit/push modes
- `src/readChangesetState.ts` — pre-mode filtering
- `src/utils.ts` — changelog / package helpers
