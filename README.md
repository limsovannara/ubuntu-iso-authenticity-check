# Ubuntu ISO authenticity check

## Preparation status

This is an uncommitted review draft, not a published GitHub repository.
The workflow has not been dispatched or executed. No Ubuntu signature or ISO
verification has been performed by this draft.

Proposed repository name: ubuntu-iso-authenticity-check.

The only proposed repository files are this README and
.github/workflows/verify-ubuntu-iso.yml. Start a new repository from scratch;
do not fork, import, mirror, or copy any existing project or its Git history.
No project files, credentials, configuration, databases, media, or ISO belong here.

GitHub Actions must remain **disabled at repository level** throughout
preparation and publication, until execution is separately approved.
A manual trigger is not a substitute for disabling Actions.
No local VM or cloud runner may be started during preparation.

## Fixed verification scope

- Release: Ubuntu Server **24.04.5 LTS**.
- Architecture: **AMD64**.
- Exact ISO: ubuntu-24.04.5-live-server-amd64.iso.
- Image-signing primary fingerprint:
  **843938DF228D22F7B3742BC0D94AA3F0EFE21092**.
- Signing key: Ubuntu CD Image Automatic Signing Key (2012).
- Fingerprint authority:
  [Canonical image-verification documentation](https://documentation.ubuntu.com/security/software-integrity/image-verification/).
- The only network documents retrieved by workflow code:
  - <https://archive.ubuntu.com/ubuntu/project/ubuntu-archive-keyring.gpg>
  - <https://releases.ubuntu.com/24.04.5/SHA256SUMS>
  - <https://releases.ubuntu.com/24.04.5/SHA256SUMS.gpg>

The ISO is never downloaded, uploaded, mounted, or read by the runner.
GitHub's own runner/control-plane traffic still exists. This is not an
air-gapped environment and is not an isolation-qualification test.

## Publication gate — separate from execution

Do not upload this workflow to an Actions-enabled repository.

If an operator later prepares the repository:

1. Use the exact approved name and a personal account, with restricted write
   access and multi-factor authentication. Do not use an existing project
   repository, template, fork, self-hosted runner, or organization integration.
2. Establish repository-level **Disable actions** under Settings → Actions →
   General before uploading the workflow. Do not modify account-wide or
   organization-wide settings.
3. Reopen that setting and retain dated evidence that Actions remain disabled.
   When an existing authorized API client is available, additionally read
   GET /repos/OWNER/ubuntu-iso-authenticity-check/actions/permissions and require
   enabled = false. Do not install a client or create broad credentials just
   to obtain this evidence.
4. Publish only the two reviewed files. Do not configure secrets, variables,
   caches, deployment environments, webhooks, apps, or repository import.
5. Record the exact commit SHA and repository URL. Recheck the disabled setting
   after publication and confirm no workflow runs exist.
6. Return that evidence for review. Do not enable Actions or click Run workflow.

Repository creation and Actions settings use separate GitHub operations.
If the disabled-state requirement cannot be established, keep this as a local
uncommitted draft. Do not assume the new repository defaults to disabled.
Any brief Actions-enabled interval during repository creation must be separately
approved if the operator cannot avoid it under the preparation restriction.

## Execution gate — NOT authorized by preparation approval

Execution requires separate approval of the exact published workflow commit,
the repository settings, and one temporary GitHub-hosted Ubuntu VM.
Local VM creation/boot and unrelated testing remain prohibited.

Before requesting that approval, the operator must independently open Canonical's
fingerprint documentation on their Mac and compare all 40 hexadecimal characters
with the workflow's pinned primary fingerprint. Record this comparison.
Do not derive the expected fingerprint from the key download itself or rely
on an email address, display name, short key ID, or an unsigned checksum.

The draft runs one standard ubuntu-24.04 job with a five-minute timeout and
permissions: {}. It has only workflow_dispatch, no editable inputs, no
checkout, no third-party actions, and no automatic triggers or retry logic.

ubuntu-24.04 is an evolving image label, not an immutable image pin.
The job records the actual image version, tool paths, versions, package ownership,
and binary digests. It requires preinstalled GnuPG, gpgv, Python, curl, coreutils,
dpkg-query, and Ubuntu signing material. Missing prerequisites stop the job.
No package installation, sudo, downloaded executable, or fallback is allowed.

## Verification behavior after execution approval

1. Reject an unexpected runner, OS, architecture, repository name, or missing
   run identity. Record only allowlisted public environment information.
2. Check required preinstalled executables and package ownership. Require
   /usr/share/keyrings/ubuntu-archive-keyring.gpg, owned by ubuntu-keyring.
3. Copy the installed public keyring into a private runner-temporary directory
   and require the pinned key before any verification-document downloads.
4. Retrieve only the three fixed HTTPS documents, without redirects or TLS
   bypass, with size and time limits. Missing/invalid input stops the job.
5. Independently select the pinned image-signing key from the downloaded
   keyring and export only that primary key and its bound subkeys.
   Use explicit temporary GnuPG homes and keyrings. Never change system keyrings,
   default GnuPG configuration, owner trust, or CA stores.
6. Require zero exit codes from both gpgv and gpg. Check machine-readable
   signature identities, key status, expiration, signature algorithm, and
   timestamp. Reject unexpected signers, failure statuses, unsupported statuses,
   and ambiguous signature results. Do not retrieve keys automatically.
7. Parse the authenticated checksum document strictly. Reject malformed lines,
   duplicate filenames, or a missing exact ISO entry.
8. Record the authenticated checksum and public evidence in the log/job summary.
   Never emit raw downloaded content, user IDs, command diagnostics, or an
   environment dump. No artifact-upload action is used.
9. The result is **PUBLIC_MANIFEST_AUTHENTICATED**, not ISO verification.
   The local ISO remains **NOT_CHECKED_NO_ISO_ACCESSED**.

gpgv alone does not reject expired/revoked keys, so the draft also checks gpg
status and selected key metadata. Owner-trust warnings are not resolved by
importing owner trust or using an always-trust model; the externally authenticated
full fingerprint is the intended trust anchor.

The supported signature profile is intentionally conservative: one valid
signature under the pinned primary key, OpenPGP v4/RSA, and SHA-256, SHA-384, or
SHA-512. Changed upstream signers, additional signatures, unexpected status
formats, or algorithms require review. Never relax these checks during a failed
run just to obtain a green result.

## Final operator comparison on the Mac

After a separately approved successful run, inspect its repository, commit,
actor, run ID/attempt, signer evidence, and full authenticated checksum.
Use the existing macOS hashing tool on the actual downloaded ISO:

~~~sh
/usr/bin/shasum -a 256 "/absolute/path/to/ubuntu-24.04.5-live-server-amd64.iso"
~~~

The quoted path is an operator-supplied local path, not a workflow input.
Compare all 64 checksum characters with the authenticated result. Do not upload
the ISO or local filesystem path. If they differ, stop.

Only a matching local hash supports **MAC_ISO_HASH_MATCHED**. Authentication
does not certify the absence of vulnerabilities or authorize VM boot.

## Required evidence

Retain repository URL, exact workflow commit, run URL/ID/attempt, actor, UTC
timestamp, actual image version, OS/architecture, package/tool provenance,
installed/downloaded/selected keyring evidence, full primary/signing fingerprints,
document URLs/sizes/SHA-256 digests, verifier exit codes and validated status
fields, exact ISO filename, and authenticated ISO checksum.

Separately retain the operator's Canonical fingerprint comparison and local
ISO checksum comparison. Save the evidence before deleting the repository;
repository deletion requires its own approval and is not automated.

## Security review and costs

- No existing project code, credentials, data, Git history, or production
  connection is needed. No Mac package changes are needed.
- Trust remains in GitHub's hosted image/infrastructure, the account and reviewed
  workflow, Canonical's published fingerprint/key, and HTTPS for bootstrap.
  A compromised runner can falsify output; its logs are not independent
  certification against compromise of the hosting provider.
- Public repository contents and run logs are public. Only public verification
  material and allowlisted public execution metadata may appear there.
- permissions: {} removes configurable token scopes; it is not a claim that
  GitHub provides no internal platform credentials or control-plane access.
- Repository write/admin access can change the workflow or enable Actions.
  Restrict that access. Reviewed commit identity and repository-level disabling
  are the preparation gates, not merely the manual trigger.
- No runner execution, negative-input testing, signature verification, or ISO
  verification has occurred during preparation. Static review is not a
  cryptographic pass or proof of runtime compatibility.
- Standard hosted compute in a public repository is currently free. No caches,
  uploaded artifacts, larger/custom runners, or billing changes are proposed.
  Preparation has used no Actions compute. Recheck pricing before execution.

Official references:

- [Runner inventory](https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2404-Readme.md)
- [Actions permissions API](https://docs.github.com/en/rest/actions/permissions)
- [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)
- [GnuPG gpgv behavior](https://www.gnupg.org/documentation/manuals/gnupg/gpgv.html)
- [GnuPG machine-readable formats](https://github.com/gpg/gnupg/blob/master/doc/DETAILS)
- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
