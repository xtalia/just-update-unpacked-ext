---
name: just-update-unpacked-ext
description: "Use when adding an updater to an unpacked Chrome extension."
version: 0.1.0
author: Serg Sinist (xtalia), Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [chrome, extensions, unpacked, updater, single-file]
---

# Just Update Unpacked Ext

Add a small, project-specific updater to an unpacked Chromium extension. This is a personal agent workflow, not a package manager or a universal binary. Generate and test the updater in the target project; loading this skill alone does not update anything.

## When to Use

- "Add an update.cmd to this extension", "обновление распакованного расширения", or an equivalent request to avoid manual downloads and extraction.
- Maintain an existing generated updater without changing the installed directory.
- Do not use for Chrome Web Store publishing, managed browser policies, a background update service, or installing arbitrary native applications.

## Prerequisites

- An explicitly identified extension project and permission to edit it. Discover the local repository and remote with `terminal`; inspect files with `search_files` and `read_file`. Ask only for missing choices that cannot be discovered.
- Inspect the working tree before editing. Preserve unrelated work. Do not push, publish private source, change repository visibility, or install into the user's browser merely because an updater was requested.
- Identify the trusted repository, branch or tag, extension subdirectory, ready-to-load files, and intended stable installation path. Read `manifest.json` and project build documentation. If multiple plausible manifests exist, resolve the ambiguity instead of selecting the first.

## How to Run

Invoke `/just-update-unpacked-ext` with a local project path or repository URL and the desired OS. Read [the worked example](references/example.md) for the expected output contract.

Default to the current OS, not every platform. On Windows generate an independent `update.cmd`; add `update.ps1` only when requested. On Linux use `update.sh`; on macOS use `update.command`. A filename is not proof of native compatibility.

## Procedure

1. **Resolve the installable input.** Confirm the source actually contains a ready-to-load extension at the selected revision. For source-only repositories, stop and offer a prebuilt release/artifact or an explicit reviewed build step. Never silently run downloaded npm scripts, shell hooks, or setup programs.
2. **Choose the smallest transport.** Prefer a fixed-repository snapshot fetched over HTTPS and handled by Python 3 standard library; no pip or Git dependency for the generated updater. An archive downloaded/extracted internally is fine: the user must not handle it. If an existing project already uses Git, a Git-based updater is also valid; document Git as a prerequisite, refuse a dirty tree and use fast-forward-only updates, never reset/clean/stash automatically. Resolve the exact remote commit before consuming its snapshot to avoid branch movement mid-download.
3. **Make each entry independent.** Embed static reviewed runtime code and source configuration in the one advertised launch file; a CMD may embed its Python payload, not depend on a neighboring PS1/PY. Require a supported Python version explicitly in Quick Start, detect it before mutation, and give an actionable error if missing. Do not install runtimes or require admin rights. Never fetch and execute an updater engine from a mutable URL. If the user requires no Python, choose an explicit native implementation and test it rather than disguising the dependency in a blob.
4. **Fix the path contract.** Launch without arguments installs/updates one dedicated stable sibling directory relative to the script, not the shell CWD. Prefer `update.cmd` beside `extension/`, outside the managed directory. Do not replace the running launcher or rewrite a developer checkout by default. For an existing installation retain its current path; require consent for migration. Record owner/repository/ref/subdirectory and an inventory outside the replaced tree, bind them to the resolved target, and reject a mismatched or nonempty unmanaged target. First adoption of an existing folder requires a reviewed backup and explicit user choice.
5. **Stage and validate before mutation.** Use an HTTPS allowlist appropriate to the chosen provider, TLS verification, network timeouts and bounded download/unpacked sizes. Resolve the archive root, allow only the chosen extension subtree, reject traversal, absolute/drive paths, links/reparse points, duplicate/case-colliding paths and unsupported file types. Never blindly extract all archive members. Reject symlink/reparse targets and ancestors. Validate the manifest and every declared local runtime asset; do not treat schema version alone as proof of a usable package.
6. **Preserve the working copy.** Use a lock against concurrent runs, compare current managed files with the previous inventory, and refuse local edits or unexpected files rather than deleting them. Download and validate fully before replacing anything. Stage on the same filesystem, retain the previous installation and state, and recover if replacement/state update fails. Do not describe multiple directory/state renames as a globally atomic transaction. Leave recoverable backups and actionable errors if rollback cannot finish; never erase the last good copy.
7. **Expose only useful behavior.** No-argument launch updates; `--check` reports the installed/remote revision without changing installed files or state; `--help` describes prerequisites, destination and local-edit policy. Re-running at the same revision is a no-op only after validating local integrity. Print old/new revision, the stable path and the remaining Chrome action. Propagate failure as a nonzero exit code; preserve errors when the file is double-clicked without hanging noninteractive runs.
8. **Document and test the generated file.** Add a short Quick Start to the extension README: runtime prerequisite, first run, exact Load unpacked path, subsequent update and Reload, backup/recovery, trust in the configured upstream, and what is not supported. Make no browser profile changes. Generate only the files needed for the agreed platform and contract.

## Verification

Use `terminal` for real execution, not invented output. Use the project's test workflow, or small stdlib Python fixtures if it has none. Write a failing test before implementing each behavior.

- Copy only the promised launch file into a clean disposable directory (including spaces and non-ASCII characters), then run it from a different CWD. No source-tree sidecars may be available.
- With a controlled local archive/transport fixture, exercise install v1, update to v2, unchanged no-op, removed upstream files and correct runtime hashes. Keep any loopback HTTP fixture override test-only; production remains HTTPS-only.
- Verify missing runtime, network interruption, corrupt archive/manifest, missing declared assets and malicious archive paths leave the good installation untouched.
- Verify unmanaged targets, edited/extra local files, symlinks/reparse points, concurrent runs and injected replacement failure are refused or recovered without data loss. Assert recovery state and backup contents, not just exit codes.
- Verify `--check` and `--help` do not mutate the installation or its state.
- When authorized and the remote exists, smoke-test a fresh install from the real configured source in a disposable directory and compare its files with that exact remote commit. Do not install into the user's browser to prove a download.
- Exercise each promised launcher on its native OS; otherwise clearly label that platform unverified. A Bash syntax check on Windows is not a macOS test.
- Report changed paths, exact tests/results, installed-file revision and whether browser activation remains manual. A written script without an executed update is not a tested updater.

## Pitfalls

- Updating disk files is not Chrome activation. Say to use Reload on `chrome://extensions`; content-script changes can also require reloading affected tabs. Automatic reload requires a separately agreed integration, not browser-profile or Preferences edits.
- Preserve upstream numeric `version` by default. If commit display is requested, prefer optional `version_name` such as `1.2.0 · abc1234` in the installed copy; track this explicit transformation in integrity comparisons. Do not insert a hash into numeric `version`, mutate source on every update, or claim a version bump reloads Chrome.
- The updater trusts future code from the configured upstream. HTTPS and archive validation do not audit that code. Do not store access tokens in generated launchers or public examples; stop for a secure auth plan if the source is private.
- Do not add a registry, daemon, GUI, self-update channel, scheduler or cross-extension manager for this personal-use workflow. Add features only when requested.
