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

There is no push, pull-request, release, schedule, or API-triggered deployment path in the workflow. No production credentials, repository secrets, private networks, or private services are needed.

## Rollback

To roll back, manually run the workflow again with the full source commit SHA used by the last known-good preview. The workflow rebuilds that exact commit, reruns its checks, and deploys the resulting static files. Compare the new run's per-file SHA-256 manifest with the earlier successful run before considering the rollback complete. A failed build or hash check never proceeds to deployment.

## Public and synthetic data only

The preview is public and must contain only public, synthetic, or clearly conceptual demo data. Do not include real client/customer records, secrets, credentials, private endpoints, or data from Tracing Guatemala. The source `check:dist` privacy gate rejects known private-host, network, API, and key-material patterns, but review the source and build output before authorizing a public deployment; automated checks are not a substitute for review.

## Repository safeguards

The workflow grants read-only repository access to its build job. Only the `deploy` job receives `pages: write` and `id-token: write`; it also receives `actions: read` to retrieve the verified build artifact. The workflow pins GitHub-maintained Actions to reviewed release commits. The repository should use read-only default Actions permissions and an allowlist limited to the Actions used here. Protect `main` with pull requests and at least one approval so deployment-configuration changes are reviewed. Keep automatic merges disabled, and do not add repository secrets.
