---
name: "Release"
about: "Checklist for creating a new OpenChoreo release"
title: "Release: v<MAJOR>.<MINOR>.<PATCH>"
labels: "type/release"
type: Task
---

### Prerequisites

- [ ] `CHANGELOG.md` updated with all changes for this version
- [ ] `VERSION` file contains `MAJOR.MINOR.PATCH` (with `-PRE_RELEASE_ID` suffix if prerelease)
- [ ] `build-and-test` passing on the target branch

All workflow dispatch triggers should be run from the **default branch (`main`)** of the respective repository. The workflows auto-resolve the correct target branch at runtime.

---

### Major/Minor Release

1. **Release backstage-plugins**
   - Merge a PR updating `VERSION` to `MAJOR.MINOR.PATCH` on `main`
   - Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH`
   - Run [**Prepare Next Version**](https://github.com/openchoreo/backstage-plugins/actions) — `MAJOR`, `MINOR+1`, `0`
   - Review and merge the version bump PR

2. **Release openchoreo**
   - Merge a PR updating `VERSION` to `MAJOR.MINOR.PATCH` on `main`
   - Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH`
   - Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) and publish it
   - Run [**Prepare Next Version**](https://github.com/openchoreo/openchoreo/actions) — `MAJOR`, `MINOR+1`, `0`
   - Review and merge the version bump PR

3. **Update docs** (on [openchoreo.github.io](https://github.com/openchoreo/openchoreo.github.io))
   - Create a new doc version snapshot: `npm run docusaurus docs:version vMAJOR.MINOR.x`
   - Update version constants in the new `versioned_docs/version-vMAJOR.MINOR.x/_constants.mdx`
   - Update `docusaurus.config.ts` (lastVersion, announcementBar, versions map)
   - Run `./scripts/clean-versions.sh`
   - Update `docs/changelog.md` and `versioned_docs/version-vMAJOR.MINOR.x/changelog.md`
   - Update `docs/releases/release-and-support-process.md` (supported versions, latest patch)
   - Submit a PR to `openchoreo.github.io` with all changes

---

### Patch Release

1. **Release backstage-plugins**
   - Merge a PR updating `VERSION` to `MAJOR.MINOR.PATCH` on `release-vMAJOR.MINOR`
   - Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: tag`, `MAJOR`, `MINOR`, `PATCH`
   - Run [**Prepare Next Version**](https://github.com/openchoreo/backstage-plugins/actions) — `MAJOR`, `MINOR`, `PATCH+1`
   - Review and merge the version bump PR

2. **Release openchoreo**
   - Merge a PR updating `VERSION` to `MAJOR.MINOR.PATCH` on `release-vMAJOR.MINOR`
   - Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: full`, `MAJOR`, `MINOR`, `PATCH`
   - Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) and publish it
   - Run [**Prepare Next Version**](https://github.com/openchoreo/openchoreo/actions) — `MAJOR`, `MINOR`, `PATCH+1`
   - Review and merge the version bump PR

3. **Update docs** (on [openchoreo.github.io](https://github.com/openchoreo/openchoreo.github.io))
   - Update version constants in `versioned_docs/version-vMAJOR.MINOR.x/_constants.mdx`
   - Update `docusaurus.config.ts` (announcementBar)
   - Update `docs/changelog.md` and `versioned_docs/version-vMAJOR.MINOR.x/changelog.md`
   - Update `docs/releases/release-and-support-process.md` (latest patch for this minor line)
   - Submit a PR to `openchoreo.github.io` with changes

---

### Prerelease (e.g., v1.1.0-rc.1)

1. **Release backstage-plugins**
   - Merge a PR updating `VERSION` to `MAJOR.MINOR.PATCH-PRE_RELEASE_ID` on the target branch
   - Run [**Release Orchestrator**](https://github.com/openchoreo/backstage-plugins/actions) — `action: tag`, `MAJOR`, `MINOR`, `PATCH`, `pre_release_id`

2. **Release openchoreo**
   - Merge a PR updating `VERSION` to `MAJOR.MINOR.PATCH-PRE_RELEASE_ID` on the target branch
   - Run [**Release Orchestrator**](https://github.com/openchoreo/openchoreo/actions) — `action: tag`, `MAJOR`, `MINOR`, `PATCH`, `pre_release_id`
   - Verify the [draft release](https://github.com/openchoreo/openchoreo/releases) and publish it (mark as Pre-release)

3. **Update docs** (optional — only if docs changes are needed for this prerelease)
   - Follow the major/minor or patch docs steps above as applicable
