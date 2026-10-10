---
name: publish
description: Build the Astro site, push the current branch to its configured GitHub remote, and deploy it to Cloudflare. Use when the user asks to publish or invoke /publish.
---

# Publish to GitHub and Cloudflare

Publish the current project only when the user invokes this skill or explicitly asks to publish. The workflow has two external steps: push the current branch to GitHub, then deploy the site to Cloudflare.

## 1. Check the repository and changes

- Confirm the working directory is a Git repository. Inspect `git status --short --branch`, the current branch, and configured remotes.
- Require a configured GitHub remote. If this directory is not a Git repository or has no GitHub remote, stop and tell the user what setup is missing. Do not initialize a repository, create a remote, or guess a repository URL.
- Check for a detached HEAD, a dirty working tree, ignored build outputs, and likely secrets. Do not stage credentials, `.env` files, generated output, or unrelated changes.
- If there are changes, review the diff and stage only the intended site changes. Use a concise commit message that reflects the changes. If the intended changes are ambiguous or include unrelated work, ask the user which changes to publish before committing.
- Never force-push. If the branch is behind, diverged, or the push would overwrite remote work, stop and explain the issue.

## 2. Build

- Use the package manager indicated by the repository lockfile. For this project, use `pnpm run build` when `pnpm-lock.yaml` is present; otherwise follow the repository's package-manager conventions.
- If the build fails, stop before pushing or deploying and report the error.

## 3. Push to GitHub

- Commit the reviewed intended changes if needed, then push the current branch to its configured GitHub remote. Do not push tags or other branches unless the user asks.
- If authentication fails, stop and ask the user to complete GitHub authentication. Do not attempt to bypass authentication.
- If the push fails, do not deploy to Cloudflare.

## 4. Deploy to Cloudflare

- Only after the GitHub push succeeds, deploy using the project's configured Cloudflare setup. For this project, use `pnpm exec wrangler deploy` and honor `wrangler.jsonc`.
- Do not change Cloudflare account, domain, route, or DNS settings as part of publishing.
- If Cloudflare authentication fails, ask the user to authenticate with Wrangler. If deployment fails after the push, report that GitHub succeeded and Cloudflare did not, and include the pushed commit.
- On success, report the GitHub branch and commit and the deployed URL returned by Wrangler.
