# Globnetics GitHub Actions Governance Workflows

These workflows provide **100% free enforcement** for branch naming, commit standards, corporate email validation, and secret protection. They work seamlessly on **both private and public repositories** under the **GitHub Free Organization** plan.

---

## Workflow Catalog

| Workflow File | Equivalent Ruleset | What It Enforces |
| :--- | :--- | :--- |
| **`branch-name-check.yml`** | `branch_name_pattern` | Validates PR branch names against `(feature\|bugfix\|hotfix\|chore\|refactor\|docs)/*`. Fails PR if branch name is non-standard. |
| **`conventional-commits-check.yml`** | `commit_message_pattern` | Validates that PR titles conform to Conventional Commits (`feat:`, `fix:`, `docs:`, etc.) with optional ticket linking (`[GLOB-123]`). |
| **`corporate-email-check.yml`** | `commit_author_email_pattern` | Verifies that all commit authors in the PR use a `@globnetics.com` email address, preventing IP leakage from personal emails. |
| **`secrets-and-file-size-guard.yml`** | `file_extension_restriction` + `max_file_size` | Blocks files with sensitive extensions (`.pem`, `.key`, `.p12`, `.pfx`, `.kdbx`, `.env`), sensitive paths (`**/secrets/**`), and files over 50MB. |
| **`semver-tag-validation.yml`** | `tag_name_pattern` | Validates that Git release tags strictly match Semantic Versioning (`v1.0.0`) and verifies the tag commit belongs to the default branch. |
| **`pr-quality-and-linear-history.yml`** | `required_linear_history` | Detects merge commits in the PR branch to enforce clean linear history (rebase/squash only). |

---

## Quick Setup Instructions

### Option 1: Install in an Individual Repository
1. In your repository, create a directory path: `.github/workflows/`
2. Copy any or all of the `.yml` files from this folder into `.github/workflows/`:
   ```bash
   # From your local project root:
   mkdir -p .github/workflows
   cp /path/to/github-actions-workflows/*.yml .github/workflows/
   ```
3. Commit and push:
   ```bash
   git add .github/workflows/
   git commit -m "ci: add Globnetics repository governance workflows"
   git push origin main
   ```

---

### Option 2: Share Workflows Across All Repositories (Org-Level)
You can centralize these workflows so you don't have to duplicate them across dozens of repositories:

1. Create a repository in your organization named `.github`:
   `https://github.com/Globnetics/.github`
2. Store these workflows in `.github/workflows/` of that central repository.
3. In any other repository in Globnetics, add a lightweight caller workflow:
   ```yaml
   name: "Globnetics Governance"
   on:
     pull_request:
       types: [opened, synchronize, reopened, edited]

   jobs:
     call-branch-check:
       uses: Globnetics/.github/.github/workflows/branch-name-check.yml@main
     call-secret-guard:
       uses: Globnetics/.github/.github/workflows/secrets-and-file-size-guard.yml@main
     call-email-guard:
       uses: Globnetics/.github/.github/workflows/corporate-email-check.yml@main
   ```

---

## Making Workflows Required Before Merge

Once a workflow runs on a Pull Request:
1. In your repository on GitHub, go to **Settings** > **Branches** (or **Rulesets** on public repos).
2. Under **Branch protection rules** (or **Rulesets**), edit your default branch rule.
3. Check **Require status checks to pass before merging**.
4. Search for and check each job name:
   * `Validate Branch Name`
   * `Validate PR Title (Conventional Commits)`
   * `Verify @globnetics.com Committer Email`
   * `Check File Extensions, Paths & Size`
   * `Linear History & PR Quality Gate`
5. Save changes. GitHub will now block the **Merge** button until all workflows pass green!
