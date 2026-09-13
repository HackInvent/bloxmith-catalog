# HackInvent BloxSmith Catalog

[![Block baseline: 0.1.0](https://img.shields.io/badge/blocks-0.1.0-blue)](bloxsmith-1.0.9.json)
[![Verified BloxSmith: 1.0.9](https://img.shields.io/badge/BloxSmith-1.0.9-brightgreen)](bloxsmith-1.0.9.json)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)

Public, tester-approved compatibility information for BloxSmith blocks maintained by **HackInvent**. Validation happens in the private `bloxmith-blocs` workspace. This repository contains only public metadata and documentation, not the proprietary framework, test logs, credentials or user data.

## Current status

- **40 block releases at version 0.1.0**, verified with **BloxSmith 1.0.9** in bundled-block test installations.
- **0 managed-installation certifications**. Published source releases have not yet passed an archive-installation/runtime validation gate.
- Block versions and BloxSmith versions are separate. No support is inferred for other framework versions, even another patch release.

## Files

All JSON files live at the repository root. For each verified BloxSmith version `x.y.z`, the filenames are `bloxsmith-x.y.z.json` and `installable-x.y.z.json`; no category or version directories are used.

| File | Consumer | Meaning |
| --- | --- | --- |
| [index.json](index.json) | HackInvent website | Available exact framework versions, paths, counts and SHA-256 digests of the JSON documents |
| [bloxsmith-1.0.9.json](bloxsmith-1.0.9.json) | HackInvent website | The 40 approved releases and their **bundled-mode** test evidence |
| [installable-1.0.9.json](installable-1.0.9.json) | BloxSmith installer | Framework catalog schema v1; `blocks` is deliberately empty pending managed-installation validation |

The website feed uses `validated_releases`, not the installer's `blocks` array. **Do not give the website feed URL to the installer.** An informational `not_validated` flag inside an otherwise installable entry would not protect users: the framework ignores unknown informational fields. Unvalidated installable packages must therefore be absent from `blocks`.

## Website integration

1. Fetch `index.json` and look up the exact requested `bloxsmith_version`.
2. Resolve `compatibility.path` relative to the index URL. Optionally check its SHA-256 and byte length against the index before parsing it.
3. Display `validated_releases` with block version, release link, validation date and **validation scope**. The current scope must be described as "verified in bundled mode", not "certified for installation".
4. Enable a certified-installation action only for entries actually present in the separate `installable` catalog. That list is currently empty.

If a framework version is absent, show **not validated**. Do not silently fall back to a neighboring version or a GitHub `latest` release. The index is a website discovery document, not a recursive framework catalog: configure the installer with a direct `installable-<version>.json` URL.

Serve these files over HTTPS with `Content-Type: application/json; charset=utf-8`. Deploy the complete set of files as one revision so the index hashes and document contents agree. The repository itself does not configure the HackInvent website or GitHub Pages.

## Public evidence contract

All documents use integer `schema_version: 1`, independently of block/framework release numbers.

The index has `catalog_id`, `publisher` and a `catalogs` array. Each entry contains an exact `bloxsmith_version` and two descriptors, `compatibility` and `installable`. Each descriptor contains a `path` naming a JSON file at the repository root, SHA-256 of the complete JSON file bytes, `size_bytes` and `release_count`.

A website feed contains `document_type: "bloxsmith_compatibility_evidence"`, `catalog_id`, `certified_by`, an exact `bloxsmith_version`, `validation_scope: "bundled"` and `validated_releases`. Each release includes:

- `kind`, `title`, `version`, the **complete** manifest `bloxsmith_compatibility` list and `tested_with_bloxsmith`.
- `repository` and `release`: the exact tag, approved commit, public release URL and a commit-pinned source archive URL.
- Commit-pinned `manifest_url` and `compatibility_url` for the public block metadata.
- `source_fingerprint`, using the block test tooling's `block-source-v1` algorithm. **This is not the SHA-256 of a ZIP download.**
- `certification.status: "verified"`, `certification.scope: "bundled"` and `certification.managed_installation: "not_validated"`.
- `validation`: exact framework version/commit/source digest, run-start timestamp, passed status, installation mode, block-relative suite paths/count and the private report's digest. The report itself and its local paths/logs are not exported.

Evidence applies to the pinned block source and the recorded framework build. It is not a promise about every OS, browser, microphone, third-party service or modified build carrying the same version number. Missing certification is not proof of incompatibility.

The source archive URL is a source-code download, **not an approved installer artifact**. Before a release enters an installable catalog, its final ZIP must be built/frozen, its downloaded bytes hashed, and its installation and runtime behavior validated. Installer entries must match the embedded manifest and include `package.format`, `url`, `sha256`, `size_bytes` and `root` according to the framework's catalog contract. No fabricated checksum or placeholder download is published here.

## Maintenance and approval

The private workspace's portable generator reads explicit approved commit pins, checks clean block sources and their `compatibility.json`, and verifies the corresponding published GitHub releases and tagged files. A declaration submitted in a public block repository does not by itself grant HackInvent certification.

Regeneration is deterministic and initially uses the current approved block pins. It never automatically commits, pushes, creates releases, uploads archives or modifies the framework. Maintainers review changes before publication. Keep catalog history in Git; review removals or certification withdrawals explicitly and do not move existing block release tags. The generator does not delete older version files.

The private workspace runs `python3 -B tests/catalog_builder.py --write` to verify releases and regenerate local JSON, and `python3 -B tests/catalog_builder.py --verify-remote` to check existing files and published evidence. Contributors do not need access to that private workspace to consume this public catalog; official certification remains HackInvent's responsibility.

## License

Catalog data and documentation are provided under [Apache License 2.0](LICENSE). Refer to each block repository for its license and third-party notices. The BloxSmith framework retains its separate license.
