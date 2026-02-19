# CI workflow: ci.vite.yaml

This README documents the post-deploy artifact cleanup used by the `ci.vite.yaml` workflow.

**Purpose**
- The workflow builds the frontend, uploads a `build-dist` artifact, deploys its contents to a VPS, then removes the artifact created by the current workflow run.

**Where to find the workflow**
- See the workflow file: [node/ci/ci.vite.yaml](node/ci/ci.vite.yaml)

**How artifact cleanup works (current implementation)**
- The job uploads the artifact with `actions/upload-artifact@v4` as `build-dist` and sets `retention-days: 1` to minimise storage lifetime.
- After the deploy step, the workflow uses `actions/github-script@v7` to call the REST API and delete the artifact created by the current run only. The script:

```js
const artifacts = await github.rest.actions.listWorkflowRunArtifacts({
  owner: context.repo.owner,
  repo: context.repo.repo,
  run_id: context.runId,
});

const artifact = artifacts.data.artifacts.find(a => a.name === "build-dist");

if (artifact) {
  await github.rest.actions.deleteArtifact({
    owner: context.repo.owner,
    repo: context.repo.repo,
    artifact_id: artifact.id,
  });
  console.log("Artifact deleted");
} else {
  console.log("Artifact not found");
}
```

**Why this cleanup is useful**
- Reduces storage use and billing for stored artifacts.
- Limits exposure of build output left in GitHub (sensible for temporary deploy artifacts).
- Gives deterministic cleanup for the current run regardless of retention policy timing.

**Permissions & troubleshooting**
- If the deletion step fails with permission errors, ensure the workflow has the `actions` permission set to `write`. Example in your workflow root:

```yaml
permissions:
  contents: read
  actions: write
```

- The default `GITHUB_TOKEN` is used by `actions/github-script`. If your organization enforces custom permissions, you may need to provide a PAT with appropriate scopes.

- Check the job logs for the `Delete artifact after deploy` step to see the `Artifact not found` or errors returned by the REST call. The script logs both deletion success and not-found cases.

**Alternative approaches / recommendations**
- Keep `retention-days` low on upload (already set to `1`) to reduce reliance on manual deletion.
- For deleting artifacts across multiple runs or by name for the whole repo, use the REST API to list repo artifacts and filter by name/date.
- Optionally use the `gh` CLI or a small action that explicitly deletes artifacts if you prefer not to use `actions/github-script`.

**Quick checklist**
- **Artifact name:** ensure `build-dist` matches in both upload and delete logic.
- **Permissions:** ensure `actions: write` if deletion fails.
- **Retention:** keep `retention-days` low (1 day) as a fallback.

If you want, I can also:
- Add a dedicated artifact-delete action that deletes all matching artifacts older than N days.
- Modify the workflow to require explicit `permissions:` and add a short test step that confirms artifact deletion.
