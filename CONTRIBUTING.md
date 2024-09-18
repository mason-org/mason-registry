# Contributing guide

## Table of Contents

<!--toc:start-->
- [Contributing guide](#contributing-guide)
  - [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Schema](#schema)
- [Testing](#testing)
- [The anatomy of a package](#the-anatomy-of-a-package)
- [Package specification](#package-specification)
  - [`name`](#name)
  - [`description`](#description)
  - [`homepage`](#homepage)
  - [`licenses`](#licenses)
  - [`languages`](#languages)
  - [`categories`](#categories)
  - [`source`](#source)
  - [`bin`](#bin)
  - [`share`](#share)
  - [`opt`](#opt)
- [Expressions](#expressions)
- [Examples](#examples)
  - [Common fields](#common-fields)
  - [Cargo](#cargo)
  - [Composer](#composer)
  - [Gem](#gem)
  - [GitHub release assets](#github-release-assets)
  - [GitHub build from source](#github-build-from-source)
  - [Golang](#golang)
  - [LuaRocks](#luarocks)
  - [npm](#npm)
  - [Nuget](#nuget)
  - [opam](#opam)
  - [PyPI](#pypi)
  - [Open VSX](#open-vsx)
<!--toc:end-->

> The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT
> RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14][bcp14],
> [RFC2119][rfc2119], and [RFC8174][rfc8174] when, and only when, they appear in all capitals, as shown here.

# Introduction

* Make sure to follow the [naming guidelines](#name).
* Refer to the [common fields example](#common-fields) for a good starting point for a new package.
* Refer to the different [examples](#examples) and/or existing package definitions for further guidance.
* Testing a package MUST be done locally prior to creating a PR. See [testing](#testing) for more information.

> [!TIP]
> Use the [YAML language server](https://mason-registry.dev/registry/list#yaml-language-server) combined with the
> [schemastore schema](https://json.schemastore.org/mason-registry.json) to get diagnostics and autocompletion (see
> [Schema](#schema)).

# Schema

Package definitions are validated against a well-defined [JSON schema](https://github.com/mason-org/registry-schema).
The full schema is hosted on <http://schemastore.org/>.

> [!TIP]
> Use [b0o/SchemaStore.nvim](https://github.com/b0o/SchemaStore.nvim) and the [YAML language
> server](https://mason-registry.dev/registry/list#yaml-language-server) to integrate these schemas in Neovim. This
> gives you diagnostics and autocompletion inside the editor when editing package definitions:
> 
> <img src="https://user-images.githubusercontent.com/6705160/230375252-40dfcd78-dcd3-43c4-8967-c7452384b818.png" height="100" />

# Testing

Testing a package locally can be achieved by configuring the `mason.nvim` client to source package definitions locally
from your filesystem.

> [!IMPORTANT]
> In order for `mason.nvim` to be able to parse the YAML files you must have `yq` installed on your system. Tip: install
> `yq` (`:MasonInstall yq`) from the core registry before testing.

Take note of the path where you have `mason-org/mason-registry` cloned on your file system. To configure `mason.nvim` to
source packages from there you'll use the `file:` protocol, like so:

```lua
require("mason").setup {
  registries = {
    "file:~/dev/mason-registry"
  }
}
```

Before continuing, make sure Mason has been properly configured to source packages locally by opening the `:Mason`
window and pressing `g?` to open the help view:

<img src="https://github.com/mason-org/mason-registry/assets/6705160/952e1b0f-6396-492e-9270-e72c26800a98" height="100" />

> [!TIP]
> You can emulate different platforms ("targets") by providing the `--target=` option to `:MasonInstall`. For example:
> <pre><code>:MasonInstall --target=linux_arm64 my-package</code></pre>  
> Note that this is only a soft emulation and only impacts how the package definition is parsed.

# The anatomy of a package

Packages are defined following a [well-defined specification](#package-specification). Package definitions are hosted as
separate YAML files that MUST be located at `packages/<package-name>/package.yaml`.

Package sources are identified via a [purl][purl] identifier. Each package source (purl) MUST contain a version
component specifying the latest available version, e.g `pkg:github/rust-lang/rust-analyzer@2023-04-04`.

Package versions are automatically kept up-to-date via [Renovate][renovate].

# Package specification

The following is a rough outline of the package definition schema:

```yaml
name: string
description: string
homepage: URL
licenses: SPDXLicense[]
languages: string[]
categories: Category[]

source:
    id: string
    [key: string]: any

bin?:
    [executable: string]: string
share?:
    [share_location: string]: string
opt?:
    [opt_location: string]: string
```

## `name`

The package name MUST be unique. The name of a package MUST follow the following naming scheme:

1. If the upstream package name is sufficiently unambiguous, or otherwise widely recognized, that name MUST be used.
1. If the upstream package provides a single executable with a name that is sufficiently unambiguous, or otherwise
   widely recognized, the name of the executable MUST be used.
1. If either the package or executable name is ambiguous, a name where a clarifying prefix or suffix is added SHOULD be
   used.
1. As a last resort, the name of the package should be constructed to best convey its target language and scope, e.g.
   `json-language-server` for a JSON language server.

## `description`

Short description of the package. The description SHOULD be sourced from the upstream package directly.

Longer descriptions MUST be split on multiple lines, as to not exceed the max line length (120).

Example:

```yaml
description: |
  Lorem ipsum dolor sit amet, officia excepteur ex fugiat reprehenderit enim labore culpa sint ad nisi Lorem pariatur
  mollit ex esse exercitation amet. Nisi anim cupidatat excepteur officia.
```

> [!TIP]
> To automatically format the description across multiple lines, run `:setlocal textwidth=120`, visually select the
> description and press `gw` (`:help gw`).

## `homepage`

The homepage of the package. The homepage SHOULD be a public website if available, otherwise it MUST be a URL to the
source code.

The URL MUST be a [well-formed URL][rfc1738]. The URL scheme MUST be either `http` or `https`.

## `licenses`

List of licenses associated with this package. MUST contain at least one entry.

The license MUST be a [SPDX-compatible](https://spdx.org/licenses/) license identifier. Should the package use a license
not available as a SPDX identifier, the license "proprietary" (all lower case) MUST be used.

Examples:

- `MIT`
- `Apache-2.0`
- `GPL-3.0-only`

## `languages`

The languages the package targets. MAY be empty. A language is an arbitrary string (e.g., `"Rust"`). The casing of the
string MUST be the same as other references to the same language in other package definitions, i.e. it's an error if
package A specifies `Javascript` and package B specifies `JavaScript`.

## `categories`

The categories the package belongs to. MAY be empty. If not empty, each entry MUST be one of:

- `Compiler`
- `DAP`
- `Formatter`
- `LSP`
- `Linter`
- `Runtime`

## `source`

The source of the package. The `source` entry contains all necessary information to properly install the package. At the
very minimum it MUST contain an `id` property. The `id` property MUST be a [purl][purl]-compatible package identifier.
The purl identifier MUST contain a version component.

The source object MAY contain additional properties to support installation.

Examples:

```yaml
source:
    id: pkg:npm/typescript-language-server@2.0.0
```

```yaml
source:
    id: pkg:github/rust-lang/rust-analyzer@2022-12-05
```

## `bin`

The executables the package provides. The key is the canonical name of the executable, and the value is either (i) a
relative path to the executable from the package directory, or (ii) an expression that delegates path resolution (e.g.,
`npm:typescript-language-server` or `cargo:rust-analyzer`), or (iii) an [expression](#expressions).

On Unix systems, a symlink is created. On Windows, a wrapper batch `.cmd` executable is always created.

Example:

```yaml
bin:
    typescript-language-server: npm:typescript-language-server
    rust-analyzer: bin/rust-analyzer
```

## `share`

The architecture independent files the package provides.

The mapping MUST either (i) link a single target file to a single source file, or (ii) link a target directory to a
source directory, or (iii) an [expression](#expressions).

This creates symlinks (`uv_fs_symlink`) on all platforms.

Example:

```yaml
share:
    # Links $MASON/share/jdtls/lombok.jar -> <package>/lombok.jar
    jdtls/lombok.jar: lombok.jar
    # Links $MASON/share/jdtls/plugins/ -> <package>/plugins/**/* (i.e. all files within the target directory)
    jdtls/plugins/: plugins/
```

> [!IMPORTANT]
> The contents of linked files MUST be compatible with all machines, regardless of hardware architecture.

## `opt`

The optional, add-on, contents of a package. This is for example useful in situations when a package provides auxiliary
binaries that should not be linked to the "global" Mason `bin/` directory.

The mapping MUST either (i) link a single target file to a single source file, or (ii) link a target directory to a
source directory, or (iii) an [expression](#expressions).

This creates symlinks (`uv_fs_symlink`) on all platforms.

Example:

```yaml
opt:
    # Links $MASON/opt/solang/llvm15.0/LICENSE -> <package>/doc/LICENSE
    solang/llvm15.0/LICENSE: doc/LICENSE
    # Links $MASON/opt/solang/llvm15.0/ -> <package>/llvm15.0/**/* (i.e. all files within the target directory)
    solang/llvm15.0/: llvm15.0/
```

# Expressions

When specified, a component of a package definition may include expressions. These expressions can only be used in
string values, and are denoted by `{{expr}}`. This allows for dynamically evaluating values, when needed.

Example:

```yaml
# ...
source:
    id: pkg:github/rust-lang/rust-analyzer@v1.0.0
    asset:
        - target: darwin_x64
          file: rust-analyzer-darwin_x64_{{ version | strip_prefix "v" }}.tar.gz
          bin: rust-analyzer-darwin_x64
          some_other_bin: rust-fmt-darwin_x64
        - target: linux_x64
          file: rust-analyzer-linux_x64_{{ version | strip_prefix "v" }}.tar.gz
          bin: rust-analyzer-linux_x64
          some_other_bin: rust-fmt-linux_x64

bin:
    # This will be evaluated to either "rust-analyzer-darwin_x64" or "rust-analyzer-linux_x64", depending on which
    # platform the package is being installed on.
    rust-analyzer: "{{source.asset.bin}}"
    rustfmt: "{{source.asset.some_other_bin}}"
```

Expressions use basic Lua syntax with the additional ability to pipe values to a limited set of transformation
functions. All expressions are evaluated in a context, where values are accessed through normal variable access.

[bcp14]: https://tools.ietf.org/html/bcp14
[purl]: https://github.com/package-url/purl-spec
[renovate]: https://github.com/renovatebot/renovate
[rfc1738]: https://www.rfc-editor.org/rfc/rfc1738
[rfc2119]: https://tools.ietf.org/html/rfc2119
[rfc8174]: https://tools.ietf.org/html/rfc8174

# Examples

## Common fields

The following fields are common for all packages and are subject to the same requirements.

Refer to the following sections for a detailed description:

- [`name`](#name)
- [`description`](#description)
- [`homepage`](#homepage)
- [`licenses`](#licenses)
- [`categories`](#categories)

Example:

```yaml
---
name: lua-language-server
description: A language server that offers Lua language support - programmed in Lua.
homepage: https://github.com/LuaLS/lua-language-server
licenses:
  - MIT
languages:
  - Lua
categories:
  - LSP
```

> [!IMPORTANT]
> The document MUST start with a YAML header notation (`---`).

---

## Cargo

Example:

```yaml
source:
  id: pkg:cargo/rnix-lsp@0.2.5

bin:
  rnix-lsp: cargo:rnix-lsp
```

<details><summary>Example: Specify features</summary>

To specify the features to install, use the `features` qualifier.

Example:

```yaml
source:
  id: pkg:cargo/flux-lsp@0.8.40?features=lsp,cli
```

</details>

<details><summary>Example: Git source</summary>

To install a cargo package from a git source you may specify the `repository_url` qualifier. This will by default target
tags in the provided git repository (i.e. `cargo install --tag <TAG>`). To target commits instead (i.e. `cargo install --rev
<REV>`), provide an additional `&rev=true` qualifier.

Example:

```yaml
source:
  id: pkg:cargo/flux-lsp@0.8.40?repository_url=https://github.com/influxdata/flux-lsp
```

</details>

<details><summary>Example: Specify supported platforms</summary>

You MUST provide the `supported_platforms` field if the package is only supported on certain platforms.

Example:

```yaml
source:
  id: pkg:cargo/flux-lsp@0.8.40
  supported_platforms:
    - linux_x64_gnu
    - linux_arm64_gnu
```

</details>

---

## Composer

Example:

```yaml
source:
  id: pkg:composer/vimeo/psalm@5.4.0

bin:
  psalm: composer:psalm
  psalm-language-server: composer:psalm-language-server
```

---

## Gem

Example:

```yaml
source:
  id: pkg:gem/standard@1.26.0

bin:
  standardrb: gem:standardrb
```

<details><summary>Example: Specify supported platforms</summary>

You MUST provide the `supported_platforms` field if the package is only supported on certain platforms.

Example:

```yaml
source:
  id: pkg:gem/standard@1.26.0
  supported_platforms:
    - linux_x64_gnu
    - linux_arm64_gnu
```

</details>

---

## GitHub release assets

Note: Downloaded release assets are automatically unpacked (e.g. if it's a `.tar` file it's unpacked in its download
location).

<details><summary>Example: Platform dependent release assets</summary>

When multiple, platform dependent, release assets are provided you MUST register an entry for each applicable platform.
This is done by providing a list of assets. The ordering of this list is important as clients may be target to more than
one platform and entries appearing first in the list have precedence.

When this source is parsed by the client, the list is "unwrapped" to the very first entry whose `target` applies to the
current system.

Example:

```yaml
source:
  id: pkg:github/LuaLS/lua-language-server@3.6.18
  asset:
    - target: darwin_arm64
      file: lua-language-server-{{version}}-darwin-arm64.tar.gz
    - target: darwin_x64
      file: lua-language-server-{{version}}-darwin-x64.tar.gz
    - target: linux_arm64_gnu
      file: lua-language-server-{{version}}-linux-arm64.tar.gz
    - target: linux_x64_gnu
      file: lua-language-server-{{version}}-linux-x64.tar.gz
    - target: win_x86
      file: lua-language-server-{{version}}-win32-ia32.zip
    - target: win_x64
      file: lua-language-server-{{version}}-win32-x64.zip
```

It's common that platform-dependent assets contain different files and different folder structures. In order to
facilitate linking executables at a later stage you may provide additional, arbitrary, fields. The following example
adds a `bin` field to each entry, which is later used in a [expression](#expression) to link the executable.

Example:

```yaml
source:
  id: pkg:github/LuaLS/lua-language-server@3.6.18
  asset:
    - target: darwin_arm64
      file: lua-language-server-{{version}}-darwin-arm64.tar.gz
      bin: lua-language-server
    - target: win_x64
      file: lua-language-server-{{version}}-win32-x64.zip
      bin: lua-language-server.exe

bin:
  lua-language-server: "{{source.asset.bin}}"
```

</details>

<details><summary>Example: Single, platform independent, release asset</summary>

Example:

```yaml
source:
  id: pkg:github/Dart-Code/Dart-Code@v3.62.0
  asset:
    file: dart-code-{{ version | strip_prefix "v" }}.vsix
```
</details>

<details><summary>Example: Downloading multiple assets</summary>

Example:

```yaml
source:
  id: pkg:github/LuaLS/lua-language-server@3.6.18
  asset:
    file:
      - lua-language-server-{{version}}
      - lua-language-server-{{version}}.sha256
      - LICENSE
```
</details>

<details><summary>Example: Change asset download location</summary>

By default, assets are downloaded in the root directory of the package directory. You MAY change the download location
by appending it to the file name itself with a `:` prefix.

If the download location ends with a `/` the file will be downloaded in that directory, otherwise it's a filename.

Example:

```yaml
source:
  id: pkg:github/lua/lua@5.1.0
  asset:
    file:
      # download "lua-language-server-{{version}}" to "bin/lua-language-server-{{version}}"
      - lua-language-server-{{version}}:bin/

      # download "lua-formatter-{{version}}" to "bin/lua-format"
      - lua-formatter-{{version}}:bin/lua-format

      # download "license" to "LICENSE.txt"
      - license:LICENSE.txt
```
</details>

> [!IMPORTANT]
> Linux binaries are commonly compiled for GNU systems. These binaries MUST be associated with a `_gnu` target, e.g.
> `linux_x64_gnu`.

---

## GitHub build from source

Note: Build scripts run on the platform's default shell. On Unix this is `bash`, on Windows it's `pwsh`.

Note: By default, Renovate is configured to look for new releases for `pkg:github` sources. However, when building from
source, the repository most likely doesn't provide GitHub releases, but instead uses normal git tags. To ensure that
Renovate picks up new versions, you MUST provide a datasource override via a comment (see example below).

Example:

```yaml
source:
  # renovate:datasource=github-tags
  id: pkg:github/stoplightio/vscode-spectral@v1.1.2
  build:
    run: |
      npm exec yarn@1 install
      npm exec --package=yarn@1 'node make package'

bin:
  spectral-language-server: node:dist/server/index.js
```

<details><summary>Example: Platform-dependent build scripts</summary>

Sometimes the build script cannot be expressed in a shell-agnostic way. You MUST then provide a list of entries with the
appropriate targets. The ordering of this list is important as clients may be target to more than one platform and
entries appearing first in the list have precedence.

When this source is parsed by the client, the list is "unwrapped" to the very first entry whose `target` applies to the
current system.

Example:

```yaml
source:
  id: pkg:github/vala-lang/vala-language-server@1.0.0
  build:
    - target: unix
      run: |
        meson -Dprefix="$PWD" build
        ninja -C build install
    - target: win
      run: |
        meson -Dprefix="($pwd).path" build
        ninja -C build install
```
</details>

---

## Golang

Example:

```yaml
source:
  id: pkg:golang/golang.org/x/tools/gopls@v0.11.0

bin:
  gopls: golang:gopls
```

<details><summary>Example: Specifying additional package path</summary>

Use the subpath component to specify a sub-path of a golang package.

Example:

```yaml
source:
  id: pkg:golang/golang.org/x/tools@v0.7.0#cmd/goimports
```
</details>

---

## LuaRocks

Example:

```yaml
source:
  id: pkg:luarocks/luacheck@1.1.0

bin:
  luacheck: luarocks:luacheck
```

<details><summary>Example: Specifying a server</summary>

Use the `repository_url` qualifier to specify a different server (i.e. `luarocks install --server`).

Example:

```yaml
source:
  id: pkg:luarocks/luaformatter@scm-1?repository_url=https://luarocks.org/dev
```
</details>

<details><summary>Example: dev target</summary>

Use the `dev` qualifier to specify a dev target (i.e. `luarocks install --dev`).

Example:

```yaml
source:
  id: pkg:luarocks/teal-language-server@dev-1?dev=true
```
</details>

---

## npm

Example:

```yaml
source:
  id: pkg:npm/typescript-language-server@3.3.1

bin:
  typescript-language-server: npm:typescript-language-server
```

<details><summary>Example: Additional npm dependencies</summary>

Some packages may require additional npm dependencies to be installed in the same location. This can be achieved by
providing the `extra_packages` field.

Example:

```yaml
source:
  id: pkg:npm/typescript-language-server@3.3.1
  extra_packages:
    - typescript
```

Packages provided in `extra_packages` are passed as-is to npm, so they may require version constraints such as
`typescript@4`.
</details>

---

## Nuget

Example:

```yaml
source:
  id: pkg:nuget/fsautocomplete@0.58.2

bin:
  fsautocomplete: nuget:fsautocomplete
```

---

## opam

Example:

```yaml
source:
  id: pkg:opam/ocaml-lsp-server@1.10.2

bin:
  ocamllsp: opam:ocamllsp
```

---

## PyPI

Example:

```yaml
source:
  id: pkg:pypi/yamllint@1.30.0

bin:
  yamllint: pypi:yamllint
```

<details><summary>Example: Adding extra specifiers</summary>

To add "extra" specifiers to a pypi package (i.e. `pip install python-lsp-server[all]`), use the `extra` qualifier.

Example:

```yaml
source:
  id: pkg:pypi/python-lsp-server@1.7.2?extra=all
```
</details>

<details><summary>Example: Additional pypi dependencies</summary>

Some packages may require additional pypi dependencies to be installed in the same location. This can be achieved by
providing the `extra_packages` field.

Example:

```yaml
source:
  id: pkg:pypi/yapf@0.32.0
  extra_packages:
    - toml
```

Packages provided in `extra_packages` are passed as-is, so they may require version constraints such as `toml==4`.
</details>

---

## Open VSX

[Open VSX](https://open-vsx.org/) is an open-source registry for VS Code extensions.

Example:

```yaml
source:
  id: pkg:openvsx/vscjava/vscode-java-debug@0.55.0
  download:
    file: vscjava.vscode-java-debug-{{version}}.vsix
```

<details><summary>Example: Platform-dependent file</summary>

If the Open VSX package provides platform-dependent files they need to be registered explicitly. In addition to `target`
and `file`, the `target_platform` field must be defined and correspond to an OpenVSX platform identifier (e.g.
`linux-x64`).

```yaml
source:
  id: pkg:openvsx/BroadcomMFD/cobol-language-support@2.1.1
  download:
    - target: linux_x64
      file: BroadcomMFD.cobol-language-support-{{ version }}@linux-x64.vsix
      target_platform: linux-x64
    - target: darwin_arm64
      file: BroadcomMFD.cobol-language-support-{{ version }}@darwin-arm64.vsix
      target_platform: darwin-arm64
```

</details>
