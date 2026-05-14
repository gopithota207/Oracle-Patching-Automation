# Oracle Database Automated Patching Script

A production-ready, end-to-end Oracle Database patching automation script that handles OPatch upgrade, patch application, database restart, post-patch SQL steps, validation, and email reporting — all in a single command.

---

## Supported Versions

- Oracle 12c - Supported
- Oracle 18c - Supported
- Oracle 19c - Supported
- Oracle 21c - Supported

OS: Linux (RHEL / OEL / CentOS / Ubuntu)
Patch Types: Release Updates (RU), Patch Set Updates (PSU), One-Off Patches

---

## What It Does — Step by Step

Step 0  - Pre-flight Check    - Validates root access, Oracle user, disk space, ulimits
Step 1  - Upgrade OPatch      - Auto-upgrades OPatch from stage directory if zip found
Step 2  - Discover & Unzip    - Finds patch zip, extracts to work directory
Step 3  - OPatch Version Check- Confirms active OPatch version
Step 4  - Prerequisite Check  - Conflict check, space check, minimum version check
Step 5  - Pre-Patch DB Health - Checks invalid objects, active sessions, RMAN status
Step 6  - Backup Inventory    - Backs up OPatch inventory before patching
Step 7  - Stop DB & Listener  - Gracefully disconnects sessions, shuts down DB
Step 8  - Apply Patch         - Runs opatch apply -silent, auto-rollback on failure
Step 9  - Start DB & Listener - Restarts listener and database
Step 10 - Post-Patch SQL      - Runs datapatch and utlrp.sql to recompile objects
Step 11 - Validate Patch      - Confirms patch in inventory, checks DB components
      - Report & Email      - Generates patch report, emails if configured

---

## Repository Structure

oracle-patch-automation/
├── oracle_patch_automation.sh    # Main patching script
├── README.md                     # This file
└── logs/                         # Auto-created at runtime
    ├── patch_ORCL_YYYYMMDD.log
    ├── patch_report_ORCL_YYYYMMDD.txt
    ├── prereq_YYYYMMDD.log
    └── opatch_apply_YYYYMMDD.log

---

## Configuration

Edit the configuration section at the top of oracle_patch_automation.sh:

ORACLE_BASE="/u01/app/oracle"
ORACLE_HOME="/u01/app/oracle/product/19.3.0/dbhome_1"
ORACLE_SID="ORCL"
ORACLE_USER="oracle"

PATCH_STAGE_DIR="/u01/patches/stage"   # Drop your patch zips here
PATCH_WORK_DIR="/u01/patches/work"     # Extracted patch location
LOG_DIR="/u01/patches/logs"            # All logs saved here

MAIL_TO="dba@yourcompany.com"          # Leave blank to skip email
AUTO_ROLLBACK="yes"                    # Auto rollback on failure

---

## Prerequisites

1. Place Patch Files in Stage Directory

/u01/patches/stage/
├── p6880880_190000_Linux-x86-64.zip     <- OPatch upgrade zip (optional)
└── p35742441_190000_Linux-x86-64.zip    <- Actual patch zip

Note: OPatch zip always starts with p6880880_. The script auto-detects both files.

2. Required OS Permissions

Script must be run as root:
sudo ./oracle_patch_automation.sh

3. Required Oracle Privileges

Oracle user must have SYSDBA access.

---

## Usage

Basic Run:
sudo ./oracle_patch_automation.sh

Override SID and send email report:
sudo ORACLE_SID=PROD ./oracle_patch_automation.sh --mail dba@example.com

Override Oracle Home and patch directory:
sudo ./oracle_patch_automation.sh --oracle-home /u01/app/oracle/product/19.3.0/dbhome_1 --patch-dir /u01/patches/stage --sid PROD

Disable auto-rollback on failure:
sudo ./oracle_patch_automation.sh --no-rollback

All Available Options:
  -b, --oracle-base   PATH    Override ORACLE_BASE
  -h, --oracle-home   PATH    Override ORACLE_HOME
  -s, --sid           SID     Override ORACLE_SID
  -p, --patch-dir     PATH    Override PATCH_STAGE_DIR
  -m, --mail          EMAIL   Send report to this email address
  --no-rollback               Disable auto-rollback on failure
  --help                      Show help message

---

## Auto-Rollback Feature

If AUTO_ROLLBACK=yes (default) and patch application fails, the script automatically runs:
opatch rollback -id <PATCH_ID> -silent
Then restarts the database to restore normal operations.

---

## Safety Features

- Ctrl+C / Kill signal handler — safely restarts DB if interrupted
- OPatch inventory backup before every patch
- Pre-patch health check — warns about invalid objects, RMAN jobs, active sessions
- Disk space validation — checks free space before patching
- Graceful session disconnect — kills user sessions before shutdown
- Auto-rollback — reverts patch if apply fails
- Full logging — every step logged with timestamps

---

## Log Files Generated

patch_<SID>_<timestamp>.log          - Full execution log
patch_report_<SID>_<timestamp>.txt   - Summary report (emailed)
prereq_<timestamp>.log               - OPatch prereq check output
opatch_apply_<timestamp>.log         - OPatch apply output
lsinventory_post_<timestamp>.txt     - Post-patch inventory
