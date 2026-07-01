# bundled-pm-change — Ruby 3.4.10 stdlib-extraction probe

## Feature exercised

Ruby 3.4.10 is a language runtime release that extracts several
standard-library gems (`bigdecimal`, `mutex_m`, `ostruct`) into
standalone Bundler dependencies. Under Ruby 3.4, Bundler must
resolve these gems explicitly; earlier Rubies provided them
silently via the interpreter. This probe exercises Bundler's
dependency resolution under a `ruby "~> 3.4.10"` constraint
alongside transitive gems (`rack`, `faraday`) that historically
relied on the now-extracted stdlib members.

## Pattern

`bundled_pm_change` — language-runtime release triggers a PM
behavior change. Category: `bundled_pm_change`.

Ruby 3.4.10 release reference:
https://www.ruby-lang.org/en/news/2025/04/09/ruby-3-4-2-released/

## Expected dependency tree

### Direct dependencies (`main` group)

| Gem           | Version    | Why selected |
|---------------|------------|--------------|
| `bigdecimal`  | 3.1.9      | Extracted from stdlib in Ruby 3.4; must appear as registry dep |
| `mutex_m`     | 0.3.0      | Extracted from stdlib in Ruby 3.4; must appear as registry dep |
| `ostruct`     | 0.6.1      | Extracted from stdlib in Ruby 3.4; must appear as registry dep |
| `rack`        | 3.1.14     | Core web gem; transitive graph stress-tests resolver |
| `faraday`     | 2.12.2     | HTTP client with net-http / uri transitive chain |

### Direct dependencies (`dev` group)

| Gem    | Version | Why selected |
|--------|---------|--------------|
| `rake` | 13.2.1  | Standard dev tooling |

### Direct dependencies (`test` group)

| Gem        | Version | Why selected |
|------------|---------|--------------|
| `minitest` | 5.25.5  | Standard test framework |

### Transitive chain

```
faraday 2.12.2
  └─ faraday-net_http 3.4.0
       ├─ net-http 0.6.0
       │    └─ uri 1.0.3
       └─ (net-http >= 0.5.0)
  └─ json 2.10.2
  └─ logger 1.6.6
```

`rack` has no transitive gems beyond itself in this minimal probe
(rack 3.x removed rack-session from the core package).

### Mend failure modes this probe catches

1. **Stdlib-extracted gems dropped**: Mend may incorrectly treat
   `bigdecimal`, `mutex_m`, `ostruct` as stdlib builtins and omit
   them from detection results under Ruby 3.4.
2. **RUBY VERSION section ignored**: Mend may not capture
   `ruby 3.4.10p0` from the `RUBY VERSION` lockfile section,
   causing the runtime constraint to be lost from `project_metadata`.
3. **faraday transitive chain broken**: `faraday-net_http` depends
   on `net-http` which in turn requires `uri`. Under Ruby 3.4, these
   gems are mandatory Bundler resolutions rather than stdlib fallbacks;
   Mend must detect the full chain.
4. **Wrong parent links**: Each stdlib-extracted gem must have a
   direct-dep parent link to root, not appear as an orphan or under
   the wrong parent.

## Mend config

**Bucket A — default-emit.** `ruby-bundler` has no dynamic version
detection from the manifest. Every probe MUST ship `.whitesource`
with `scanSettings.versioning`.

For this probe the Ruby version is pinned exactly because the
pattern specifically targets Ruby 3.4.10 behavior:

```json
{
  "scanSettings": {
    "versioning": {
      "bundler": ">=2.6 <3",
      "ruby": "3.4.10"
    }
  }
}
```

`ruby "3.4.10"` exact pin (not a range) ensures Mend's `install-tool`
provisions the exact patch release that changed stdlib bundling
behavior. `bundler ">=2.6 <3"` covers the Bundler major series
that first shipped full Ruby 3.4 support (Bundler 2.6.0 landed
alongside Ruby 3.4 GA).

No additional `whitesource.config` UA config is required for this
probe — resolver-driven detection is lockfile-based (default).

## Probe metadata

| Field             | Value |
|-------------------|-------|
| `pm`              | `bundler` |
| `pattern`         | `bundled_pm_change` |
| `pm_version_tested` | `2.6.9` |
| `ruby_version`    | `3.4.10` |
| `generated_at`    | `2026-07-01T03:28:29Z` |
| `resolver_sha`    | `b5e271b6a951f20bf55563f5bd046bf7d47524ad` |
| `resolver_fetched_at` | `2026-07-01T03:28:29+00:00` |
