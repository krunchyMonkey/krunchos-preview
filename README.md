# KrunchOS public preview

This repository is an isolated GitHub Pages host for a future public preview of KrunchOS. It does not change or deploy the production website at [krunchyMonkey/krunchymonkey-site](https://github.com/krunchyMonkey/krunchymonkey-site), and it has no dependency on private Tracing Guatemala services.

**The application has not been published.** The deployment workflow is manually triggered and requires an explicit publication confirmation. No custom domain or DNS configuration is used.

## Preview location

The expected URL is <https://krunchymonkey.github.io/krunchos-preview/os/>. Only verified files from `krunchos/dist` are staged below `/os/`; the source static-artifact check enforces relocatable relative references for the project path.

## Source, verification, and approval boundary

The static build originates from `krunchos/` in the KrunchyMonkey site repository. This workflow accepts only the exact full SHA pinned in its reviewed configuration **and** currently at the head of source PR [#67](https://github.com/krunchyMonkey/krunchymonkey-site/pull/67), from the fixed repository and branch. The source PR may be a draft; the source commit does not have to be merged into the source repository's `main`. The workflow does not accept a repository URL, branch, command, or workflow path as input, and checks out that exact commit rather than a moving ref.

Before building, the workflow requires successful GitHub Actions `static-validation`, `preview-checks`, and `e2e` check runs for that exact SHA. `preview-checks` includes TypeScript, lint and dependency-boundary checks, formatting, unit/component tests, Vite build, and the static `check:dist` allowlist/privacy/budget gate. `e2e` is a separate CI job and must pass for that SHA; `npm run check` is not represented as running Playwright. The workflow rechecks the pinned PR head and these CI checks after environment approval, before deployment.

The owner attested to the seven original visual baselines in [PR #55](https://github.com/krunchyMonkey/krunchymonkey-site/pull/55#issuecomment-6704440128), and their updates in [PR #62](https://github.com/krunchyMonkey/krunchymonkey-site/pull/62#issuecomment-6704647932) and [PR #63](https://github.com/krunchyMonkey/krunchymonkey-site/pull/63#issuecomment-6704648784). These sign-offs cover visual baselines only, not a deployment. The workflow records their exact commit SHAs, requires them to be ancestors of the pinned candidate, and compares the entire Linux screenshot tree to the approved PR #63 commit; it also checks the seven expected files exist. The exact-SHA `e2e` check verifies the current visual baselines. The workflow does not treat arbitrary comments as authorization or require a formal GitHub review of one's own PR.

The workflow's fixed candidate and baseline pins are an auditable release record, not proof of permission to publish: protect preview `main` and require review of every workflow/pin change. If repository policy requires an independent PR approval, a separate reviewer must provide it; do not bypass the rule. A new source head or changed visual baselines require fresh owner review and a reviewed PR updating the pins and evidence before deployment.

The build produces a SHA-256 manifest and provenance for the source SHA, preview workflow commit, required CI checks, and baseline commit IDs. A separate publishing job verifies the downloaded artifact against that manifest and rejects missing entry points, unexpected top-level content, symlinks, source maps, and key material. The only artifact passed to Pages is `krunchos/dist` staged under `os/`.

## Manually publishing a preview

Publishing requires both explicit authorization in the manual dispatch and approval of the `github-pages` environment if required reviewers are configured. The deploy job only runs when the workflow is dispatched from this preview repository's `main` branch.

1. Complete the owner settings below. Review source PR #67 and its exact pinned candidate, including its build/privacy report, E2E/visual CI results, the baseline sign-offs, and the reviewed preview workflow pins.
2. After this preview configuration is reviewed and merged to `main`, open **Actions → Deploy KrunchOS preview → Run workflow**.
3. Enter the full 40-character SHA pinned in the workflow, which must also be the current PR #67 head. Check **I reviewed this exact build and authorize publishing it publicly** only when you intend to publish that build. The build job first verifies the source and produces a manifest.
4. Approve the pending deployment in the `github-pages` environment when GitHub requests it. The deploy job then revalidates the PR head and CI, and validates the artifact manifest before Pages upload.
5. Review the Actions summary, provenance artifact, successful deployment, and `github-pages` deployment URL before sharing <https://krunchymonkey.github.io/krunchos-preview/os/>.

There is no push, pull-request, release, or schedule trigger; `workflow_dispatch` can be invoked via GitHub's UI or API. The source build job has no secrets and read-only repository access. Do not add production credentials or connect the workflow to private networks or services.

## Rollback

To roll back, run the workflow manually with the full source SHA used by the last known-good preview, but only if it remains the workflow's pinned candidate and the current head of PR #67 and its checks are still successful. Otherwise, review the desired candidate and update the workflow pins through a reviewed preview PR; PR #67 must point at that exact SHA. Confirm the new run's source SHA and per-file manifest against the previous successful run. A failed build, check, source identity test, or hash check cannot proceed to deployment.

## Public and synthetic data only

The preview is public and must contain only public, synthetic, or clearly conceptual demo data. Do not include real client/customer records, secrets, credentials, private endpoints, or data from Tracing Guatemala. The source `check:dist` privacy gate rejects known private-host, network, API, and key-material patterns, but manual review is still required; automated checks are not a substitute for review.

## Repository safeguards and owner setup

The workflow has repository-level `contents: read`. Its build job only has `contents: read`; only the deploy job receives `pages: write`, `id-token: write`, and `actions: read`. Official GitHub Actions are pinned to reviewed commit SHAs. Automatic merges are disabled. The workflow contains no secrets.

GitHub settings APIs returned 403 or 404 in this session, so these settings have **not** been configured or verified. Before merging this PR or attempting any deployment, an owner should use:

1. [Pages settings](https://github.com/krunchyMonkey/krunchos-preview/settings/pages): choose **GitHub Actions** as the publishing source. Do not configure branch publishing, a custom domain, or DNS.
2. [Environment settings](https://github.com/krunchyMonkey/krunchos-preview/settings/environments): create `github-pages`, allow deployment from `main` only, and add required owner reviewers if the repository plan supports them. Do not add environment secrets. The dispatch confirmation is an additional explicit intent check, not a substitute for environment protection.
3. [Branch protection](https://github.com/krunchyMonkey/krunchos-preview/settings/branches): require pull requests and at least one approval for `main`, dismiss stale approvals, and keep automatic merges disabled. This ensures workflow changes cannot be pushed directly without review.
4. [Actions settings](https://github.com/krunchyMonkey/krunchos-preview/settings/actions): set default `GITHUB_TOKEN` permissions to read-only; disable the permission for Actions to create and approve pull requests; allow only these pinned Actions: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020`, `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`, `actions/download-artifact@d3f86a106a0bac45b974a628896c90dbdf5c8093`, `actions/upload-pages-artifact@56afc609e74202658d3ffba0e8f6dda462b719fa`, and `actions/deploy-pages@d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e`.
5. [Secrets settings](https://github.com/krunchyMonkey/krunchos-preview/settings/secrets/actions): confirm no repository secrets are present. No repository or environment secret is needed for this workflow.

The pinned source candidate is commit `d66bfa2e0eec3d977fd8e44b0c7df5cae8341103`. At audit time, source PR #67 was open and draft; its `static-validation`, `preview-checks`, and `e2e` checks all passed for that exact SHA. The candidate still needs explicit publication authorization; the owner's baseline sign-offs do not authorize deployment. This repository's deployment workflow itself has not been run, and the application remains unpublished.
