# KrunchOS public preview

This repository is an isolated GitHub Pages host for a future public preview of KrunchOS. It does not change or deploy the production website at [krunchyMonkey/krunchymonkey-site](https://github.com/krunchyMonkey/krunchymonkey-site), and it has no dependency on private Tracing Guatemala services.

**The application has not been published.** The deployment workflow is manually triggered and requires an explicit publication confirmation. No custom domain or DNS configuration is used.

## Preview location

The expected URL is <https://krunchymonkey.github.io/krunchos-preview/os/>. Only verified files from `krunchos/dist` are staged below `/os/`; the source static-artifact check enforces relocatable relative references for the project path.

## Source, verification, and approval boundary

The static build originates from `krunchos/` in the main KrunchyMonkey site repository. This workflow accepts only the full SHA that is the current head of source PR [#67](https://github.com/krunchyMonkey/krunchymonkey-site/pull/67), from the fixed repository and branch. It does not accept a repository URL, branch, command, or workflow path as input, and checks out that exact commit rather than a moving ref.

Before building, the workflow requires at least one submitted `APPROVED` review and no outstanding `CHANGES_REQUESTED` review for that exact SHA, plus successful `static-validation`, `preview-checks`, and `e2e` check runs for the same SHA. `preview-checks` includes TypeScript, lint and dependency-boundary checks, formatting, unit/component tests, Vite build, and the static `check:dist` allowlist/privacy/budget gate. `e2e` is a separate CI job and must pass for that SHA; `npm run check` is not represented as running Playwright. The workflow rechecks the PR head, its approval, and these CI checks after environment approval, before deployment.

The workflow pins the current head commits from source PRs #55, #62, and #63, requires those commits to remain the actual PR heads and ancestors of the selected source SHA, and requires approval (with no outstanding change request) on each exact baseline commit. It also checks for the seven expected Linux screenshot files. The exact-SHA `e2e` check then verifies the current visual baselines. At the time this configuration was prepared, PRs #55, #62, and #63 were open drafts with no submitted reviews; PR #63's commit message says its baselines are pending owner review. PR #67 also has no submitted approval. Accordingly, the workflow will currently stop before building until all four exact commits have formal PR approval. Do not treat file presence or passing tests as owner approval.

The build produces a SHA-256 manifest and provenance for the source SHA, required CI checks, and baseline commit IDs. A separate publishing job verifies the downloaded artifact against that manifest and rejects missing entry points, unexpected top-level content, symlinks, source maps, and key material. The only artifact passed to Pages is `krunchos/dist` staged under `os/`.

## Manually publishing a preview

Publishing requires both explicit authorization in the manual dispatch and approval of the `github-pages` environment if required reviewers are configured. The deploy job only runs when the workflow is dispatched from this preview repository's `main` branch.

1. Review source PR #67 and its candidate commit, including the privacy/build report and the E2E/visual CI results. Confirm PRs #55, #62, and #63's screenshot baselines have owner review.
2. After this preview configuration is reviewed and merged to `main`, open **Actions → Deploy KrunchOS preview → Run workflow**.
3. Enter the full 40-character SHA of the current PR #67 head. Check **I reviewed this exact build and authorize publishing it publicly** only when you intend to publish that build.
4. Approve the pending deployment in the `github-pages` environment when GitHub requests it. The workflow revalidates the PR head and CI after that approval, builds and checks the exact source SHA, then validates the artifact manifest before Pages upload.
5. Review the Actions summary, provenance artifact, successful deployment, and `github-pages` deployment URL before sharing <https://krunchymonkey.github.io/krunchos-preview/os/>.

There is no push, pull-request, release, schedule, or API-triggered deployment path. The source build job has no secrets and read-only repository access. Do not add production credentials or connect the workflow to private networks or services.

## Rollback

To roll back, run the workflow manually with the full source SHA used by the last known-good preview, but only if it remains the current head of PR #67 and its checks are still successful. Otherwise, update/review PR #67 to restore the desired release candidate and use its new full head SHA. Confirm the new run's source SHA and per-file manifest against the previous successful run. A failed build, check, source identity test, or hash check cannot proceed to deployment.

## Public and synthetic data only

The preview is public and must contain only public, synthetic, or clearly conceptual demo data. Do not include real client/customer records, secrets, credentials, private endpoints, or data from Tracing Guatemala. The source `check:dist` privacy gate rejects known private-host, network, API, and key-material patterns, but manual review is still required; automated checks are not a substitute for review.

## Repository safeguards and owner setup

The workflow has repository-level `contents: read`. Its build job only has `contents: read`; only the deploy job receives `pages: write`, `id-token: write`, and `actions: read`. Official GitHub Actions are pinned to reviewed commit SHAs. Automatic merges are disabled. The workflow contains no secrets.

GitHub settings APIs returned 403 or 404 in this session, so these settings have **not** been configured or verified. Before merging this PR or attempting any deployment, an owner should use:

1. [Pages settings](https://github.com/krunchyMonkey/krunchos-preview/settings/pages): choose **GitHub Actions** as the publishing source. Do not configure branch publishing, a custom domain, or DNS.
2. [Environment settings](https://github.com/krunchyMonkey/krunchos-preview/settings/environments): create `github-pages`, allow deployment from `main` only, and add required owner reviewers if the repository plan supports them. Do not add environment secrets. The dispatch confirmation is an additional explicit intent check, not a substitute for environment protection.
3. [Branch protection](https://github.com/krunchyMonkey/krunchos-preview/settings/branches): require pull requests and at least one approval for `main`, dismiss stale approvals, and keep automatic merges disabled. This ensures workflow changes cannot be pushed directly without review.
4. [Actions settings](https://github.com/krunchyMonkey/krunchos-preview/settings/actions): set default `GITHUB_TOKEN` permissions to read-only; disable the permission for Actions to create and approve pull requests; allow only these pinned Actions: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020`, `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`, `actions/download-artifact@d3f86a106a0bac45b974a628896c90dbdf5c8093`, `actions/upload-pages-artifact@56afc609e74202658d3ffba0e8f6dda462b719fa`, and `actions/deploy-pages@d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e5`.
5. [Secrets settings](https://github.com/krunchyMonkey/krunchos-preview/settings/secrets/actions): confirm no repository secrets are present. No repository or environment secret is needed for this workflow.

The current source candidate is commit `d66bfa2e0eec3d977fd8e44b0c7df5cae8341103`. At audit time, source PR #67 was open and draft; its `static-validation`, `preview-checks`, and `e2e` checks all passed for that exact SHA, but it had no submitted approval. Baseline PRs #55, #62, and #63 likewise had no submitted reviews. This repository's deployment workflow itself has not been run, and the application remains unpublished.
