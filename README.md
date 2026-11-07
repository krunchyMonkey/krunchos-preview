# KrunchOS public preview

This repository is an isolated GitHub Pages host for a future public preview of KrunchOS. It does not change or deploy the production website at [krunchyMonkey/krunchymonkey-site](https://github.com/krunchyMonkey/krunchymonkey-site), and it has no dependency on private Tracing Guatemala services.

**The application has not been published.** The Pages workflow is manual-only and has not been run. No custom domain or DNS configuration is used.

## Preview location

The expected URL is <https://krunchymonkey.github.io/krunchos-preview/os/>. The workflow publishes only the verified `krunchos/dist` output under the `/os/` prefix in this repository's Pages artifact.

## Source and approval boundary

The static build originates from `krunchos/` in the main KrunchyMonkey site repository. Source PR [#67](https://github.com/krunchyMonkey/krunchymonkey-site/pull/67) adds static artifact checks and privacy checks; its `npm run check` includes type checks, lint, formatting, unit tests, build, and the `check:dist` allowlist/privacy/budget gate. The source commit must be on that repository's `main` branch before this workflow will accept it. This prevents deploying an unmerged PR head. The workflow checks that relationship and builds from the exact full commit SHA supplied at dispatch; it does not fetch or execute a workflow from the source repository.

At setup time, PR #67 is open and marked draft with a "do not merge" instruction, and PR [#66](https://github.com/krunchyMonkey/krunchymonkey-site/pull/66) is also open. Do not select either PR head as a release source. An owner must first review and merge the desired source changes; if that does not happen, the workflow's source-main check will fail. This preview setup does not authorize or perform any change to the source repository.

The workflow produces a GitHub Actions artifact from the pinned source commit, records per-file SHA-256 hashes, and checks those hashes again in the separate publishing job. The Actions run summary and retained provenance artifact identify the source commit and published file hashes. The build is checked by the source repository's `check:dist` script; only the static `dist/` tree is passed to Pages, at `os/`.

## Manually publishing a preview

Publishing requires an owner to explicitly authorize the first public deployment. Once the source commit is reviewed and merged:

1. Open **Actions → Deploy KrunchOS preview → Run workflow** in this repository.
2. Enter the full 40-character SHA of the approved commit on `krunchyMonkey/krunchymonkey-site` `main`. Confirm it is the intended commit and that its source PR and verification checks were reviewed.
3. Run the workflow manually. It verifies the commit is already on source `main`, runs the source checks and static build, validates `dist/index.html`, creates a file-hash manifest, and deploys only after the second job verifies that manifest.
4. Review the Actions run summary and `github-pages` deployment URL. Confirm the source SHA and manifest match the approved build before sharing the preview URL.

There is no push, pull-request, release, or schedule trigger; deployment requires an explicit `workflow_dispatch`. No production credentials, repository secrets, private networks, or private services are needed.

## Rollback

To roll back, manually run the workflow again with the full source commit SHA used by the last known-good preview. The workflow rebuilds that exact commit, reruns its checks, and deploys the resulting static files. Compare the new run's per-file SHA-256 manifest with the earlier successful run before considering the rollback complete. A failed build or hash check never proceeds to deployment.

## Public and synthetic data only

The preview is public and must contain only public, synthetic, or clearly conceptual demo data. Do not include real client/customer records, secrets, credentials, private endpoints, or data from Tracing Guatemala. The source `check:dist` privacy gate rejects known private-host, network, API, and key-material patterns, but review the source and build output before authorizing a public deployment; automated checks are not a substitute for review.

## Repository safeguards

The workflow grants read-only repository access to its build job. Only the `deploy` job receives `pages: write` and `id-token: write`; it also receives `actions: read` to retrieve the verified build artifact. The workflow pins GitHub-maintained Actions to reviewed release commits. The repository should use read-only default Actions permissions and an allowlist limited to the Actions used here. Protect `main` with pull requests and at least one approval so deployment-configuration changes are reviewed. Keep automatic merges disabled, and do not add repository secrets.

### Owner-side settings still required

The session can create repository contents but cannot change repository settings. Before the first deployment, an owner must:

1. Under **Settings → Pages**, set the publishing source to **GitHub Actions**. Do not select branch publishing or enter a custom domain.
2. Under **Settings → Environments**, create `github-pages` and restrict deployments to `main`. The workflow references this exact environment; no environment secrets are required.
3. Under **Settings → Branches**, protect `main` by requiring pull requests and at least one approval; dismiss stale approvals. This applies review to deployment configuration and all other changes on `main`.
4. Under **Settings → Actions → General**, allow only the pinned actions used in the workflow: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020`, `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`, `actions/download-artifact@d3f86a106a0bac45b974a628896c90dbdf5c8093`, `actions/upload-pages-artifact@56afc609e74202658d3ffba0e8f6dda462b719fa`, and `actions/deploy-pages@d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e`. Set the default workflow token permission to read-only and leave **Allow GitHub Actions to create and approve pull requests** disabled.
5. Keep repository and environment secrets empty and automatic merges disabled.

No application deployment is part of these setup steps. The first public deployment still requires explicit authorization after the source commit and static output have been reviewed.
