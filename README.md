# tickdown

A Windows desktop countdown timer built with .NET 10 and WinUI 3.

## Local launch

Run `pwsh -File ./run.ps1` with PowerShell 7.4 or newer. The script builds when
the selected output is missing or an application source, project, dependency
lock, XAML, asset, manifest, or repository build setting has changed. Unchanged
repeat launches skip build and restore. Add `-Configuration Release` when a
Release launch is needed.

## Isolated validation

Set `TICKDOWN_SETTINGS_DIRECTORY` to a disposable directory before launching
when testing. The application uses that directory for both timer and window
settings instead of your normal profile.

```powershell
$env:TICKDOWN_SETTINGS_DIRECTORY = Join-Path $env:TEMP ([guid]::NewGuid().ToString())
pwsh -File ./run.ps1
```

Pausing freezes the remaining countdown. Starting a paused timer resumes that
remaining time; starting a completed timer begins its full configured duration.
Stop preserves remaining time, while Reset restores the full duration.

## Quality tooling

Install the locked Markdown tooling with `npm ci --ignore-scripts`, then run
`npm run lint:md` and `npm audit --audit-level=low`. The linter includes Markdown
in hidden directories and excludes only generated `obj`, `bin`, and
`node_modules` directories. It uses the existing `.markdownlint.json` rules.

The pinned markdownlint CLI uses tinyglobby rather than the vulnerable braces
dependency. Parser overrides pin patched YAML, TOML, Markdown and math packages
while preserving the repository rules and the audit failure threshold.
