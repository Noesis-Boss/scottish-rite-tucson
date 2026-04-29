# Scottish Rite Site — Deployment Notes

## Architecture (Updated 2026-04-29)

Two deployments share the same source files:

| Deployment | URL | Source path |
|-----------|-----|-------------|
| zo.space | https://jaknyfe.zo.space/scottish-rite | `/home/workspace/scottish-rite-site/index.html` (direct read) |
| External hosting | https://scottish-rite-jaknyfe.zocomputer.io/ | `/home/workspace/scottish-rite/dist/` (served by python http.server on port 3102) |
| wildcatart.com | https://wildcatart.com/scottish-rite/ | rsync from `scottish-rite-site/` via cron |

**Source of truth**: `scottish-rite-site/index.html` (git tracked)

## Synchronization Rule (CRITICAL)

Whenever the Scottish Rite site is updated, ALL three must be updated in sequence:

1. `cp /home/workspace/scottish-rite-site/index.html /home/workspace/scottish-rite/dist/index.html`
2. `cp -r /home/workspace/scottish-rite-site/uploads/* /home/workspace/scottish-rite/dist/uploads/`
3. Restart the external service: `update_user_service` with `service_id=svc_zLzh3iJP_c4`
4. Push to GitHub: `cd /home/workspace/scottish-rite-site && git add . && git commit -m "..." && git push`
5. wildcatart.com syncs automatically via cron from the git-pushed `scottish-rite-site/` directory.

## Failure History

- **2026-04-29**: The external hosting service (`scottish-rite/dist/`) fell out of sync because only the git repo (`scottish-rite-site/`) was updated after a content fix. The dist folder was rebuilt but never synchronized with the external hosting service, and the service wasn't restarted to pick up the new file.

## Relevant Service IDs

- External Scottish Rite hosting: `svc_zLzh3iJP_c4`