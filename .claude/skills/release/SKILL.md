---
name: release
description: Trigger the CI release pipeline for gemini-plugin-cc. Maintainer-only.
disable-model-invocation: true
argument-hint: patch|minor|major
allowed-tools: Bash(gh workflow run *) Bash(gh run *) Bash(gh api *)
---

## Trigger the release pipeline

This skill fires `release-draft.yml` on CI. The workflow bumps versions, generates the CHANGELOG entry, and opens a release PR. You review and merge that PR; `release-publish.yml` then tags and creates the GitHub Release.

## Validate the argument

The argument must be exactly one of `patch`, `minor`, `major`. If `$ARGUMENTS` is anything else (empty, misspelled, multiple values), stop and print:

> Usage: `/release patch|minor|major`

Do not run any commands.

## Dispatch the workflow

With a valid bump type, run:

```
gh workflow run release-draft.yml -f bump=$ARGUMENTS
```

If the call fails with a permissions error, print:

> This skill is maintainer-only. The repository permissions gate release dispatch; your account does not have `actions:write` on abiswas97/gemini-plugin-cc.

Stop.

## Show the run

On success, wait a moment, then run:

```
gh run list --workflow=release-draft.yml --limit=1 --json databaseId,url,status
```

Print the URL so the user can watch it. Tell them:

- The run will pause on the `release` environment approval gate
- They need to approve it in the Actions tab to proceed
- Once approved, it will run CI, generate the CHANGELOG, and open a release PR
