# TapRelay Remote Adapter Test Catalog

This repository is a declarative adapter fixture for TapRelay. The PC client reads
`catalog.json` from the raw GitHub URL and interprets the rules locally. It never
downloads or executes DLL, EXE, PowerShell, JavaScript, or other program code.

Default catalog URL:

`https://github.com/guliacer/TapRelay-remote-adapter-test/raw/refs/heads/main/catalog.json`

The same repository also publishes the declarative version manifest used by the
PC and Android update buttons:

`https://github.com/guliacer/TapRelay-remote-adapter-test/raw/refs/heads/main/version.json`

The manifest contains `version`, `downloadUrl`, and optional `releaseNotes`.
It is only surfaced when its version is newer than the caller's installed
version; a missing or invalid manifest never blocks notification delivery.

## Local smoke test

Create the configured test log and append one line per event:

```powershell
$logRoot = Join-Path $env:TEMP 'TapRelayRemoteAdapterDemo'
New-Item -ItemType Directory -Path $logRoot -Force | Out-Null
Add-Content -LiteralPath (Join-Path $logRoot 'events.log') -Value 'task=demo-100;status=done;message=Remote catalog loaded'
```

The application chip should be named `Remote Adapter Demo`. The same task ID and
status are emitted only once even when the log is scanned more than once.

## Submitting an adapter change

Add a validated rule to `catalog.json` through a pull request. A rule must provide:

- a new stable `appKey`, display name, brand color, and a real 256x256 PNG icon referenced by a relative `iconPath`;
- one or more `%LOCALAPPDATA%`, `%APPDATA%`, `%TEMP%`, `%USERPROFILE%`, `%PROGRAMDATA%`, `%PROGRAMFILES%`, or `%PROGRAMFILES(X86)%` log paths;
- a bounded regular expression with named `taskId`, `status`, and `message` captures;
- mappings for both `completed` and `failed`, plus `waiting` or `stopped` when the application exposes them.

The catalog also supports the allow-listed `parser: "workbuddy-acp"` rule for
WorkBuddy's ACP JSON-over-log format. It is still data-only: the installed TapRelay
binary owns the parser, while GitHub can update its watched log paths and activation
without shipping a new executable. The WorkBuddy rule uses
`%USERPROFILE%\\.workbuddy\\logs\\*.log` recursively and preserves the built-in
success, failure, permission, question, cancellation, and duplicate filtering
semantics.

The catalog also supports a data-only close reminder rule. The `leigod` rule
matches only `leigod.exe` below `%PROGRAMFILES(X86)%\\LeiGod_Acc`, so another
same-named executable in a different directory cannot trigger it. The PC binary
uses WMI process start/stop events, a bounded polling fallback, and the Windows
session-ending event. Its notification title and body are defined in the rule;
`actionUrl` is optional and must be an official `https://` or `weixin://` URI.
For a locally supplied official mini-program deep link, set
`TAPRELAY_LEIGOD_MINIPROGRAM_URL`; an empty value keeps the reminder functional
and opens TapRelay when the user taps it.
After the pull request is merged, a running TapRelay instance checks the raw catalog
every five minutes. It applies a valid change without restart or repackaging. If the
network is unavailable or the catalog fails validation, the last valid local cache
continues to be used.

This repository is a test source, not a general code execution channel. Applications
requiring custom database parsing, IPC, process hooks, or protocol logic still need a
normal TapRelay program update.
