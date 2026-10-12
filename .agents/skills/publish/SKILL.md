---
name: publish
description: Build the Astro site, push the current branch to its configured GitHub remote, and deploy it to Cloudflare. Use when the user asks to publish or invoke /publish.
---

# Publish to GitHub and Cloudflare

Publish the current project only when the user invokes this skill or explicitly asks to publish. The workflow has two external steps: push the current branch to GitHub, then deploy the site to Cloudflare.

## 1. Check the repository and changes

- Confirm the working directory is a Git repository. Inspect `git status --short --branch`, the current branch, and configured remotes.
- If this directory is not a Git repository or has no GitHub remote, explain what is missing and continue setup only if the user explicitly asks. Never initialize a repository or add a remote on your own.
- When explicitly asked to enable Git, initialize only this project directory and use its existing files. When explicitly asked to add a GitHub remote, use the URL the user supplied or a single exact project match from the connected GitHub repository list. Confirm the repository is accessible and has push permission; if the match is ambiguous, ask which repository to use. Never create a GitHub repository or guess a URL.
- If a newly initialized local repository needs to connect to an existing remote branch, fetch it and inspect its commit and file tree first. If the local repository has no commits, attaching the branch with `git reset --mixed origin/<branch>` preserves the working files; review the resulting status and diffs, then set the upstream. Do not reset a repository that already has local commits, and do not overwrite remote files.
- For GitHub authentication, use the installed Git Credential Manager browser login (`git credential-manager github login --browser`) when the user asks to log in. Verify the account with `git credential-manager github list` and verify remote access with `git ls-remote`. If the credential manager is unavailable or login cannot complete, tell the user what authentication is needed. Never handle or expose tokens manually.
- Check for a detached HEAD, a dirty working tree, ignored build outputs, and likely secrets. Do not stage credentials, `.env` files, generated output, or unrelated changes.
- If there are changes, review the diff and stage only the intended site changes. Use a concise commit message that reflects the changes. If the intended changes are ambiguous or include unrelated work, ask the user which changes to publish before committing.
- Never force-push. If the branch is behind, diverged, or the push would overwrite remote work, stop and explain the issue.

## 2. Build

- Use the package manager indicated by the repository lockfile. For this project, use `pnpm run build` when `pnpm-lock.yaml` is present; otherwise follow the repository's package-manager conventions.
- If the build fails, stop before pushing or deploying and report the error.

## 3. Push to GitHub

- Commit the reviewed intended changes if needed, then push the current branch to its configured GitHub remote. Do not push tags or other branches unless the user asks.
- If authentication fails, help the user complete GitHub authentication with Git Credential Manager when available. Do not attempt to bypass authentication.
- If the push fails, do not deploy to Cloudflare.

## 4. Deploy to Cloudflare

- Only after the GitHub push succeeds, deploy using the project's configured Cloudflare setup. For this project, use `pnpm exec wrangler deploy` and honor `wrangler.jsonc`.
- Do not change Cloudflare account, domain, route, or DNS settings as part of publishing.
- If Cloudflare authentication fails, ask the user to authenticate with Wrangler. If deployment fails after the push, report that GitHub succeeded and Cloudflare did not, and include the pushed commit.
- On success, report the GitHub branch and commit and the deployed URL returned by Wrangler.
