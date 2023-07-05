---
title: Usage of duplicity for doodba
date: 2023-03-15 20:00:00 -300
categories: [doodba]
tags: [duplicity,backups]
---

# Why duplicity?

Duplicity is considered one of the best options for making backups because it offers several advantages over other backup tools:

* Encryption: Duplicity uses encryption to secure your backups, ensuring that your data remains private and protected. Achieved by using GPG.
* Incremental Backups: Duplicity uses an incremental backup strategy that only backs up the changes made to your files since the last backup, saving both storage space and backup time.
* Remote Backup: Duplicity can back up your data to remote locations, such as cloud storage or other servers, making it a versatile option for backup solutions.
* Compression: Duplicity can compress your data before backing it up, reducing the amount of storage space required for backups.
* Open Source: Duplicity is open-source software, meaning it is free to use and modify, and its source code is available for inspection and review.

Overall, *Duplicity is a reliable, flexible, and secure backup solution* that offers advanced features that can be customized to suit your backup needs.

# Operations

## Inspecting backups

### List backups available in a bucket

```bash
duplicity --file-prefix-archive archive-backup- --file-prefix-manifest manifest-backup- --file-prefix-signature signature-backup- collection-status boto3+s3://bucket/prefix
```

### List current files in backup

```bash
duplicity --file-prefix-archive archive-backup- --file-prefix-manifest manifest-backup- --file-prefix-signature signature-backup- list-current-files boto3+s3://bucket/prefix
```

## Downloading and Restoring

### Extract a specific file

Use parameter --file-to-restore

```bash
duplicity --file-prefix-archive archive-backup- --file-prefix-manifest manifest-backup- --file-prefix-signature signature-backup- restore --file-to-restore=db.sql boto3+s3://bucket/prefix db.sql
```

### Extract all the files in the backup

```bash
duplicity --file-prefix-archive archive-backup- --file-prefix-manifest manifest-backup- --file-prefix-signature signature-backup- restore boto3+s3://bucket/prefix .
```
