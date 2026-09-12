# Atriarch fork of SimulationCraft

This fork feeds the Atriarch Simc platform. Three branches matter:

| Branch | Role |
| --- | --- |
| *(default, e.g. `midnight`)* | Mirror of upstream's default branch, synced daily by the workflow, **plus** `.github/workflows/atriarch-sync-build.yml` (crons only fire from the default branch). |
| `atriarch-overlay` | The Atriarch-specific files layered onto upstream: this README and the sync/build workflow. Edit the overlay → push → the build branch regenerates and the base image rebuilds. |
| `atriarch-simc` | **Generated — never commit here.** Rebuilt from scratch by the workflow: upstream tree, upstream CI workflows removed, overlay applied. Force-pushed on every change. |

## How the pipeline works

`atriarch-sync-build.yml` (daily cron / manual dispatch / overlay push):

1. Resolves upstream's *current* default branch at runtime — expansion
   branch renames (`thewarwithin` → `midnight` → ...) don't break anything.
2. Syncs the fork's copy of that branch via the merge-upstream API
   (creates the fork branch if upstream renamed).
3. Regenerates `atriarch-simc` = upstream tree + overlay. No merges, so
   no conflicts, ever.
4. If the tree changed (or `force_build` on manual dispatch), builds the
   base image with upstream's own `Dockerfile` and pushes
   `docker.atriarch.systems/atriarch-simc-simulation:<vYYYYMMDD.run>` and
   `:latest`.
5. Fires `repository_dispatch` (`simc-base-updated`) so
   `Atriarch.Simc.Engine.Node` rebuilds on the new base.

## After an expansion branch rename

The pipeline keeps working by itself. Optionally, for tidiness:
1. Make the new mirror branch the fork's default (Settings → Branches) so
   the cron runs from it.
2. The workflow file is carried on the old default — copy
   `.github/workflows/atriarch-sync-build.yml` onto the new default branch.

## Required secrets (org-level)

`GITOPS_TOKEN`, `DOCKER_USERNAME`, `DOCKER_PASSWORD`,
`DISCORD_BUILD_NOTIFY_WEBHOOK`.
