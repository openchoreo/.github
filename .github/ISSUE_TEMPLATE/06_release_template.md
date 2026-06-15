---
name: "Release"
about: "Checklist for creating a new OpenChoreo release"
title: "Release: v<MAJOR>.<MINOR>.<PATCH>"
labels: "type/release"
type: Task
---

### Prerequisites

- [ ] `VERSION` file is set to the target release version in both **backstage-plugins** and **openchoreo** repos on the target branch (i.e., main/release-vMAJOR.MINOR) — if not, run the [**Prepare Next Version**](#post-release) workflow to create the bump PR
- [ ] `CHANGELOG.md` updated with all changes for this version on the target branch
- [ ] `build-and-test` passing on the target branch

All workflow dispatch triggers should be run from the **default branch (`main`)** of the respective repository. The workflows auto-resolve the correct target branch at runtime.

> **E2E gate (openchoreo):** every `Release Orchestrator` run is gated on the full e2e suite (~30 min) before the tag is created. To gate up front, cut the branch first with `action: branch`, then run `action: tag` — the gate is reused when the run targets the same commit, so it is not re-run; any new commit (a fix or backport) is gated afresh. `skip_e2e: true` bypasses the gate for declared emergencies only. See the [release guide](https://github.com/openchoreo/openchoreo/blob/main/docs/contributors/release.md#e2e-release-gate).

> **Backstage image dependency:** the gate's UI leg installs Backstage from the `openchoreo-ui` image built by **backstage-plugins**, tagged with that repo's commit SHA. The gate resolves the backstage-plugins release branch (its name mirrors the openchoreo release branch) and pulls its branch-tip image. So **branch backstage-plugins first** (and let its `build-and-test` publish the image) before releasing openchoreo, but **tag backstage-plugins last** — only after the openchoreo tag succeeds — so the backstage-plugins release tag is only cut for a release that passed the gate. When no mirror branch exists (prereleases tag on `main`), the gate falls back to `latest-dev`.

---

### Major/Minor Release

1. **Branch backstage-plugins**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: branch`, `MAJOR`, `MINOR`, `PATCH` (creates the release branch only — no tag yet)
   - [ ] Wait for `build-and-test` on the new branch to finish publishing the `openchoreo-ui` image — the openchoreo e2e gate's UI leg pulls this branch-tip image

2. **Release openchoreo**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH` (creates the release branch, runs the e2e gate against the backstage-plugins branch image, and tags the release)
   - [ ] Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) is created

3. **Tag backstage-plugins** — only after the openchoreo tag succeeds
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: tag`, `MAJOR`, `MINOR`, `PATCH` (tags the release branch cut in step 1)

4. **Update docs** (on [openchoreo.github.io](https://github.com/openchoreo/openchoreo.github.io))
   - [ ] Create a new doc version snapshot: `npm run docusaurus docs:version vMAJOR.MINOR.x`
   - [ ] Update version constants in the new `versioned_docs/version-vMAJOR.MINOR.x/_constants.mdx`
   - [ ] Update `docusaurus.config.ts` (lastVersion, announcementBar, versions map)
   - [ ] Update `docs/changelog.md` and `versioned_docs/version-vMAJOR.MINOR.x/changelog.md`
   - [ ] Update `docs/releases/release-and-support-process.md` (supported versions, latest patch)
   - [ ] If prereleases were done for this version, remove any prerelease doc version snapshots (e.g., `versioned_docs/version-vMAJOR.MINOR.x-rc.*`)
   - [ ] Submit a PR to `openchoreo.github.io` with all changes (keep open)

5. **Publish release and merge docs PR**
   - [ ] Publish the [draft release](https://github.com/openchoreo/openchoreo/releases)
   - [ ] Merge the docs PR in `openchoreo.github.io`

---

### Patch Release

The release branches already exist in both repos, so backstage-plugins is not re-branched — but it is still tagged **after** openchoreo, and its branch-tip `openchoreo-ui` image is what the openchoreo e2e gate pulls.

1. **Prepare backstage-plugins release branch**
   - [ ] Confirm the patch commits are on the backstage-plugins `release-vMAJOR.MINOR` branch and its `build-and-test` is green (its branch-tip `openchoreo-ui` image is what the openchoreo e2e gate pulls)

2. **Release openchoreo**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH` (runs the e2e gate against the backstage-plugins branch image, then pushes a tag to the existing release branch)
   - [ ] Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) is created

3. **Tag backstage-plugins** — only after the openchoreo tag succeeds
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: tag`, `MAJOR`, `MINOR`, `PATCH` (pushes a tag to the existing release branch)

4. **Update docs** (on [openchoreo.github.io](https://github.com/openchoreo/openchoreo.github.io))
   - [ ] Update version constants in `versioned_docs/version-vMAJOR.MINOR.x/_constants.mdx`
   - [ ] Update `docusaurus.config.ts` (announcementBar)
   - [ ] Update `docs/changelog.md` and `versioned_docs/version-vMAJOR.MINOR.x/changelog.md`
   - [ ] Update `docs/releases/release-and-support-process.md` (latest patch for this minor line)
   - [ ] Submit a PR to `openchoreo.github.io` with changes (keep open)

5. **Publish release and merge docs PR**
   - [ ] Publish the [draft release](https://github.com/openchoreo/openchoreo/releases)
   - [ ] Merge the docs PR in `openchoreo.github.io`

---

### Prerelease (e.g., v1.1.0-rc.1)

A temporary `release-vMAJOR.MINOR.PATCH-PRE_RELEASE_ID` branch is created for the **openchoreo** repo (via `action: full`). The **backstage-plugins** repo tags directly on `main` (via `action: tag`). Because no mirror release branch is cut, the openchoreo e2e gate's UI leg falls back to the `latest-dev` `openchoreo-ui` image. The temporary branch can be cleaned up after the final release is done.

1. **Release openchoreo**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH`, `pre_release_id` (creates a temporary prerelease branch; the e2e gate uses the `latest-dev` backstage image)
   - [ ] Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) is created (will be marked as a prerelease)

2. **Tag backstage-plugins** — only after the openchoreo tag succeeds
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: tag`, `MAJOR`, `MINOR`, `PATCH`, `pre_release_id` (tags directly on `main`)

3. **Update docs** (optional — only if docs changes are needed for this prerelease)
   - [ ] Follow the major/minor or patch docs steps above as applicable
   - [ ] Submit a PR to `openchoreo.github.io` with changes (keep open)

4. **Publish release and merge docs PR**
   - [ ] Publish the [draft release](https://github.com/openchoreo/openchoreo/releases) (mark as prerelease)
   - [ ] Merge the docs PR in `openchoreo.github.io`

---

### Post-Release

Run [**Prepare Next Version**](https://github.com/openchoreo/backstage-plugins/actions) / [**Prepare Next Version**](https://github.com/openchoreo/openchoreo/actions) with the params below — the pipeline auto-creates a PR bumping `VERSION` to the next development version. Review and merge the generated PRs.

| Release type | `MAJOR` | `MINOR` | `PATCH` | `PRE_RELEASE_ID` |
|---|---|---|---|---|
| Major/minor | `MAJOR` | `MINOR+1` | `0` | |
| Major/minor (prerelease) | `MAJOR` | `MINOR` | `PATCH` | `pre_release_id` |
| Patch | `MAJOR` | `MINOR` | `PATCH+1` | |

**backstage-plugins**
- [ ] Run [**Prepare Next Version**](https://github.com/openchoreo/backstage-plugins/actions) with the params above
- [ ] Merge the generated PR updating `VERSION`

**openchoreo**
- [ ] Run [**Prepare Next Version**](https://github.com/openchoreo/openchoreo/actions) with the params above
- [ ] Merge the generated PR updating `VERSION`
