# Placard plugins

> **Archived repository.** Placard’s official modules are developed and released in their own repositories. Use the repository for the module you want to install, contribute to, or report an issue with.
>
> [Board](https://github.com/placard-app/module-board) · [Day Out](https://github.com/placard-app/module-day-out) · [Decide Together](https://github.com/placard-app/module-decide-together) · [Menus](https://github.com/placard-app/module-menus) · [Pick](https://github.com/placard-app/module-pick)
>
> The original source history and releases are available here for reference.

Official modules for [Placard](https://github.com/JobiJoba/Placard), built and
released independently from a shared pnpm workspace. Install only the modules you
need; each has its own version, manifest and GitHub Releases. No npm publishing or
separate module server is required.

## Modules

| Module | What it adds | Host requirements | Documentation |
| --- | --- | --- | --- |
| **Decide Together** (`places.decide-together`) | Household shortlist voting and revealed favorites | SDK 1, `persistent-data`, household mode | [Decide Together guide](plugins/places.decide-together/README.md) |
| **Day Out** (`places.day-out`) | Private outings around a saved place, nearby suggestions and MCP editing | SDK 1, `persistent-data` | [Day Out guide](plugins/places.day-out/README.md) |
| **Board** (`places.board`) | Printable travel boards from 22 places, with editing and 3D preview | SDK 1, `persistent-data` | [Board guide](plugins/places.board/README.md) |
| **Menus** (`places.menus`) | Structured menus, item photos and MCP editing on place details | SDK 0.2.0+, `place-sections` and `persistent-data` | [Menus guide](plugins/places.menus/README.md) |
| **Pick** (`places.pick`) | A random place from your library, with category, list, tag, opening-hours and distance filters | SDK 0.2.0+, module uploads and the host web bridge | [Pick guide](plugins/places.pick/README.md) |

## Install in Placard

1. Open **Settings → Module marketplace → Install from GitHub**.
2. Paste the full URL for the module you want:

   | Module | Install URL |
   | --- | --- |
   | Decide Together | `https://github.com/JobiJoba/plugins#places.decide-together` |
   | Day Out | `https://github.com/JobiJoba/plugins#places.day-out` |
   | Board | `https://github.com/JobiJoba/plugins#places.board` |
   | Menus | `https://github.com/JobiJoba/plugins#places.menus` |
   | Pick | `https://github.com/JobiJoba/plugins#places.pick` |

3. Enable the installed module and make it visible for your account as needed.
   Menus appears on place details; Pick adds a navigation page.

The GitHub installer must support the `#module.id` selector for repositories with
multiple modules. Keep that selector: a bare repository URL selects the
repository's latest release, which may belong to another module. Each full module
URL selects that module's stable releases and updates.

For hosts without selector support, download the desired module's `.placard-module`
or `.zip` asset from [Releases](https://github.com/JobiJoba/plugins/releases) and
upload it through the marketplace. Both extensions contain the same package;
install either one. A local build can be uploaded the same way without publishing
a GitHub Release.

The module IDs remain `places.menus` and `places.pick`, so replacing an existing
installation retains its identity and associated settings/data. See the module
guides for storage and backup details.

## Develop

Requires **Node.js 24+** and **pnpm 12.4.1**. Run these commands from the repository root:

```sh
pnpm install
pnpm typecheck
pnpm test
pnpm build
```

Work on one module with its workspace package name:

```sh
pnpm --filter @placard-official/places.menus typecheck
pnpm --filter @placard-official/places.menus test
pnpm --filter @placard-official/places.menus build
pnpm --filter @placard-official/places.pick build
```

`pnpm test` runs tests only in packages that define a test script. Both Menus and Pick
have unit tests. Menus also has optional browser
tests against a real Placard host; see its [test setup](plugins/places.menus/README.md#browser-tests-against-core).

Each build creates:

| Path | Contents |
| --- | --- |
| `plugins/<moduleId>/pack/` | Staged manifest, server bundle, web bundle and packaged assets |
| `plugins/<moduleId>/dist/<moduleId>-<version>.placard-module` | Installable module archive |
| `plugins/<moduleId>/dist/<moduleId>-<version>.zip` | The same archive with a ZIP extension |

Builds recreate `pack/`; generated files are ignored by Git. All packages share the
root `pnpm-lock.yaml`. Regular builds do not need a Placard core checkout. Both
modules bundle server dependencies and use the host's React/router web bridge
rather than shipping a second React runtime.

## Repository layout

```text
plugins/
  places.menus/        # Menus manifest, source, tests and build script
  places.pick/         # Pick manifest, source and build script
scripts/release.mjs    # Prepare versions or validate a module release tag
.github/workflows/    # Workspace CI and per-module release publishing
```

Each module's README covers usage, host compatibility, permissions, MCP tools and
implementation details. CI installs from the frozen lockfile, then typechecks,
tests and builds the workspace.

## Release one module

Run from a clean checkout. Choose the next version for the module you are releasing;
the following example prepares Menus `0.1.1`:

```sh
pnpm release:prepare places.menus 0.1.1
pnpm --filter @placard-official/places.menus typecheck
pnpm --filter @placard-official/places.menus test
pnpm --filter @placard-official/places.menus build

git add plugins/places.menus/package.json plugins/places.menus/placard.module.json
git commit -m "Release places.menus 0.1.1"
git push origin main
git tag places.menus@0.1.1
git push origin places.menus@0.1.1
```

`release:prepare` updates the package and manifest versions. It does not commit,
tag or publish. Versions must use `major.minor.patch`; the helper does not accept
prerelease suffixes. To validate a tag locally without changing files:

```sh
node scripts/release.mjs --check-tag places.menus@0.1.1
```

Pushing a `places.*@*` tag triggers the release workflow. It checks that the tag,
manifest ID and both versions agree, then typechecks, runs any defined unit tests,
builds only that module and publishes both archives as GitHub Release assets.
Browser tests are separate from this workflow.

For Pick, substitute `places.pick`, `@placard-official/places.pick` and its next
version, and use its unit-test command. Releasing one module does not require
bumping or releasing another.

## Add a module

1. Create `plugins/<moduleId>/` with a `package.json`, `placard.module.json`, source
   and build script. The workspace includes `plugins/*` automatically.
2. Use a unique manifest ID matching the directory. The release helper currently
   accepts IDs matching `places.[a-z][a-z0-9-]*` and a `major.minor.patch` version.
3. Add `typecheck` and `build` package scripts, plus `test` if the module has tests.
   Keep the package and manifest versions in sync.
4. Make the build emit `<moduleId>-<version>.placard-module` and `.zip` in the
   module's `dist/` directory. Existing build scripts show the packaging and host
   web bridge setup.
5. Document installation, host requirements, permissions, usage and persistence in
   the module README, then add it to the tables above.
6. Run the workspace checks before creating the module's first release tag.

## License

See [LICENSE](LICENSE) for the GNU Affero General Public License v3 text.
