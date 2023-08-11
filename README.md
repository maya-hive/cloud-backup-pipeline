# Cloud Backup Pipeline

This pipeline uses AWS-compatible S3 storage to store application databases and file backups. The backup pipeline retains only a specified count of backups to prevent storage overflow.

### Configuration Guide

| Requirement                     | Calculation Logic                                              | Usage                                         |
| ------------------------------- | -------------------------------------------------------------- | --------------------------------------------- |
| **Run**: Every 1 Day at 1.00 AM | Format the cron job to run twice a day                         | `0 1 * * *`                                   |
| **Keep**: 2 Days                | Multiply the daily `Run` count with the number of days to keep | `DATABASE_BACKUP_RETAIN/FILE_BACKUP_RETAIN=4` |

Summary: The above will keep running every day at 1.00 AM and will retain up to 2 days of backups for us to roll back.

### Required GitHub Repository Action Secrets

We use Encrypted secrets to store sensitive credentials.

```env
AWS_REGION
AWS_S3_BUCKET
AWS_S3_ENDPOINT
AWS_S3_OBJECT

MYSQL_HOST
MYSQL_PORT
MYSQL_USER
MYSQL_DATABASE

SERVER_HOSTNAME
SERVER_PORT
SERVER_USERNAME

FILE_BACKUP_APPLICATION_PATH
FILE_BACKUP_RETAIN
DATABASE_BACKUP_RETAIN
```

### Required GitHub Repository Action Variables

We use variables for non-sensitive credentials that can be read or updated later.

```env
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
DISCORD_WEBHOOK
SERVER_SECRET_KEY
MYSQL_PASS
```

### Using This Workflow

For database backups insert the below code in `.github/workflows/backup-database.yml`:

```yaml
---
name: Backup Application Database

on:
  schedule:
    - cron: "30 22,16 * * *" # For SL timezone add 5 hrs & 30 mins
  workflow_dispatch:

jobs:
  vars:
    name: Setup Environment Vars
    runs-on: ubuntu-latest
    outputs:
      DATABASE_BACKUP_RETAIN: ${{ steps.set.outputs.DATABASE_BACKUP_RETAIN }}
      AWS_REGION: ${{ steps.set.outputs.AWS_REGION }}
      AWS_S3_ENDPOINT: ${{ steps.set.outputs.AWS_S3_ENDPOINT }}
      AWS_S3_BUCKET: ${{ steps.set.outputs.AWS_S3_BUCKET }}
      AWS_S3_OBJECT: ${{ steps.set.outputs.AWS_S3_OBJECT }}
      SERVER_HOSTNAME: ${{ steps.set.outputs.SERVER_HOSTNAME }}
      SERVER_PORT: ${{ steps.set.outputs.SERVER_PORT }}
      SERVER_USERNAME: ${{ steps.set.outputs.SERVER_USERNAME }}
      MYSQL_HOST: ${{ steps.set.outputs.MYSQL_HOST }}
      MYSQL_PORT: ${{ steps.set.outputs.MYSQL_PORT }}
      MYSQL_USER: ${{ steps.set.outputs.MYSQL_USER }}
      MYSQL_DATABASE: ${{ steps.set.outputs.MYSQL_DATABASE }}
      NOTIFY_SERVER_HOSTNAME: ${{ steps.set.outputs.NOTIFY_SERVER_HOSTNAME }}
      NOTIFY_SERVER_PORT: ${{ steps.set.outputs.NOTIFY_SERVER_PORT }}
      NOTIFY_SERVER_USERNAME: ${{ steps.set.outputs.NOTIFY_SERVER_USERNAME }}
      NOTIFY_MYSQL_HOST: ${{ steps.set.outputs.NOTIFY_MYSQL_HOST }}
      NOTIFY_MYSQL_PORT: ${{ steps.set.outputs.NOTIFY_MYSQL_PORT }}
      NOTIFY_MYSQL_USER: ${{ steps.set.outputs.NOTIFY_MYSQL_USER }}
      NOTIFY_MYSQL_DATABASE: ${{ steps.set.outputs.NOTIFY_MYSQL_DATABASE }}
    steps:
      - name: Assign Vars
        id: set
        run: |
          echo "DATABASE_BACKUP_RETAIN=${{ vars.DATABASE_BACKUP_RETAIN }}" >> $GITHUB_OUTPUT
          echo "AWS_REGION=${{ vars.AWS_REGION }}" >> $GITHUB_OUTPUT
          echo "AWS_S3_ENDPOINT=${{ vars.AWS_S3_ENDPOINT }}" >> $GITHUB_OUTPUT
          echo "AWS_S3_BUCKET=${{ vars.AWS_S3_BUCKET }}" >> $GITHUB_OUTPUT
          echo "AWS_S3_OBJECT=${{ vars.AWS_S3_OBJECT }}" >> $GITHUB_OUTPUT
          echo "SERVER_HOSTNAME=${{ vars.SERVER_HOSTNAME }}" >> $GITHUB_OUTPUT
          echo "SERVER_PORT=${{ vars.SERVER_PORT }}" >> $GITHUB_OUTPUT
          echo "SERVER_USERNAME=${{ vars.SERVER_USERNAME }}" >> $GITHUB_OUTPUT
          echo "MYSQL_HOST=${{ vars.MYSQL_HOST }}" >> $GITHUB_OUTPUT
          echo "MYSQL_PORT=${{ vars.MYSQL_PORT }}" >> $GITHUB_OUTPUT
          echo "MYSQL_USER=${{ vars.MYSQL_USER }}" >> $GITHUB_OUTPUT
          echo "MYSQL_DATABASE=${{ vars.MYSQL_DATABASE }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_SERVER_HOSTNAME=${{ vars.NOTIFY_SERVER_HOSTNAME }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_SERVER_PORT=${{ vars.NOTIFY_SERVER_PORT }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_SERVER_USERNAME=${{ vars.NOTIFY_SERVER_USERNAME }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_HOST=${{ vars.NOTIFY_MYSQL_HOST }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_PORT=${{ vars.NOTIFY_MYSQL_PORT }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_USER=${{ vars.NOTIFY_MYSQL_USER }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_DATABASE=${{ vars.NOTIFY_MYSQL_DATABASE }}" >> $GITHUB_OUTPUT

  call-workflow-backup-database:
    name: Call Backup Database Workflow
    uses: maya-hive/cloud-backup-pipeline/.github/workflows/database.yml@v2.0.0 # Use the required release version
    secrets: inherit
    needs: vars
    with:
      DATABASE_BACKUP_RETAIN: ${{ needs.vars.outputs.DATABASE_BACKUP_RETAIN }}
      AWS_REGION: ${{ needs.vars.outputs.AWS_REGION }}
      AWS_S3_BUCKET: ${{ needs.vars.outputs.AWS_S3_BUCKET }}
      AWS_S3_OBJECT: ${{ needs.vars.outputs.AWS_S3_OBJECT }}
      AWS_S3_ENDPOINT: ${{ needs.vars.outputs.AWS_S3_ENDPOINT }}
      SERVER_PORT: ${{ needs.vars.outputs.SERVER_PORT }}
      SERVER_HOSTNAME: ${{ needs.vars.outputs.SERVER_HOSTNAME }}
      SERVER_USERNAME: ${{ needs.vars.outputs.SERVER_USERNAME }}
      MYSQL_HOST: ${{ needs.vars.outputs.MYSQL_HOST }}
      MYSQL_PORT: ${{ needs.vars.outputs.MYSQL_PORT }}
      MYSQL_USER: ${{ needs.vars.outputs.MYSQL_USER }}
      MYSQL_DATABASE: ${{ needs.vars.outputs.MYSQL_DATABASE }}
      NOTIFY_SERVER_PORT: ${{ needs.vars.outputs.NOTIFY_SERVER_PORT }}
      NOTIFY_SERVER_HOSTNAME: ${{ needs.vars.outputs.NOTIFY_SERVER_HOSTNAME }}
      NOTIFY_SERVER_USERNAME: ${{ needs.vars.outputs.NOTIFY_SERVER_USERNAME }}
      NOTIFY_MYSQL_HOST: ${{ needs.vars.outputs.NOTIFY_MYSQL_HOST }}
      NOTIFY_MYSQL_PORT: ${{ needs.vars.outputs.NOTIFY_MYSQL_PORT }}
      NOTIFY_MYSQL_USER: ${{ needs.vars.outputs.NOTIFY_MYSQL_USER }}
      NOTIFY_MYSQL_DATABASE: ${{ needs.vars.outputs.NOTIFY_MYSQL_DATABASE }}
```

For application files backups insert the below code in `.github/workflows/backup-files.yml`:

```yaml
---
name: Backup Application Files

on:
  schedule:
    - cron: "30 22 * * *" # For SL timezone add 5 hrs & 30 mins
  workflow_dispatch:

jobs:
  vars:
    name: Setup Environment Vars
    runs-on: ubuntu-latest
    outputs:
      FILE_BACKUP_RETAIN: ${{ steps.set.outputs.FILE_BACKUP_RETAIN }}
      FILE_BACKUP_APPLICATION_PATH: ${{ steps.set.outputs.FILE_BACKUP_APPLICATION_PATH }}
      AWS_REGION: ${{ steps.set.outputs.AWS_REGION }}
      AWS_S3_ENDPOINT: ${{ steps.set.outputs.AWS_S3_ENDPOINT }}
      AWS_S3_BUCKET: ${{ steps.set.outputs.AWS_S3_BUCKET }}
      AWS_S3_OBJECT: ${{ steps.set.outputs.AWS_S3_OBJECT }}
      SERVER_HOSTNAME: ${{ steps.set.outputs.SERVER_HOSTNAME }}
      SERVER_PORT: ${{ steps.set.outputs.SERVER_PORT }}
      SERVER_USERNAME: ${{ steps.set.outputs.SERVER_USERNAME }}
      NOTIFY_SERVER_HOSTNAME: ${{ steps.set.outputs.NOTIFY_SERVER_HOSTNAME }}
      NOTIFY_SERVER_PORT: ${{ steps.set.outputs.NOTIFY_SERVER_PORT }}
      NOTIFY_SERVER_USERNAME: ${{ steps.set.outputs.NOTIFY_SERVER_USERNAME }}
      NOTIFY_MYSQL_HOST: ${{ steps.set.outputs.NOTIFY_MYSQL_HOST }}
      NOTIFY_MYSQL_PORT: ${{ steps.set.outputs.NOTIFY_MYSQL_PORT }}
      NOTIFY_MYSQL_USER: ${{ steps.set.outputs.NOTIFY_MYSQL_USER }}
      NOTIFY_MYSQL_DATABASE: ${{ steps.set.outputs.NOTIFY_MYSQL_DATABASE }}
    steps:
      - name: Assign Vars
        id: set
        run: |
          echo "FILE_BACKUP_RETAIN=${{ vars.FILE_BACKUP_RETAIN }}" >> $GITHUB_OUTPUT
          echo "FILE_BACKUP_APPLICATION_PATH=${{ vars.FILE_BACKUP_APPLICATION_PATH }}" >> $GITHUB_OUTPUT
          echo "AWS_REGION=${{ vars.AWS_REGION }}" >> $GITHUB_OUTPUT
          echo "AWS_S3_ENDPOINT=${{ vars.AWS_S3_ENDPOINT }}" >> $GITHUB_OUTPUT
          echo "AWS_S3_BUCKET=${{ vars.AWS_S3_BUCKET }}" >> $GITHUB_OUTPUT
          echo "AWS_S3_OBJECT=${{ vars.AWS_S3_OBJECT }}" >> $GITHUB_OUTPUT
          echo "SERVER_HOSTNAME=${{ vars.SERVER_HOSTNAME }}" >> $GITHUB_OUTPUT
          echo "SERVER_PORT=${{ vars.SERVER_PORT }}" >> $GITHUB_OUTPUT
          echo "SERVER_USERNAME=${{ vars.SERVER_USERNAME }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_SERVER_HOSTNAME=${{ vars.NOTIFY_SERVER_HOSTNAME }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_SERVER_PORT=${{ vars.NOTIFY_SERVER_PORT }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_SERVER_USERNAME=${{ vars.NOTIFY_SERVER_USERNAME }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_HOST=${{ vars.NOTIFY_MYSQL_HOST }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_PORT=${{ vars.NOTIFY_MYSQL_PORT }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_USER=${{ vars.NOTIFY_MYSQL_USER }}" >> $GITHUB_OUTPUT
          echo "NOTIFY_MYSQL_DATABASE=${{ vars.NOTIFY_MYSQL_DATABASE }}" >> $GITHUB_OUTPUT

  call-workflow-backup-files:
    name: Call Backup Database Workflow
    uses: maya-hive/cloud-backup-pipeline/.github/workflows/files.yml@v2.0.0 # Use the required release version
    secrets: inherit
    needs: vars
    with:
      FILE_BACKUP_RETAIN: ${{ needs.vars.outputs.FILE_BACKUP_RETAIN }}
      FILE_BACKUP_APPLICATION_PATH: ${{ needs.vars.outputs.FILE_BACKUP_APPLICATION_PATH }}
      AWS_REGION: ${{ needs.vars.outputs.AWS_REGION }}
      AWS_S3_BUCKET: ${{ needs.vars.outputs.AWS_S3_BUCKET }}
      AWS_S3_OBJECT: ${{ needs.vars.outputs.AWS_S3_OBJECT }}
      AWS_S3_ENDPOINT: ${{ needs.vars.outputs.AWS_S3_ENDPOINT }}
      SERVER_PORT: ${{ needs.vars.outputs.SERVER_PORT }}
      SERVER_HOSTNAME: ${{ needs.vars.outputs.SERVER_HOSTNAME }}
      SERVER_USERNAME: ${{ needs.vars.outputs.SERVER_USERNAME }}
      NOTIFY_SERVER_PORT: ${{ needs.vars.outputs.NOTIFY_SERVER_PORT }}
      NOTIFY_SERVER_HOSTNAME: ${{ needs.vars.outputs.NOTIFY_SERVER_HOSTNAME }}
      NOTIFY_SERVER_USERNAME: ${{ needs.vars.outputs.NOTIFY_SERVER_USERNAME }}
      NOTIFY_MYSQL_HOST: ${{ needs.vars.outputs.NOTIFY_MYSQL_HOST }}
      NOTIFY_MYSQL_PORT: ${{ needs.vars.outputs.NOTIFY_MYSQL_PORT }}
      NOTIFY_MYSQL_USER: ${{ needs.vars.outputs.NOTIFY_MYSQL_USER }}
      NOTIFY_MYSQL_DATABASE: ${{ needs.vars.outputs.NOTIFY_MYSQL_DATABASE }}
```

### Development/Testing

Use nektos/act to run GitHub workflows locally:

```bash
act --secret-file .env workflow_call -v
```
