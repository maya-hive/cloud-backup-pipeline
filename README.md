# Cloud Backup Pipeline

This pipeline uses AWS compatible S3 storage to store application database and file backups. The backup pipeline retains only a specified count of backups to prevent storage overflow.

### Required GitHub Action Secrets

```env
AWS_ACCESS_KEY_ID
AWS_REGION
AWS_S3_BUCKET
AWS_S3_ENDPOINT
AWS_S3_OBJECT
AWS_SECRET_ACCESS_KEY

MYSQL_HOST
MYSQL_PORT
MYSQL_PASS
MYSQL_USER
MYSQL_DATABASE

SERVER_HOSTNAME
SERVER_PORT
SERVER_USERNAME
SERVER_SECRET_KEY

DISCORD_WEBHOOK

FILE_BACKUP_APPLICATION_PATH
FILE_BACKUP_RETAIN

DATABASE_BACKUP_RETAIN
```

### Using This Workflow

For database backups insert the below code in `.github/workflows/backup-database.yml`:

```yaml
---
name: Backup Application Database

on:
  schedule:
    - cron: "30 22,16 * * *" # For SL timezone add 5hrs 30mins
  workflow_dispatch:

jobs:
  call-workflow-backup-database:
    name: Call Backup Database Workflow
    uses: maya-hive/cloud-backup-pipeline/.github/workflows/database.yml@v1.0.0 # Specify the required release version
    secrets: inherit
```

For database backups insert the below code in `.github/workflows/backup-files.yml`:

```yaml
---
name: Backup Application Files

on:
  schedule:
    - cron: "30 22 * * *" # For SL timezone add 5hrs 30mins
  workflow_dispatch:

jobs:
  call-workflow-backup-files:
    name: Call Backup Files Workflow
    uses: maya-hive/cloud-backup-pipeline/.github/workflows/files.yml@v1.0.0 # Use the required release version
    secrets: inherit
```

### Development/Testing

Use nektos/act to run GitHub workflows locally:

```bash
act --secret-file .env workflow_dispatch -v
```
