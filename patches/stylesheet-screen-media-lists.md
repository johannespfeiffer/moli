# Screen media-list patch

This branch contains a patch artifact. Apply it to the engine sources before building:

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
- Formatting and full workspace/all-target/all-feature Clippy passed on macOS.
- The full native Nextest run hit `cli_connection_failure_reports_curl_code_and_cause`
  (expected curl connection failure code 7, received timeout code 28) and was stopped.
- Linux workspace Clippy could not complete in the 8 GB validation VM. The renderer
  test compiler was OOM-killed with eight build workers and again with one; the
  container recorded three OOM kills.

The full contribution gates have not passed, so no engine pull request is submitted.
The patch addresses the stylesheet reproduction; full application startup remains
unverified.
