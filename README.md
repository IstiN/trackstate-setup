# TrackState.AI Setup

This repository is the lightweight fork-and-run template for TrackState.AI.

Fork this repository to your own GitHub account or organization, enable GitHub Pages with **Source: GitHub Actions**, then run **Install / Update TrackState** from the Actions tab. The workflow downloads the selected version of `IstiN/trackstate`, builds the Flutter SPA for your fork, and deploys it to GitHub Pages.

## First install

1. Fork `IstiN/trackstate-setup`.
2. Open **Settings > Pages** in your fork.
3. Set **Build and deployment > Source** to **GitHub Actions**.
4. Optional: open **Settings > Secrets and variables > Actions > Variables** and configure GitHub App login:
   - `TRACKSTATE_GITHUB_APP_CLIENT_ID`: the GitHub App **Client ID** from the app's General settings, not the numeric App ID.
   - `TRACKSTATE_GITHUB_AUTH_PROXY_URL`: your auth broker URL. A static Flutter app must not contain a client secret or private key, so this broker exchanges the GitHub callback code for a user access token and redirects back to your Pages URL with `#trackstate_token=<token>`.
5. Open **Actions > Install / Update TrackState**.
6. Click **Run workflow**.
7. Keep `trackstate_ref` as `main` for latest, or enter a tag/commit SHA for a fixed version.

Your app will be published at:

```text
https://<owner>.github.io/<repo>/
```

## Updating TrackState

Run the same **Install / Update TrackState** workflow again:

- `main` installs the latest source from `IstiN/trackstate`;
- a tag such as `v0.1.0` installs that release line;
- a commit SHA pins the app to an exact build.

The workflow does not commit generated web assets to this repository. It publishes the compiled app as a Pages artifact, keeping the fork clean and focused on tracker data/configuration.

The deployed Pages artifact contains only the Flutter application. Tracker files are read at runtime from this repository through the GitHub API (`git/trees` for file discovery and `contents` for markdown/config reads), and writes are committed back with the GitHub Contents API.

## Release automation

Every push to `main` publishes a stable GitHub release and matching semantic version tag (`vX.Y.Z`). The release workflow derives the next patch version from existing semantic tags, creates the new tag on the pushed commit, and publishes the release notes automatically.

Workflow changes are validated by the `Actionlint` GitHub Actions workflow on
pushes and pull requests touching `.github/workflows/**`, so broken workflow
syntax fails with a dedicated `actionlint` run before it can silently disrupt
release automation.

## Repository permissions

Use a **Fine-grained personal access token (default)** when you want the
simplest setup path. Grant only the target setup repository and these
repository permissions:

- Metadata: Read-only
- Contents: Read and write

The optional GitHub App flow uses the same repository permissions and also
requires `TRACKSTATE_GITHUB_APP_CLIENT_ID` and
`TRACKSTATE_GITHUB_AUTH_PROXY_URL` before rebuilding the Pages artifact.

## CLI quick start

The generated app reads from `IstiN/trackstate` by default and uses
`DEMO/project.json` plus `DEMO/config/*.json` for editable tracker
configuration. Keep attachments under each issue's `attachments/` directory
and store large binaries through Git LFS.

Run this command to validate that your authenticated GitHub CLI session can
read the project JSON from your forked setup repository:

```bash
gh api repos/<repository>/contents/<project-path>?ref=<default-branch> --header "Accept: application/vnd.github.raw+json"
```

Replace `<repository>` with your fork (for example,
`octocat/trackstate-setup`), `<default-branch>` with that fork's default
branch (usually `main`), and `<project-path>` with `DEMO/project.json`. The
command should print the same JSON object stored in your fork's
`DEMO/project.json`.

If you also want a negative check, query a non-existent file path (for
example, `DEMO/project.missing.json`) and expect `gh api` to fail with an
HTTP `404 Not Found` response (non-zero exit status):

```bash
gh api repos/<repository>/contents/DEMO/project.missing.json?ref=<default-branch> --header "Accept: application/vnd.github.raw+json"
```

## Login options

TrackState offers two login paths:

1. **Fine-grained token**: click **Connect GitHub**, paste a token with repository **Metadata: Read-only** and **Contents: Read and write**, and optionally enable **Remember on this browser**. Remembered tokens are stored in this browser/device storage so you do not need to paste them after every refresh. Do not enable this on shared machines.
2. **GitHub App login**: configure `TRACKSTATE_GITHUB_APP_CLIENT_ID` and `TRACKSTATE_GITHUB_AUTH_PROXY_URL`, then rebuild with **Install / Update TrackState**. The app redirects to the broker/GitHub flow and accepts a callback token from the URL fragment.

### GitHub App redirect setup

In your GitHub App settings, add your deployed Pages URL as a callback target:

```text
https://<owner>.github.io/<repo>/
```

Because GitHub's web authorization flow requires a secret exchange, the callback must be handled by your auth broker/proxy. The proxy should:

1. Receive the GitHub callback code.
2. Exchange it server-side using the GitHub App client secret/private key.
3. Redirect to `https://<owner>.github.io/<repo>/#trackstate_token=<user-access-token>`.

The Flutter app stores that token locally for the repository and uses it for read/write GitHub API calls.

## Tracker data

The demo project is stored in `DEMO/` using Git-native markdown and JSON files. You can replace it with your own project while preserving the same structure.

```text
DEMO/
  .trackstate/
    index/
      issues.json
      deleted.json
  project.json
  config/
    resolutions.json
  DEMO-1/
    main.md
    DEMO-2/
      main.md
      acceptance_criteria.md
      comments/
      links.json
      attachments/
        board-preview.svg
      DEMO-3/
        main.md
```

Issue frontmatter stores canonical machine ids such as `issueType: story`, `status: in-review`, `priority: high`, and `fixVersions: [mvp]`. Localized labels come from `config/` and `config/i18n/*.json`, while `.trackstate/index/*.json` provides the generated key/path and tombstone lookup artifacts needed for stable issue resolution after moves.

Large attachments should be stored through Git LFS. `.gitattributes` already tracks common binary formats.

<!-- TS-230 probe 2026-05-09T18:36:10.545601Z -->

<!-- TS-230 probe 2026-05-09T18:57:39.111190Z -->

<!-- TS-230 probe 2026-05-10T10:03:05.409963Z -->

<!-- TS-230 probe 2026-05-10T12:35:20.406985Z -->

<!-- TS-230 probe 2026-05-10T15:42:53.397417Z -->
