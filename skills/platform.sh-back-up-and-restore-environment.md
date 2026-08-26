---
name: platform.sh-back-up-and-restore-environment
description: Take a Platform.sh / Upsun environment backup, list restorable backups, and restore one — the only reversal path this API offers for environment data, with its documented retention window.
api: Platform.sh REST API
base_url: https://api.upsun.com
generated: '2026-08-26'
method: generated
source: openapi/platform.sh-rest-api-openapi.json
operations:
  - backup-environment
  - list-projects-environments-backups
  - get-projects-environments-backups
  - restore-backup
  - delete-projects-environments-backups
  - get-projects-environments-activities
---

# Back up and restore an environment

Restore is the strongest reversal this API provides. Environment and project **deletion have no
undelete** — if there is no backup, the data is gone.

## Retention window (read this before relying on it)

- **Manual backups** are retained until you delete or replace them.
- **Automated backups** are retained for **2 days** under the default backup policy — two days'
  worth of backups at any point. A custom `data_retention` schedule on project settings changes
  that. Source: https://developer.upsun.com/docs/environments/backup#data-retention
- Backups are stored internally and **cannot be downloaded**. To get data out, export mount and
  service data instead.

## Steps

1. **Take a backup.** `backup-environment`
   (`POST /projects/{projectId}/environments/{environmentId}/backup`). Only **active**
   environments can be backed up; activate an inactive one first. Returns an activity — poll
   `get-projects-environments-activities` until `state` is `complete`.
2. **List backups.** `list-projects-environments-backups`
   (`GET /projects/{projectId}/environments/{environmentId}/backups`). Only entries marked
   restorable can be used.
3. **Inspect one.** `get-projects-environments-backups`
   (`GET .../backups/{backupId}`).
4. **Restore.** `restore-backup`
   (`POST /projects/{projectId}/environments/{environmentId}/backups/{backupId}/restore`).
   Returns an activity; poll it to completion.
5. **Clean up.** `delete-projects-environments-backups`
   (`DELETE .../backups/{backupId}`) — irreversible.

## Rules

- **Restore is itself destructive.** It replaces current environment data and redeploys the built
  app from when the backup was taken. Take a fresh backup before restoring an older one.
- Upsun does **not** modify your Git repository during a restore. Code in the repository may no
  longer match the restored data.
- Not idempotent — a repeated `backup-environment` creates another backup and can evict the oldest
  one under a count-limited retention policy.
- `403` means the caller lacks the Admin role for that environment type.
