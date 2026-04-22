Deploy the latest changes to Cloudflare Pages via the hpv333v2 GitHub account. This skill works on any project — it dynamically reads the repo config.

## Steps

### Phase 1: Gather & Verify

1. **Detect repository**: Run `git remote -v` to get the remote URL and `git branch --show-current` to get the current branch. Do NOT hardcode any repo URL — always read it from the project.
2. **Verify SSH identity**: Run `ssh -T github-codeflux 2>&1` and confirm it says `Hi hpv333v2!`. If it does NOT authenticate as `hpv333v2`, STOP and alert the user — do not push.
3. **Verify git user**: Run `git config user.name` and `git config user.email` to get the committer identity.
4. **Check changes**: Run `git status` to detect uncommitted changes and `git log --oneline -3` for recent commits.

### Phase 2: Confirm with User

5. **Present a summary table** to the user using AskUserQuestion. Display the following in a clean readable markdown table:

| Field | Value |
|---|---|
| **GitHub User (SSH)** | *(from ssh -T output)* |
| **Git User** | *(user.name)* |
| **Git Email** | *(user.email)* |
| **Remote URL** | *(from git remote -v)* |
| **Branch** | *(current branch)* |
| **Pending Changes** | *(count of modified/new/deleted files)* |

6. Ask the user: **"Does this look correct? Proceed with deploy?"**
   - Options: **Yes, deploy** / **No, abort**

### Phase 3a: If User says Yes — Deploy

7. If there are unstaged/uncommitted changes:
   - Stage all modified, new, and deleted files (exclude sensitive files like `.env`, credentials, `Official_Documents/`, etc.).
   - Commit with a clear, descriptive message summarizing the changes.
8. Push to the detected remote and branch (e.g., `git push origin main`).
9. Confirm the push succeeded and report deployment status.

### Phase 3b: If User says No — Troubleshoot

7. Ask the user what is wrong using AskUserQuestion with these options:
   - **Wrong GitHub account** — Guide user to check `~/.ssh/config` and update the remote URL to use the correct SSH alias.
   - **Wrong branch** — Ask which branch to push to and switch.
   - **Wrong remote/repo** — Ask for the correct remote URL and offer to update it with `git remote set-url`.
   - **Other issue** — Let user describe the problem in free text.
8. After the user clarifies, re-run Phase 1 and Phase 2 to re-verify before deploying.

## Notes

- SSH alias `github-codeflux` uses key `~/.ssh/id_codeflux` which authenticates as `hpv333v2` on GitHub.
- Cloudflare Pages is connected to the `hpv333v2` GitHub account and auto-deploys on push to `main`.
- Do NOT force push. Always create new commits.
- Do NOT commit `.env`, credentials, secrets, or directories like `Official_Documents/`.
- This skill is project-agnostic — it reads the remote URL from whatever repo it is run in.
