# Internal Android APK delivery

This fork retains upstream application code. The manual `internal-apk.yml` workflow republishes the exact successfully built upstream release APK after an arm64-only slice and stable internal signing. It does not rebuild or execute candidate code in the secret-bearing job.

## Approved initial source

- Upstream source: `Codename-11/hermes-relay`, commit `62de6b51289d933a3f8d68841652b044130aa72f`.
- Source build: existing `ci-required.yml` / `all-final` workflow, defined at `52f3f7565811828824d42bc9b432296271a2f9c3`.
- Expected package: `com.axiomlabs.hermesrelay.sideload`.
- Expected version: `1.18.1-sideload`, version code `59`.
- Target devices: Android 8+ (`minSdk 26`), `arm64-v8a`.

The signer intentionally pins the initial source and metadata. Review and update those pins together for a subsequent source version; require a strictly higher delivered version code. An unchanged-version rerun is a reinstall, not a new update version.

## Stable internal signing identity

- Environment: `internal-apk-signing`, custom deployment branch policy allows only `main`.
- Alias: `hermes-relay-internal`.
- Certificate SHA-256: `3176ae55da9fad0f50f935cb4e60346d3ca95d9a9529ba811e9a3191985d3aba`.
- Private PKCS12 keystore and password are encrypted GitHub environment secrets. No private key is committed, printed, uploaded as an artifact, or stored as a repository-wide secret.
- Never regenerate or rotate this key without explicit authorization. The secrets are write-only; preserve the protected environment when maintaining the fork.

This certificate differs from upstream/store and old ephemeral CI signatures. An existing installation under this package with another signer cannot update into this line. Export any app settings before removing an incompatible installation. Later internal builds must preserve package and certificate.

## Signing boundary

The workflow only runs manually on trusted `main`. It has no checkout, package installation, source script execution, inherited run shell, container, services, or matrix. The secret-bearing Python is inline in the reviewed workflow. The only third-party action is immutable-pinned artifact upload after signing; no signing values are passed to it.

The workflow verifies the successful trusted source run, exact-source artifact name, ZIP paths/duplicates/CRC, package/version, a single pinned signing certificate, arm64 ABI, and 16 KiB alignment. It compares every remaining non-signature ZIP entry digest to the original after dropping only other native ABIs. It publishes a prerelease, downloads the resulting asset back, and compares its checksum. Signing evidence records both source and signing workflow SHAs.

Before executing any edited signer, independently review the complete workflow shape and current exact diff, not only the embedded Python. The main-only environment is a branch boundary, not a defense against malicious changes already merged to main.

## Evidence limits

The upstream all-final build includes assembly, focused tests, lint and release smoke. Signing and payload verification do not prove launchability, connection compatibility, background notifications or device permissions. Those require hosted emulator or consenting physical-phone tests. No emulator/device proof is claimed for a build-only delivery.

A complete delivery includes the APK, `signing-evidence.json`, `signature-verification.txt`, `apk-badging.txt`, checksums, successful source run URL and signing run URL. Verify the downloadable asset again before sending it to a tester.
