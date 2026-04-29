# Scottish Rite Site — Deployment Notes

## Architecture (Updated 2026-04-29)

Two deployments share the same source files:

| Deployment | URL | Source path |
|-----------|-----|-------------|
| zo.space | https://jaknyfe.zo.space/scottish-rite | `/home/workspace/scottish-rite-site/index.html` (direct read) |
| External hosting | https://scottish-rite-jaknyfe.zocomputer.io/ | `/home/workspace/scottish-rite-site/` (served by python http.server on port 3102) |
| wildcatart.com | https://wildcatart.com/scottish-rite/ | rsync from `scottish-rite-site/` via cron |

**Source of truth**: `scottish-rite-site/index.html` (git tracked)

## Synchronization Rule (CRITICAL)

Whenever the Scottish Rite site is updated, ALL three must be updated in sequence:

1. Build: `cd /home/workspace/scottish-rite && bun run build`
2. Copy to serve dir: `cp dist/index.html dist/assets/* /home/workspace/scottish-rite-site/`
3. Push to GitHub: `cd /home/workspace/scottish-rite-site && git add . && git commit -m "..." && git push`
4. Restart external hosting: `update_user_service` with `service_id=svc_zLzh3iJP_c4`
5. Deploy to wildcatart.com: `rsync -avz -e "ssh -o StrictHostKeyChecking=no" --delete /home/workspace/scottish-rite-site/ wildcatart.com@ssh.us.stackcp.com:~/public_html/scottish-rite/`

## Failure History

- **2026-04-29**: The external hosting service (`scottish-rite/dist/`) fell out of sync because only the git repo (`scottish-rite-site/`) was updated after a content fix. The dist folder was rebuilt but never synchronized with the external hosting service, and the service wasn't restarted to pick up the new file.

## Scottish Rite Website (`scottish-rite-site/`)

**Source of truth**: `/home/workspace/scottish-rite-site/` — this directory is the canonical deploy source for all three Scottish Rite deployment targets (zo.space API route, external hosting on port 3102, wildcatart.com rsync).

**Key lesson**: When new content arrives via zip upload, it extracts into the Vite source directory (`scottish-rite/`). The serve directory `scottish-rite-site/` is always the correct destination for deployment. Never use `scottish-rite/` or `scottish-rite/dist/` as a deployment source.

**Deployment workflow** (every time):
1. `cd /home/workspace/scottish-rite && bun run build`
2. `cp dist/index.html dist/assets/* /home/workspace/scottish-rite-site/`
3. `cd /home/workspace/scottish-rite-site && git add . && git commit -m "Update" && git push`
4. `update_user_service` with `service_id=svc_zLzh3iJP_c4` to restart external hosting

**Service IDs**: External hosting `svc_zLzh3iJP_c4` (port 3102, python http.server serving `scottish-rite-site/`)

**URLs**:
- External: `https://scottish-rite-jaknyfe.zocomputer.io/`
- zo.space: `https://jaknyfe.zo.space/scottish-rite`