# Globnetics Enterprise GitHub Rulesets

This directory contains a complete, production-grade collection of **GitHub Repository & Organization Rulesets** tailored for **Globnetics**, inspired by the official [`github/ruleset-recipes`](https://github.com/github/ruleset-recipes) catalog and enhanced with modern security and governance rules widely adopted by top-tier tech companies.

---

## Catalog Overview

```
rulesets/
├── branch-rulesets/
│   ├── 01-enterprise-default-branch.json      # Hardened main/master protection
│   ├── 02-feature-branch-workflow.json        # Branch naming convention (feature/*, bugfix/*)
│   ├── 03-conventional-commits-and-jira.json  # Conventional Commits & GLOB-* ticket linking
│   ├── 04-corporate-committer-email.json      # Committer email enforcement (@globnetics.com)
│   └── 05-release-and-hotfix-branches.json    # Release and staging branch stability
├── push-rulesets/
│   ├── 01-secret-and-credential-protection.json # Server-side block for .pem, .key, .env, secrets
│   └── 02-max-file-size-guard.json            # 50MB max file size to prevent repo bloat
├── tag-rulesets/
│   └── 01-protected-semver-release-tags.json  # Immutable SemVer release tags (vX.Y.Z)
└── org-rulesets/
    ├── 01-org-wide-default-branch-baseline.json # Baseline branch policy across ALL repos
    ├── 02-org-wide-security-and-secret-prevention.json # Baseline secret push guard across ALL repos
    └── 03-org-wide-tag-governance.json        # Baseline tag protection across ALL repos
```

---

## Detailed Ruleset Descriptions

### 1. Branch Rulesets (`branch-rulesets/`)

#### `01-enterprise-default-branch.json`
* **Target:** `~DEFAULT_BRANCH` (`main` / `master`)
* **Mode:** `evaluate` (test without blocking)
* **What it enforces:**
  * **2 Approving Reviews:** Peer review requirement before merge.
  * **CODEOWNERS Review:** Approvals from designated area owners are mandatory.
  * **Dismiss Stale Reviews:** New pushes automatically reset existing approvals.
  * **Require Last Push Approval:** Prevents sneaking commits in after sign-off.
  * **Conversation Resolution:** All comment threads on the PR must be explicitly marked resolved.
  * **Linear History:** Rebase or squash merges only (no merge commits).
  * **Signed Commits:** Requires cryptographically verified GPG/SSH commit signatures (SOC2 / ISO 27001 standard).
  * **Strict CI Status Checks:** Requires `ci/test`, `ci/lint`, and `security/trivy` to pass against the up-to-date base branch.
  * **Deletion & Force Push Protection:** Permanently blocks deleting or force-pushing to the default branch.

#### `02-feature-branch-workflow.json`
* **Target:** All working branches (excludes `main`, `release/*`, `staging`)
* **What it enforces:**
  * **Structured Naming:** Enforces regex `^(feature|bugfix|hotfix|chore|refactor|docs)/[a-z0-9._-]+$`.
  * Prevents messy or untraceable branch names like `temp-fix`, `test`, or personal names.
  * Allows developers full freedom to rebase (`force-push`) and delete their own feature branches upon merge.

#### `03-conventional-commits-and-jira.json`
* **Target:** `~DEFAULT_BRANCH`, `refs/heads/release/*`
* **What it enforces:**
  * Validates commit messages using the **Conventional Commits** specification (`feat`, `fix`, `docs`, `chore`, `refactor`, etc.) with optional Globnetics issue keys (e.g. `feat(auth): GLOB-124 add SSO login`).
  * Enables automated changelog generation and SemVer release management (via `semantic-release` or GitHub `Release Please`).

#### `04-corporate-committer-email.json`
* **Target:** `~DEFAULT_BRANCH`, `refs/heads/feature/*`, `refs/heads/release/*`
* **What it enforces:**
  * Requires all commit authors to use a verified corporate email: `*@globnetics.com`.
  * Blocks accidental commits from personal emails (`@gmail.com`, `@users.noreply.github.com`), safeguarding corporate intellectual property (IP) and audit trails.

#### `05-release-and-hotfix-branches.json`
* **Target:** `refs/heads/release/*`, `refs/heads/staging`
* **What it enforces:**
  * Stabilization rules during release preparation: requires 1 approval, CODEOWNERS sign-off, passing integration/E2E test status checks, and blocks deletion and force-pushes.

---

### 2. Push Rulesets (`push-rulesets/`)

#### `01-secret-and-credential-protection.json`
* **Target:** `push`
* **What it enforces:**
  * Blocks pushes containing files with sensitive extensions: `.pem`, `.key`, `.p12`, `.pfx`, `.pkcs12`, `.kdbx`, `.jks`, `.keystore`, `.ovpn`.
  * Restricts commits touching sensitive paths: `**/.env*`, `**/secrets/**`, `**/credentials/**`, `**/id_rsa*`, `**/id_ed25519*`.
  * Catches credential leaks at the Git protocol level even if `.gitignore` was mistakenly modified.

#### `02-max-file-size-guard.json`
* **Target:** `push`
* **What it enforces:**
  * Limits maximum individual file size to **50 MB** (`52,428,800 bytes`).
  * Prevents accidental commits of database dumps, large binaries, or video assets from permanently bloating the Git packfile history.

---

### 3. Tag Rulesets (`tag-rulesets/`)

#### `01-protected-semver-release-tags.json`
* **Target:** All tags (`~ALL`)
* **What it enforces:**
  * Enforces standard Semantic Versioning naming (`^v[0-9]+\.[0-9]+\.[0-9]+(-[0-9A-Za-z.-]+)?$`, e.g. `v1.0.0`, `v2.4.1-rc.1`).
  * Blocks deleting (`deletion`), updating (`update`), or moving (`non_fast_forward`) tags.
  * Guarantees release immutability to prevent software supply-chain attacks.

---

### 4. Organization Baseline Rulesets (`org-rulesets/`)

#### `01-org-wide-default-branch-baseline.json`
* **Scope:** All repositories in the Globnetics GitHub organization (`~ALL`).
* **What it enforces:** Establishes an organization-wide safety net ensuring every default branch requires peer review, conversation resolution, and blocks deletion and force-pushes.

#### `02-org-wide-security-and-secret-prevention.json`
* **Scope:** All repositories in the Globnetics organization (`~ALL`).
* **What it enforces:** Applies private key/certificate extension blocks and 50MB file size limits globally across every repo.

#### `03-org-wide-tag-governance.json`
* **Scope:** All repositories in the Globnetics organization (`~ALL`).
* **What it enforces:** Global SemVer naming enforcement and immutable tag guarantees across every repo.

---

## How to Import into GitHub

### Option A: Using the GitHub Web UI
1. Navigate to your repository or organization on GitHub.
2. Go to **Settings** > **Rules** > **Rulesets**.
3. Click the dropdown next to **New ruleset** and select **Import a ruleset**.
4. Upload any of the JSON files from this directory.
5. Review the imported rules and click **Save changes**.

> [!NOTE]
> Each JSON file includes a top-level `"_comment"` property for easy human reference. If GitHub's UI importer reports an unrecognized property error, run the stripping snippet below or delete the `_comment` line before uploading.

### Stripping `_comment` for Strict GitHub Imports (PowerShell)
```powershell
# Strips _comment from a JSON file and outputs a clean import file
$json = Get-Content .\01-enterprise-default-branch.json -Raw | ConvertFrom-Json
$json.PSObject.Properties.Remove('_comment')
$json | ConvertTo-Json -Depth 10 | Set-Content .\01-clean.json
```

### Option B: Using GitHub CLI (`gh api`)
```bash
# Import a repository-level ruleset
gh api --method POST -H "Accept: application/vnd.github+json" \
  /repos/globnetics/{repo_name}/rulesets \
  --input 01-clean.json

# Import an organization-level ruleset
gh api --method POST -H "Accept: application/vnd.github+json" \
  /orgs/globnetics/rulesets \
  --input 01-org-clean.json
```

---

## Moving from `evaluate` to `active`
All rulesets are initially configured with:
```json
"enforcement": "evaluate"
```
**Evaluate mode** allows you to see how the rules would affect your team's pull requests and pushes under **Insights > Rule insights** without actually blocking contributors. Once your team is aligned and CI checks match your configuration, change `"enforcement"` to `"active"` to begin full enforcement.
