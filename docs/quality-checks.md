# Local quality checks

Use PowerShell 7, the exact .NET SDK selected by `global.json`, and Node.js 24.21.0 (the pinned CI version) on Windows.

```powershell
npm ci --ignore-scripts
dotnet restore TickDown.sln --locked-mode
pwsh -NoProfile -File scripts/tests/DependencyResolution.Tests.ps1
pwsh -NoProfile -File scripts/tests/StylePolicy.Tests.ps1
pwsh -NoProfile -File scripts/tests/CoverageQuality.Tests.ps1
pwsh -NoProfile -File scripts/check-coverage.ps1
pwsh -NoProfile -File scripts/tests/MutationQuality.Tests.ps1
pwsh -NoProfile -File scripts/check-mutation.ps1
dotnet build src/TickDown.csproj
pwsh -NoProfile -File scripts/check-quality.ps1
pwsh -NoProfile -File scripts/tests/QualityChecks.Tests.ps1
dotnet test TickDown.sln --configuration Release
```

The C# conventions and formatter contract are documented in
[`code-style.md`](code-style.md). The style-policy assertions prevent the Roslyn
and StyleCop import and member-qualification settings from diverging again.

The quality runner executes every independent check and prints a final summary.
`Pass` means the command succeeded without findings. `Findings` means a valid
package report contains outdated or vulnerable packages. `ToolError` means the
command failed, threw, or returned an incomplete or unsupported package report.
Any non-pass result makes the runner exit 1. Full command output is retained on
the console.

Both direct and transitive dependencies are scanned for vulnerabilities. Package
findings use structured JSON rather than localized English output. The existing
policy that any reported outdated direct package fails the check is preserved;
a newer major release is an update finding, not a vulnerability assertion.

Markdown is a separate pinned tool, not an MSBuild target. `npm run lint:md`
checks tracked source documentation and agent guidance, excluding generated
build and dependency directories. Its dependencies are locked in the npm lockfile.
The pinned `markdownlint-cli` wrapper uses the same markdownlint 0.41.1 engine
and unchanged rules. It includes hidden agent-guidance directories and preserves
the existing build/dependency exclusions. The old cli2 wrapper depended on braces,
which has no patched release for
[GHSA-vfj7-8cjw-p6xm](https://github.com/advisories/GHSA-vfj7-8cjw-p6xm).
Parser overrides pin patched smol-toml, KaTeX, js-yaml and markdown-it versions;
remove each override when the CLI's dependency ranges accept that fixed release.
The runner also executes `npm audit --audit-level=low` for these tooling dependencies.

The quality runner does not run tests or coverage implicitly. Run `dotnet test`
the maintained [coverage gate](coverage-quality.md), and the bounded
[countdown mutation gate](mutation-testing.md) separately so their results and
risk artifacts remain visible. A baseline failure must not be presented
as a clean result or hidden with a suppression.
