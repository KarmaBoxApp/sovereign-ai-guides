[dependency-mirroring.md](https://github.com/user-attachments/files/35029699/dependency-mirroring.md)
# Dependency mirroring: keep a copy of what your build pulls

On November 17, 2026, `github.com/microsoft/onnxruntime`, a repository with roughly 22k stars, began returning 404 with no notice; the PyPI package kept working while source, submodules, release assets, and issue history went unreachable. This guide is the setup that makes that kind of event boring: a mirror of everything your build pulls, on storage you control.

Mirrors address one risk only: an upstream going away. They do not review code (a backdoored commit mirrors perfectly), and they do not maintain it (a frozen copy gets no fixes). Handle those separately.

## 1. Inventory what you pull

List every external source your build touches. The grep below seeds the list; finish it by hand, because weights and release assets often hide in scripts and docs.

```bash
grep -rhoE "github.com/[A-Za-z0-9_.-]+/[A-Za-z0-9_.-]+" \
  --include="*.json" --include="*.lock" --include="*.yml" --include="*.yaml" \
  --include="*.toml" --include="*.txt" --include="Dockerfile*" . 2>/dev/null | sort -u
```

| Dependency | Host | What you pull | Copy held? | Mirror location |
|---|---|---|---|---|
| _example: microsoft/onnxruntime_ | GitHub | source + release wheels | no | _to set up_ |
| _example: actions/checkout_ | GitHub | workflow action, fetched per run | no | _to set up_ |
| _example: model weights, 7B Q4_ | Hugging Face | 4.1 GB file by URL | yes | NAS `/mirrors/weights/` |

Keep one row per upstream. "Copy held?" means a copy on storage you control, restorable without the upstream existing. A pinned version or commit SHA is an address, so it does not count.

## 2. Mirror git repositories

```bash
# one-time, per repository
git clone --mirror https://github.com/ORG/REPO.git /mirrors/git/REPO.git

# refresh on a schedule (cron shown; a scheduled CI job works the same)
cd /mirrors/git/REPO.git && git remote update
```

A `--mirror` clone carries all branches, tags, and refs. Restore path when the upstream is gone:

```bash
git clone /mirrors/git/REPO.git REPO
```

## 3. Mirror release assets

`--mirror` does not include release binaries. For repositories whose releases you download in CI or Dockerfiles:

```bash
gh release download --repo ORG/REPO --pattern '*' --dir /mirrors/assets/REPO/
```

Rerun per release you depend on. If a Dockerfile pulls a release URL, point it at your mirror and fall back to upstream, never the other way around.

## 4. Mirror registry packages

| Approach | Covers | Effort | Gap it leaves |
|---|---|---|---|
| Wheelhouse (`pip download -r requirements.txt -d ./wheelhouse`) | Python deps at pinned versions | One command, rerun on dep changes | Only the versions you downloaded |
| Pull-through proxy (Verdaccio, devpi, Artifactory, Nexus) | Everything your builds actually request | A small always-on service | Only packages requested at least once |
| Vendored source tree (deps checked into your repo) | Exact code, reviewable in diffs | Heavy; noisy diffs | Binary/platform artifacts still external |
| Object-storage dump (sync mirror dirs to S3-compatible bucket you own) | Everything above, off-site | One sync job | It is a backup of mirrors, not a mirror itself |

A lockfile plus a pull-through proxy covers the common case: builds keep working from cache when an upstream entry disappears.

## 5. Mirror the easy-to-forget layers

- **CI actions.** Every `uses: org/action@ref` line in a workflow is a repo fetched at run time. Mirror the ones you use, or vendor them into `.github/actions/`.
- **Model weights.** Fetched by URL at setup time in most AI projects, and the largest single artifacts in the stack. Store them like release assets, with the config or revision hash alongside.
- **Base images.** `docker pull` then `docker save`, or push to a registry you run.

## Storage estimate

A full dependency mirror for a typical service measures in tens of gigabytes. Price that against one day of a blocked pipeline and decide deliberately. For most teams the mirror wins by a wide margin, which is why the surprising part of November 17 was not that a repo vanished, but how few teams could say whether it affected them.
