# Worked example: add an updater to an existing extension

This is an illustrative workflow and acceptance contract, not a downloadable updater. The agent generates project-specific code after inspecting the real target; this skill does not ship a prebuilt update engine.

## Input

```text
/just-update-unpacked-ext
Добавь этому локальному репозиторию расширения один update.cmd для Windows.
Источник — текущий origin и его основная ветка, готовое расширение в extension/.
Python допустим, Git у конечного пользователя требовать не нужно.
Скрипт должен работать из любой текущей директории. Не публикуй изменения.
```

The agent still verifies origin, default branch and extension/manifest.json. These values are not instructions to invent a URL or assume that a build exists.

## Generated repository layout

```text
my-extension/
  README.md                   # Quick Start and explicit prerequisites
  update.cmd                  # one independent launcher, Python payload embedded
  extension/
    manifest.json
    ...ready-to-load assets...
  tests/
    ...isolated updater regression tests...
```

## End-user workflow

1. Read the generated Quick Start and install the stated Python version if needed.
2. Download only the generated update.cmd into a dedicated local folder.
3. Run update.cmd. It fetches the configured extension snapshot, validates it and installs extension/ beside itself.
4. Once: Chrome → chrome://extensions → Developer mode → Load unpacked → select that extension/ directory.
5. Later: run the same update.cmd, then click Reload in Chrome. The extension path stays unchanged.

Runtime layout, distinct from the author's Git checkout:

```text
chosen-folder/
  update.cmd
  extension/                  # stable browser installation, managed by updater
  ...updater state/backup...   # explicit names chosen and documented per project
```

Do not double-click the updater in a developer checkout expecting it to replace source. In that layout extension/ is a nonempty unmanaged folder: refuse it, and explain how to use a dedicated installation folder. Adopting an existing manually installed folder requires explicit consent and a backup.

## Required observations from the actual test run

Record real results for: first install, update, unchanged revision, preservation after failed download, rejection of edited/unmanaged targets, failed replacement recovery, and a launch from another CWD with only update.cmd present. Include the installed revision and runtime-file hash comparison, not plausible-looking example success logs.

State separately:

- Generated and tested on the named OS.
- Files updated in the disposable or approved installation directory.
- Chrome activation is manual unless separately tested and authorized.

## Deliberately excluded

No background polling, browser helper, automatic package build, central package registry or cross-extension manager. No commit-derived version mutation unless explicitly requested. Additional launchers are generated only for requested platforms and need native verification.
