# Cloud Backup Pipeline

This pipeline uses AWS compatible S3 storage to store application database and file backups. Also only retains a specified count of backups to prevent stale backups.

### Required GitHub Action Secrets

```.env
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
