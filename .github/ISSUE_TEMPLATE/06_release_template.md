---
name: "Release"
about: "Checklist for creating a new OpenChoreo release"
title: "Release: v<MAJOR>.<MINOR>.<PATCH>"
labels: "type/release"
type: Task
---

### Prerequisites

- [ ] `VERSION` file updated to the target release version on the target branch (i.e., main/release-vMAJOR.MINOR) — submit a PR first if it needs updating
- [ ] `CHANGELOG.md` updated with all changes for this version on the target branch
- [ ] `build-and-test` passing on the target branch

All workflow dispatch triggers should be run from the **default branch (`main`)** of the respective repository. The workflows auto-resolve the correct target branch at runtime.

---

### Major/Minor Release

1. **Release backstage-plugins**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH` (creates the release branch and tags the release)

2. **Release openchoreo**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH` (creates the release branch and tags the release)
   - [ ] Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) is created

3. **Update docs** (on [openchoreo.github.io](https://github.com/openchoreo/openchoreo.github.io))
   - [ ] Create a new doc version snapshot: `npm run docusaurus docs:version vMAJOR.MINOR.x`
   - [ ] Update version constants in the new `versioned_docs/version-vMAJOR.MINOR.x/_constants.mdx`
   - [ ] Update `docusaurus.config.ts` (lastVersion, announcementBar, versions map)
   - [ ] Update `docs/changelog.md` and `versioned_docs/version-vMAJOR.MINOR.x/changelog.md`
   - [ ] Update `docs/releases/release-and-support-process.md` (supported versions, latest patch)
   - [ ] Submit a PR to `openchoreo.github.io` with all changes (keep open)

4. **Publish release and merge docs PR**
   - [ ] Publish the [draft release](https://github.com/openchoreo/openchoreo/releases)
   - [ ] Merge the docs PR in `openchoreo.github.io`

---

### Patch Release

1. **Release backstage-plugins**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH` (pushes a tag to the existing release branch)

2. **Release openchoreo**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH` (pushes a tag to the existing release branch)
   - [ ] Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) is created

3. **Update docs** (on [openchoreo.github.io](https://github.com/openchoreo/openchoreo.github.io))
   - [ ] Update version constants in `versioned_docs/version-vMAJOR.MINOR.x/_constants.mdx`
   - [ ] Update `docusaurus.config.ts` (announcementBar)
   - [ ] Update `docs/changelog.md` and `versioned_docs/version-vMAJOR.MINOR.x/changelog.md`
   - [ ] Update `docs/releases/release-and-support-process.md` (latest patch for this minor line)
   - [ ] Submit a PR to `openchoreo.github.io` with changes (keep open)

4. **Publish release and merge docs PR**
   - [ ] Publish the [draft release](https://github.com/openchoreo/openchoreo/releases)
   - [ ] Merge the docs PR in `openchoreo.github.io`

---

### Prerelease (e.g., v1.1.0-rc.1)

A temporary `release-vMAJOR.MINOR.PATCH-PRE_RELEASE_ID` branch is created for the **openchoreo** repo (via `action: full`). The **backstage-plugins** repo tags directly on `main` (via `action: tag`). The temporary branch can be cleaned up after the final release is done.

1. **Release backstage-plugins**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: tag`, `MAJOR`, `MINOR`, `PATCH`, `pre_release_id` (tags directly on `main`)

2. **Release openchoreo**
   - [ ] Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH`, `pre_release_id` (creates a temporary prerelease branch)
   - [ ] Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) is created (will be marked as a prerelease)

3. **Update docs** (optional — only if docs changes are needed for this prerelease)
   - [ ] Follow the major/minor or patch docs steps above as applicable
   - [ ] Submit a PR to `openchoreo.github.io` with changes (keep open)

4. **Publish release and merge docs PR**
   - [ ] Publish the [draft release](https://github.com/openchoreo/openchoreo/releases) (mark as prerelease)
   - [ ] Merge the docs PR in `openchoreo.github.io`

---

### Post-Release

Run [**Prepare Next Version**](https://github.com/openchoreo/backstage-plugins/actions) / [**Prepare Next Version**](https://github.com/openchoreo/openchoreo/actions) to generate a PR bumping `VERSION` to the next development version, then review and merge it.

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
