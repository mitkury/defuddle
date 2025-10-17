## Using a temporary fork of `defuddle` respectfully

This document explains why a temporary fork exists, how to use it from Git without publishing a new package to the public npm registry, and how to cleanly migrate back once upstream is updated.

### Context

- Feature request: "Allow to use markdown outside of the Node bundle" — see issue [#105](https://github.com/kepano/defuddle/issues/105).
- Pull request proposing the change: [#82](https://github.com/kepano/defuddle/pull/82).

This fork is published on Git (not npm) for my own immediate use while upstream review proceeds. The goal is to avoid polluting npm and to keep the package namespace respectful.

### What this fork adds

- A separate bundle export `defuddle/markdown` that enables Markdown output outside of Node.js by default, suitable for browsers/Deno and other runtimes.
- The core bundle remains dependency‑light; Markdown processing can be injected or used via the dedicated markdown bundle.

### Install from Git (recommended)

Add a Git dependency in your app's `package.json`. Pin to a commit for reproducible builds:

```json
{
  "dependencies": {
    "defuddle": "github:mitkury/defuddle#<commit-sha>"
  },
  "overrides": {
    "defuddle": "github:mitkury/defuddle#<commit-sha>"
  }
}
```

Notes:

- Replace `<commit-sha>` with the exact commit you want to lock to (avoid branches for production).
- If `defuddle` is a transitive dependency, keep the direct dependency as‑is and use `overrides` (npm)/`resolutions` (Yarn)/`overrides` (pnpm) to force your forked version.
- For private repos in CI, prefer the SSH form: `git+ssh://git@github.com/mitkury/defuddle.git#<commit-sha>` and ensure CI has an SSH key with access.

### Build artifacts and install scripts

- This repo includes built artifacts under `dist/`, so Git installs work without running build scripts.
- If you prefer building on install, ensure a `prepare` script exists in `package.json` (npm runs `prepare` for Git deps):

```json
{
  "scripts": {
    "prepare": "npm run build"
  }
}
```

If your environment uses `--ignore-scripts`, rely on the committed `dist/` instead.

### Usage examples

Node bundle (no markdown by default):

```ts
import { Defuddle } from 'defuddle/node';

// ... construct Document and options ...
const result = new Defuddle(document, { /* options */ });
```

Markdown‑enabled bundle (works outside Node):

```ts
import DefuddleWithMarkdown from 'defuddle/markdown';

// ... construct Document ...
const result = new DefuddleWithMarkdown(document, { /* options */ });
// result includes contentMarkdown when separateMarkdown is enabled (default here)
```

Or inject your own markdown processor via options in environments where you need a different implementation.

### Alternative: tarball install (no registry)

If you need an artifact, you can pack and install directly:

```bash
npm pack  # from the fork repo; creates defuddle-<version>.tgz
npm i ./defuddle-<version>.tgz
```

### Respectful publishing guidelines

- Do not publish to the public npm registry under the original name.
- If publishing is absolutely necessary, use your own scope, a prerelease version, and a non‑latest dist‑tag; clearly link to the upstream PR and deprecate once upstream ships.

### Migration back to upstream

Once the upstream change is released, remove the Git dependency/override and depend on the official `defuddle` version that includes the feature.


