# Screen media-list patch

This branch contains a patch artifact. The engine fix is also available in
[draft PR #2279](https://github.com/lexmount/moli/pull/2279).
Apply the patch to this branch before building:

```sh
git apply --check patches/stylesheet-screen-media-lists.patch
git apply patches/stylesheet-screen-media-lists.patch
```

Base: `a75ca04c98020b46cdd74ac2f81dd5b3fe57b99f` on upstream main.
A clean application check also passed on release `v1.1.20`
(`77c7f1fdc22f4e3bef941c86c731a5dbe85440ed`).

The shared parser gate recognizes unconditional `screen` and `all` members of
comma-separated media lists. It uses the existing cssparser version to preserve
comments, escaped identifiers, strings and nested comma boundaries. Conditional
media-query handling retains its existing policy.

The observed release failure is a parser-created link with
`media="screen,print"` inside an iframe: no stylesheet request, `link.sheet === null`,
and the default body margin. The same document works directly or with
`media="screen"`. A patched main executable passed all four synthetic browser
combinations. These fixtures use loopback HTTP and require no external account.

Validation with Rust 1.96.1:

- The media-list regression failed on unmodified main at `screen,print`.
- The focused stylesheet crate passed all 28 tests after the fix on macOS and Linux.
- Both the existing and new child-parser HTTP regressions passed on macOS,
  verifying stylesheet installation before the classic script and subsequent load.
- All required commands ran natively on macOS arm64 at PR revision
  `9910d2c82aeac7f061a7d1818629430fa2e9f13c`, with no source changes during validation.
- `cargo fmt --all` and
  `cargo clippy --workspace --all-targets --all-features -- -D warnings` passed.
- `cargo nextest run --no-fail-fast --test-threads 4` completed without filters or
  additional exclusions: 22,162 tests run, all 22,162 passed, no failures or leak
  reports, and 20 skipped by the repository's normal selection. Test execution took
  829 seconds; the command exited zero. The new stylesheet regressions passed.
- The PR contains a separate test-fixture portability commit. The two original
  failures were first reproduced on clean upstream main
  `28701350d91991670b5690b2f0bec8e6cdb9d51d`. The connection test now releases its
  reserved port before connecting and retries only confirmed port collisions,
  preserving the curl code 7 assertion. The Unicode-range test uses bundled narrow
  and wide fonts instead of platform monospace, preserving its layout and
  descriptor-mutation assertions. Both corrected tests passed individually and in
  the full run. The committed changes exactly match the tested source.

All required native contribution checks passed. This patch artifact contains only
the stylesheet fix; the fixture-portability changes are a separate commit in the PR.
The patch addresses the stylesheet reproduction; full application startup remains
unverified.
