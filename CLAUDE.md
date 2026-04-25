# HelloWorld — Project Guide

## Overview

Minimal ASP.NET Core 8 web API containerized with Docker. Exposes a root endpoint and a health check, intended for deployment on OSC (Open Shift / cloud platform).

- **Language:** C# / .NET 8
- **Framework:** ASP.NET Core Minimal API (`Microsoft.NET.Sdk.Web`)
- **Container:** Docker multi-stage build (SDK → ASP.NET runtime)
- **Port:** `8080` (configurable via `PORT` env var)

## Project Structure

```
HelloWorld.csproj   # SDK-style project file, targets net8.0
Program.cs          # Application entry point — all routes defined here
Dockerfile          # Multi-stage build; publishes Release build
```

## Build & Run

```bash
# Restore & build
dotnet build

# Run locally (listens on http://localhost:8080)
dotnet run

# Publish release artifact
dotnet publish -c Release -o ./out

# Docker build & run
docker build -t helloworld .
docker run -p 8080:8080 helloworld
```

## Testing

No test project exists yet. When adding tests:

- Create an `xunit` project: `dotnet new xunit -n HelloWorld.Tests`
- Add project reference to `HelloWorld.csproj`
- Use `Microsoft.AspNetCore.Mvc.Testing` (`WebApplicationFactory`) for integration tests
- Run: `dotnet test`

## Linting & Formatting

```bash
# Format code (requires .editorconfig or default rules)
dotnet format

# Verify formatting without modifying files (CI-safe)
dotnet format --verify-no-changes

# Analyze for warnings/errors
dotnet build -warnaserror
```

## Coding Conventions

- Minimal API style — keep routes in `Program.cs` until the file exceeds ~100 lines, then split into `RouteExtensions` classes
- No implicit usings — add `using` statements explicitly if needed (project does not set `<ImplicitUsings>enable</ImplicitUsings>`)
- Use `Results.Ok(...)` / `Results.Problem(...)` for structured responses rather than raw strings where possible
  - Exception: the root `/` endpoint intentionally returns a plain-text string greeting (`"Hi Jonas was here"`), not a JSON object; do not convert it to JSON or change the text without explicit instruction
- Health check at `/healthz` must always return `{ status: "healthy" }` with HTTP 200; do not remove or rename it
- Environment-driven configuration via `Environment.GetEnvironmentVariable` — do not hard-code port or external URLs
- **URL binding:** use `app.Urls.Add($"http://0.0.0.0:{port}")` before `app.Run()` — do **not** pass the URL directly to `app.Run(url)`, as that bypasses the env-var port logic
- Always bind to `0.0.0.0` (not `localhost`) so the app is reachable from outside the container
- `app.Run()` (no arguments) must always be the last statement in `Program.cs`

## Docker Notes

- Base image: `mcr.microsoft.com/dotnet/aspnet:8.0` (runtime-only, smaller)
- Build image: `mcr.microsoft.com/dotnet/sdk:8.0`
- `ENTRYPOINT` is `dotnet HelloWorld.dll` — update if the assembly name changes
- `PORT` env var must be honoured; default is `8080`; `EXPOSE` directive must match this default
- No `.dockerignore` exists yet — the build context currently includes `.git/`, `.claude/`, etc. Add one when Docker build times become a concern

## Git & PR Conventions

- **Commit messages:** follow [Conventional Commits](https://www.conventionalcommits.org/) — `type: description` where type is one of `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`.
- **Issue references:** append `(Closes #<issue>)` to the commit message subject when a commit resolves a GitHub issue.
- **Workflow:** open a PR for every change; merge via squash. Never commit directly to `main` except for initial scaffolding.
- **Branch naming:** `<type>/<short-slug>` e.g. `feat/add-version-endpoint`.
- No CI pipeline exists yet (no `.github/workflows`). Validation is manual: run build, format, and test checks locally before requesting review.

## Agent Team

- **developer** (`.claude/agents/developer.md`): Implements features, fixes bugs, writes tests
- **reviewer** (`.claude/agents/reviewer.md`): Reviews diffs for correctness, security, and style
