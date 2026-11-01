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
  `2477227fbac32cae855465f20b092262a6580f7f`, with no source changes.
- `cargo fmt --all` and
  `cargo clippy --workspace --all-targets --all-features -- -D warnings` passed.
- `cargo nextest run --no-fail-fast --test-threads 4` completed without filters or
  additional exclusions: 22,162 tests run, 22,160 passed (one leaky), two failed,
  and 20 skipped by the repository's normal selection. Test execution took 857
  seconds; the command exited 100. The new stylesheet regressions passed.
- Both failures reproduced with one test worker:
  `cli_connection_failure_reports_curl_code_and_cause` expected curl connection
  failure code 7 but received timeout code 28, and
  `font_face_unicode_range_limits_layout_and_updates_after_loading` failed its
  custom-font-versus-fallback control assertion. Both test files are unchanged by
  the patch.
- The passing selector test marked leaky in the full run,
  `rejects_more_upstream_invalid_selector_shapes`, passed without a leak on its
  individual rerun. No assertions or source code were changed during validation.

The full contribution gates have run, but the suite is not green. The engine fix
remains a draft with these results documented.
The patch addresses the stylesheet reproduction; full application startup remains
unverified.
