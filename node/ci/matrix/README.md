# CI workflow: ci.matrix.yaml — Matrix guide

This README explains the job matrix used by the `ci.matrix.yaml` workflow and offers practical guidance for sizing, artifact naming, concurrency, and cost-control.

**Purpose**
- The matrix strategy allows the workflow to run the same job across multiple combinations (OS, Node versions, browsers, etc.). Use it to validate builds across environments while keeping runs efficient.

**Where to find the workflow**
- See the workflow file: [node/ci/matrix/ci.matrix.yaml](node/ci/matrix/ci.matrix.yaml)

**Matrix basics**
- A typical matrix block looks like:

```yaml
strategy:
  matrix:
    node-version: [18, 20]
    os: [ubuntu-latest, macos-latest]
  fail-fast: false
  max-parallel: 4
```

- `matrix` creates one job per combination of listed axes. `fail-fast: false` keeps other combinations running if one fails. `max-parallel` limits how many run at once.

**Naming and artifacts per matrix run**
- When producing artifacts per matrix entry, include matrix placeholders in names so they are unique and traceable, e.g.:

```yaml
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: build-${{ matrix.os }}-node-${{ matrix.node-version }}
    path: dist/
```

- When cleaning up artifacts, target specific artifact names (or use `run_id`) to avoid accidentally deleting unrelated artifacts from other runs.

**Sizing & cost-control recommendations**
- Only include matrix axes you actually need to validate. Each axis multiplies job count.
- Use `include` to add one-off combos and `exclude` to remove invalid combos.
- Set `max-parallel` to a safe number to avoid spiking runner usage and costs.
- Use `retention-days` on artifact uploads (low value like `1`) and consider post-run deletion for ephemeral artifacts.

**Concurrency & throttling**
- Use the `concurrency` key to prevent duplicate runs for the same branch/ref and to cancel in-progress runs:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

- Keep `max-parallel` and `concurrency` in sync with your team’s CI capacity.

**Selective runs**
- Use `if:` conditions or `paths:` filters on jobs to skip matrix runs when irrelevant files change.
- Use `matrix.include` to add a special job that runs only when a particular label/flag is present.

**Debugging matrix runs**
- Logs: each matrix combination is a separate job — inspect the failing job for full logs.
- Re-run: GitHub UI allows re-running a single failed job; use that for iterative fixes.

**Best practices checklist**
- **Minimize axes:** remove unnecessary OS/versions.
- **Unique artifact names:** include `matrix` context in artifact names.
- **Control parallelism:** use `max-parallel` to limit simultaneous runners.
- **Fail strategy:** use `fail-fast: false` to gather full feedback across all combos.
- **Retention & cleanup:** set `retention-days` and/or delete artifacts post-run.

If you want, I can:
- Add a small workflow snippet that demonstrates `include`/`exclude` and artifact naming.
- Modify `ci.matrix.yaml` to add `concurrency`, `max-parallel`, or artifact naming using matrix variables.
