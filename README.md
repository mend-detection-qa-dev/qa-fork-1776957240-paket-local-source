# paket-local-source

Paket probe: local path source alongside public NuGet source.

## Feature exercised

Exercises Paket's local path source type, where `paket.lock` contains a `remote: ./local-packages` entry using a relative filesystem path instead of a URL.

## What this tests

- Two `remote:` entries in `paket.lock`: one HTTPS URL and one relative local path
- Mend UA parser handling of `remote: ./local-packages` without treating it as a URL
- Packages from a local source appear in the dependency tree with `dependencyType: NUGET`
- `storage: none` mode (no `.paket/` storage directory)
- Mixed-source project: `Newtonsoft.Json` and `Serilog` from public NuGet; `MyCompany.LocalLib` from local path

## Expected dependency tree

- **Group**: Main (default, no explicit group keyword)
- **Packages from `https://api.nuget.org/v3/index.json`**:
  - `Newtonsoft.Json 13.0.3` — direct, no children
  - `Serilog 3.1.1` — direct, no children
- **Packages from `./local-packages`**:
  - `MyCompany.LocalLib 2.0.0` — direct, no children, sourced from local path remote
- All three packages appear as top-level direct dependencies of `MyProject`
- No framework restrictions
- No transitive dependencies declared

## Probe metadata

| Field         | Value                        |
|---------------|------------------------------|
| pattern       | local-source                 |
| target        | local                        |
| target fw     | net8.0                       |
| storage mode  | none                         |
| created       | 2026-04-22                   |

## Detection failure modes targeted

The critical risk is that a UA parser resolves `./local-packages` relative to the working directory at scan time (not at lockfile location), resulting in a path resolution error that causes the local package to be silently dropped from the dependency tree. This probe validates that `MyCompany.LocalLib` is present in the tree output regardless of how the parser resolves the path.
