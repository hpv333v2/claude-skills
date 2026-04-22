Create a new GitHub repository under a verified account. This skill verifies identity, confirms details, and lets the user choose the repo name before creating.

## Steps

### Phase 1: Discover Available GitHub Identities

1. Read `~/.ssh/config` to find all GitHub SSH host aliases (entries with `HostName github.com`).
2. For each alias, run `ssh -T <alias> 2>&1` to discover which GitHub username it authenticates as.
3. Collect results into a list of: **SSH Alias → GitHub Username**.

### Phase 2: Present Identity Summary & Choose Account

4. Present a summary table to the user:

| SSH Alias | GitHub Username | Type |
|---|---|---|
| *(alias)* | *(username)* | Personal / Org |

5. Ask the user using AskUserQuestion: **"Which GitHub account should own the new repository?"**
   - Provide each discovered username as an option.

### Phase 3: Choose Repository Name

6. Based on the current project folder name, suggest 3 repository name options (e.g., kebab-case, lowercase, abbreviated).
7. Ask the user using AskUserQuestion: **"What should the repository be named?"**
   - Provide the 3 suggestions as options — the user can also type a custom name via "Other".

### Phase 4: Confirm Before Creating

8. Present a final confirmation table:

| Field | Value |
|---|---|
| **GitHub Username** | *(selected)* |
| **Repository Name** | *(selected)* |
| **Visibility** | Public |
| **Full URL** | github.com/username/repo-name |

9. Ask: **"Create this repository?"**
   - Options: **Yes, create it** / **No, abort**

### Phase 5a: If Yes — Create Repository

10. Use the GitHub API via SSH or `gh` CLI to create the repository:
    - Try `gh repo create <username>/<repo-name> --public --confirm` if `gh` is available.
    - If `gh` is not available, use `curl` with a GitHub personal access token, or guide the user to install `gh`.
11. Add the new remote to the current project if the user wants: `git remote add origin git@<ssh-alias>:<username>/<repo-name>.git`
12. Report success with the repo URL.

### Phase 5b: If No — Troubleshoot

10. Ask what needs changing using AskUserQuestion:
    - **Wrong account** — Go back to Phase 2.
    - **Wrong repo name** — Go back to Phase 3.
    - **Want private instead of public** — Update visibility and re-confirm.
    - **Other issue** — Let user describe in free text.
11. Re-run confirmation (Phase 4) after corrections.

## Notes

- Always verify identity BEFORE creating anything.
- Never assume which account to use — always ask.
- Do NOT create the repo without explicit user confirmation.
- Support both personal accounts and organization accounts.
