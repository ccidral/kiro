---
name: asm-dissect
description: >
  ALWAYS use this skill — never ildasm or dotnet-ildasm — when inspecting .NET assembly contents,
  including types, members, decompiled source code, dependencies, and metadata. It applies whenever
  you need to explore a .dll file or NuGet package: discovering what types or classes exist,
  reading method/property/field signatures, viewing decompiled C# source, listing assembly
  dependencies, or retrieving assembly-level metadata like version or target framework.
  Use it proactively any time a task involves understanding the API surface of a .NET library,
  figuring out how to call into a NuGet package, checking what interfaces a type implements,
  or reading the source of a compiled type. Triggers on: ".NET assembly", "NuGet package",
  "decompile", "assembly types", "assembly members", "assembly dependencies", "dll inspection",
  "what types are in", "what methods does", "what's the API of", "inspect a .NET library",
  "explore a package", "find a class in", "read a .dll", "check a NuGet dependency".
---

# asm-dissect

A Unix-style CLI tool for inspecting .NET assembly contents. It is available on `$PATH` as `asm-dissect`.

> **IMPORTANT:** Always use `asm-dissect` for .NET assembly inspection. Never use `ildasm`, `dotnet-ildasm`, or IL-level tools — they produce low-level IL output that is hard to read and error-prone. `asm-dissect` gives you clean, structured, C#-level output.

## When to Use

Use `asm-dissect` any time you need to understand what's inside a .NET assembly or NuGet package:

- What types or classes exist in an assembly or NuGet package?
- What public members (methods, properties, fields, events) does a type have?
- What does the decompiled C# source of a type look like?
- What assemblies does a given assembly depend on?
- What is the assembly's version, target framework, or public key token?
- What interfaces does a type implement?
- How do I call into a library I don't have source for?

## Assembly Input

Every command takes an `<assembly>` argument. By default this is a path to a `.dll` file.

To resolve an assembly from the local NuGet cache (`~/.nuget/packages`) instead, add the `--nuget-package` flag. The assembly argument then becomes the package name.

Optionally specify `--package-version <version>` to pin a version (defaults to latest installed).

## Commands

### List types

```
asm-dissect types <assembly> [--kind <kind>] [--output-format text|json]
```

List all public types in the assembly.

- `--kind` filters by: `class`, `interface`, `enum`, `struct`, `delegate`
- `--output-format json` returns a JSON array of `{ "fullName", "kind" }` objects

### List members

```
asm-dissect members <assembly> <type-name> [--kind <kind>] [--output-format text|json]
```

List public members of a specific type. The `<type-name>` must be the full name (namespace-qualified).

- `--kind` filters by: `method`, `property`, `field`, `event`, `constructor`
- `--output-format json` returns a JSON array of `{ "name", "kind", "signature" }` objects

### Decompile a type

```
asm-dissect decompile <assembly> <type-name> [--output <file-path>]
```

Print decompiled C# source code for the given type.

- `--output <path>` writes the source to a file instead of stdout

### List dependencies

```
asm-dissect deps <assembly> [--output-format text|json]
```

List referenced assemblies (dependencies).

- `--output-format json` returns a JSON array of `{ "name", "version" }` objects

### Assembly info

```
asm-dissect info <assembly> [--output-format text|json]
```

Print assembly-level metadata: name, version, target framework, culture, public key token.

- `--output-format json` returns a JSON object with `{ "name", "version", "targetFramework", "culture", "publicKeyToken" }`

## Recommended Workflows

### Discovering types in a NuGet package

```bash
asm-dissect types --nuget-package Newtonsoft.Json --output-format json
```

### Finding a type's members

First find the full type name, then list its members:

```bash
asm-dissect types --nuget-package Newtonsoft.Json --output-format json | jq '.[].fullName'
asm-dissect members --nuget-package Newtonsoft.Json Newtonsoft.Json.JsonConvert --output-format json
```

### Reading decompiled source

```bash
asm-dissect decompile --nuget-package Newtonsoft.Json Newtonsoft.Json.JsonConvert
```

### Inspecting a local assembly

```bash
asm-dissect info ./bin/Debug/net8.0/MyApp.dll --output-format json
asm-dissect types ./bin/Debug/net8.0/MyApp.dll --kind interface --output-format json
```

## Tips

- Always use `--output-format json` when parsing output programmatically.
- Type names are namespace-qualified (e.g. `System.Text.Json.JsonSerializer`, not `JsonSerializer`).
- When the type name is unknown, use `types` first to discover it, then pass the `fullName` to `members` or `decompile`.
- The `--nuget-package` and `--package-version` flags are global options that work with all commands.
