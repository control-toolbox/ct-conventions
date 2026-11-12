# Changelog & Breaking Changes

How changes are tracked and communicated. For version numbering, see
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Identifiers: `CHG1`–`CHG6` (see [`README.md`](../README.md#rule-identifiers)).

---

## CHG1 — Two companion files

**Rule.** Every package maintains **two companion files** at the repository root:

- `CHANGELOG.md` — the full, chronological record of every notable change.
- `BREAKING.md` — a focused companion listing **only** breaking changes (and
  near-breaking changes, for clarity). Cross-referenced from every CHANGELOG entry.

**Why.** Keeping them separate lets downstream maintainers grep `BREAKING.md` in one
place without reading every entry in the full log.

**Check.** `exact` (to be tooled): both files exist at the repository root.

---

## CHG2 — The `CHANGELOG.md` format

**Rule.** Based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/). The file
header:

```markdown
# Changelog

All notable changes to <PackageName> will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
```

Each release entry:

```markdown
## [X.Y.Z] - YYYY-MM-DD

### 💥 Breaking Changes   ← only if present; see CHG3

### ✨ New Features

### 🐛 Bug Fixes

### 🛠 Enhancements

### 🔄 Refactoring

### 🧪 Testing

### 📚 Documentation

### 📦 Dependencies

### ✅ Compatibility

- **No breaking changes**: <one-line reason>. See [BREAKING.md](BREAKING.md).
```

Omit sections that have nothing to say. The `### ✅ Compatibility` footer is
**always present**: it either states "No breaking changes" or points to the
`### 💥 Breaking Changes` subsection above it.

| Emoji | Section | Use when |
| --- | --- | --- |
| 💥 | Breaking Changes | Public API removed, renamed, or semantically changed |
| ✨ | New Features | Additive public API or behavior |
| 🐛 | Bug Fixes | Behavioral corrections |
| 🛠 | Enhancements | Non-additive improvements (performance, error messages, …) |
| 🔄 | Refactoring | Internal restructure with no observable API change |
| 🧪 | Testing | Test-only changes |
| 📚 | Documentation | Docs, docstrings, guides — no source change |
| 📦 | Dependencies | Compat bumps, added or removed dependencies |
| ✅ | Compatibility | Summary footer — always the last subsection of an entry |

**Why.** One format across the ecosystem: a reader finds the same sections in the same
order in every package, and the footer answers "can I upgrade?" at a glance.

**Check.** `exact`: for `## [Unreleased]`, the section names and order and `✅
Compatibility` last are checked by `release.jl lint-changelog` ([`CHG6`](#chg6--unreleased-stays-in-format-checked-by-the-lint)).
To be tooled: the header, and the older entries.

---

## CHG3 — Breaking entries appear in both files, with a migration block

**Rule.** A breaking change appears **twice**: once in `CHANGELOG.md`, once in
`BREAKING.md`. In `CHANGELOG.md`, the `### 💥 Breaking Changes` subsection includes:

1. What was removed or changed, and why.
2. A **migration block** showing old and new code side by side, with `# Before` /
   `# After` Julia comments.
3. A cross-reference to `BREAKING.md`.

```markdown
### 💥 Breaking Changes

#### `Foo.bar` renamed to `Foo.baz`

- **`Foo.bar` is removed** from the public API. Use `Foo.baz` instead.
- **Why**: `bar` was ambiguous with the `bar` exported by `CTBase.Core`.

**Migration**:

\`\`\`julia
# Before
result = Foo.bar(x)

# After
result = Foo.baz(x)
\`\`\`

See [BREAKING.md](BREAKING.md).
```

In `BREAKING.md`, mirror it with a concise summary + the same migration block.

**Why.** The migration block is what a downstream maintainer actually needs: the exact
edit to make.

**Check.** `exact` (to be tooled): every `💥 Breaking Changes` entry has a migration
block and a matching entry in `BREAKING.md`.

---

## CHG4 — Near-breaking changes get a note in `BREAKING.md`

**Rule.** Near-breaking changes — a renamed internal symbol, a changed default value, a
display change — get a `## Non-breaking note (vX.Y.Z)` entry in `BREAKING.md` even
though no migration is required:

```markdown
## Non-breaking note (0.5.3)

- **`Foo`: default `:precision` option changed from `1e-6` to `1e-8`.**
  The option name and override mechanism are unchanged; only the computed
  default value changes. **No breaking change**: any caller already setting
  `:precision` explicitly is unaffected. No migration required.
```

**Why.** This keeps `BREAKING.md` the single authoritative source for any reader trying
to understand whether a version bump may affect them.

**Check.** `judgment`: whether a change is near-breaking.

---

## CHG5 — Retroactive bootstrap: a baseline entry

**Rule.** When a package has several existing releases but no `CHANGELOG.md` yet, do not
try to reconstruct every past entry. Instead:

1. Identify the **last stable release** (last non-beta tag, or the last release
   the team considers the public reference).
2. Add a single entry for it, labelled as the baseline:

   ```markdown
   ## [X.Y.Z] - YYYY-MM-DD — baseline

   This is the reference version. No changelog was maintained before this point;
   use `git log` for earlier history. Breaking changes from this version onward
   are tracked in [BREAKING.md](BREAKING.md).
   ```

3. From the **next release onward**, apply the full convention above.

The same applies to `BREAKING.md`: start a fresh file alongside the baseline entry and
track only changes from that point. The baseline is in place before the next release
is tagged.

**Why.** A reconstructed history is costly and unreliable; a declared baseline is
honest and immediately useful.

**Check.** `exact` (to be tooled): a package with release tags but no `CHANGELOG.md`
fails [`CHG1`](#chg1--two-companion-files).

---

## CHG6 — `[Unreleased]` stays in format, checked by the lint

**Rule.** Between two releases, every change goes under `## [Unreleased]` of
`CHANGELOG.md`, in the sections of [`CHG2`](#chg2--the-changelogmd-format), in their
order, with `### ✅ Compatibility` last; and its counterpart in `BREAKING.md` goes under
`## Unreleased` (a breaking change, with its migration block, `CHG3`) or
`## Non-breaking note (Unreleased)` (`CHG4`). The release names them after the version.

Whoever edits `CHANGELOG.md` or `BREAKING.md` — a person or an agent, in any pull request,
a release or not — runs the lint and leaves it green:

```bash
julia --startup-file=no CTActions/scripts/release/release.jl lint-changelog .
```

The lint only reports. An agent fixes the format itself; where the fix needs a judgment
(which section an entry belongs to, whether it is breaking), it decides and says so in its
report. Older entries are never edited, whatever their format.

**Why.** A release reads its version from `[Unreleased]`: a `### Breaking` instead of
`### 💥 Breaking Changes` is not seen, and the version comes out wrong without any signal.
A heading the tool cannot read below `[Unreleased]` merges an older entry into it. Caught in
the pull request that writes it, a format problem costs a line; at the release, it stops it.

**Check.** `exact`: `release.jl lint-changelog` (CTActions). The release gate reports its
findings (ℹ️ `CHG2`) on every pull request that is not a release; `release.jl facts` and
`request` refuse a release until it is green.
