# Releases

What a correct release is: its version number, its betas, its registration. How to make
one is a procedure, in [`ct-agents`](https://github.com/control-toolbox/ct-agents); how
the registries work is in
[`ct-registry/RELEASING.md`](https://github.com/control-toolbox/ct-registry/blob/HEAD/RELEASING.md).
For the changelog of a release, see [`changelog.md`](changelog.md).

Identifiers: `REL1`–`REL7` (see [`README.md`](../README.md#rule-identifiers)).

Two registries are involved: **General**, the public Julia registry, holds the stable
releases; **ct-registry**, the private registry of the organization, holds the betas.

---

## REL1 — Version numbers follow SemVer as Pkg reads it

**Rule.** The breaking component of a version is its **leftmost non-zero** component, as
Pkg reads a `[compat]` entry. The next version is chosen from the kind of change since
the last release:

| Last version | Breaking change | New feature | Bug fix only |
| --- | --- | --- | --- |
| `X.Y.Z`, `X ≥ 1` | `(X+1).0.0` | `X.(Y+1).0` | `X.Y.(Z+1)` |
| `0.Y.Z`, `Y ≥ 1` | `0.(Y+1).0` | `0.Y.(Z+1)` | `0.Y.(Z+1)` |

A package does not release `0.0.Z` versions: every one of them is breaking for Pkg, so
its first release is `0.1.0`. A change is breaking when it has a
[`CHG3`](changelog.md#chg3--breaking-entries-appear-in-both-files-with-a-migration-block)
entry.

**Why.** [SemVer](https://semver.org/spec/v2.0.0.html) §4 says only that anything may
change while the major version is zero. Pkg is precise: `compat = "0.4"` means
`[0.4.0, 0.5.0)`, so in `0.Y.Z` a minor bump is breaking and a patch bump is not. A
breaking change released as a patch is installed silently by every dependent package.

**Check.** `judgment`: whether a change is breaking. `exact` (to be tooled): a release
whose changelog entry has a `💥 Breaking Changes` section bumps the breaking component.
`heuristic`: the breakage tests of the release, on the **stable** column (the last release
of each dependent package): a dependent that passes on the base and fails with the release
is broken by it, so the version is breaking. A dependent that already fails on the base
says nothing about the release.

---

## REL2 — A beta takes the `-beta` suffix and a new number

**Rule.** A beta is the version number given by
[`REL1`](#rel1--version-numbers-follow-semver-as-pkg-reads-it) with the suffix `-beta`;
no other pre-release suffix is used (`-rc`, `-beta.2`, `-alpha`). Every release, beta or
stable, has a **new triple** `X.Y.Z`: strictly greater than the triple of every version
already released, in both registries. A stable never takes the triple of a beta.

The kind of change is measured from the last release for a beta, and from the **last
stable** for a stable:

| Last versions | Change | Next version |
| --- | --- | --- |
| `0.4.2` | beta, no breaking change | `0.4.3-beta` |
| `0.4.2`, `0.4.3-beta` | stable, no breaking change since `0.4.2` | `0.4.4` |
| `0.4.2` | beta, breaking change | `0.5.0-beta` |
| `0.4.2`, `0.5.0-beta`, `0.5.1-beta` | stable, breaking since `0.4.2` | `0.5.2` |

**Why.** SemVer orders `0.5.0-beta` before `0.5.0`, as a candidate of it: a stable
`0.5.0` would read as the final version of a tree that may differ from the beta's. A
workflow that uses both registries must never see two trees under one triple.

**Check.** `exact` (to be tooled): the triple is above every version of the package in the
`Versions.toml` of General and of ct-registry; the suffix is empty or `-beta`.

---

## REL3 — A version jump in General is explained by betas

**Rule.** Since the last version of the package in General, `X.Y.Z`, the valid increments
are those of the [`REL1`](#rel1--version-numbers-follow-semver-as-pkg-reads-it) table.
Any other stable number is a **jump**. A jump is allowed only when it comes from
[`REL2`](#rel2--a-beta-takes-the--beta-suffix-and-a-new-number): every skipped triple is
a beta in ct-registry. It is then approved on the registration pull request of General,
by a comment `[merge approved]`.

**Why.** General's AutoMerge stops on a jump, which usually reveals a typo
(`0.4.2 → 0.6.0`). With betas the jump is expected, but only betas explain it.

**Check.** `exact` (to be tooled): compare the new version with the `Versions.toml` of
General and of ct-registry.

---

## REL4 — A stable release resolves from General alone

**Rule.** The package and each of its weak dependencies resolve in an environment whose
only registry is General. No `[compat]` bound, of a dependency or of a weak dependency, is
satisfied by betas only.

**Why.** The CI of the packages uses ct-registry (`use_ct_registry: true`), so it passes
with a beta dependency. General's AutoMerge rejects such a release, after it is tagged.

**Check.** `exact` (to be tooled): the CTActions workflow that resolves the package from a
fresh depot holding General only, run on every pull request that sets a stable version.

---

## REL5 — Version, tag, registry and GitHub release agree

**Rule.** The `version` of `Project.toml`, the registered version and the tag `vX.Y.Z`
(with the `v`) are the same, and the tag points to the registered commit. Every release,
beta included, has a GitHub release on that tag; a beta's is marked pre-release. The
notes of a stable release list the changes since the **previous stable**.

**Why.** Pkg installs the registered commit, a reader browses the tag, a user reads the
GitHub release: they must describe the same tree. Notes since the previous beta would
hide, for a user of General, everything the betas brought.

**Check.** `exact` (to be tooled): `Project.toml` at the tag, the registry entry and the
release agree.

---

## REL6 — The registration bot is mentioned once

**Rule.** For a stable release, the JuliaRegistrator bot is mentioned (an `@` followed by
its name) **exactly once**: in one comment of the release issue, posted when the release
is ready — or, when `main` has moved past the release commit, in one comment on that
commit, since the bot registers the head of `main` when asked on an issue. It is never mentioned in an issue description, an issue template, a pull
request, a commit message, a GitHub release or a quote, and never for a beta. Prose that
needs to refer to it says "the Registrator mention".

**Why.** The bot reacts to its mention wherever it appears, in a description or in a code
block, and every reaction is a registration attempt: an issue opened with the mention in
its description is registered before its notes are final.

**Check.** `exact`: the release issue and the release commit hold one mention, in a
comment; no pull request mentions the bot, in its text or its commit messages.

---

## REL7 — A release ships a change to what users get

**Rule.** Since the previous release of the same kind — the last stable for a stable, the
last release for a beta — the release changes what a user of the package gets:

| Changed | A release? |
| --- | --- |
| `src/`, `ext/` (code and docstrings) | yes |
| `Project.toml`: dependencies, `[compat]`, extensions | yes |
| `docs/` | yes: the documentation of the stable version is built from its tag |
| `test/`, `.github/`, repository files (`.gitignore`, `_typos.toml`, …) only | no |
| `CHANGELOG.md`, `BREAKING.md` only | no |

A stable that follows betas with no change since the last beta is a release: it is
compared with the last stable.

**Why.** A release that ships nothing new registers a second version of the same package,
and makes every dependent re-resolve for nothing.

**Check.** `exact` (to be tooled): the files changed between the tag of the previous
release of the same kind and the release commit.
