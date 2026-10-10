# Teamcenter Weekly Maintenance

A Windows batch script for the weekly maintenance and governance routine of a **Siemens Teamcenter 2606** deployment (4-tier architecture, Windows Server). It is the weekly companion to the daily health check (`tc-daily-maintenance`).

The script asks for confirmation before it starts, then writes everything to a dated log file.

## What it does

| Step | Task | Output |
|------|------|--------|
| 0 | Confirmation prompt, then loads `tc_profilevars.bat` | Exits with code 1 if the profile is missing |
| 1 | Copies `*.syslog` and `*.log` from `TC_ROOT\logs` to `ARCHIVE_DIR` and compresses them | `logs_<date>.zip` |
| 2 | FSC cache purge using `fscadmin ./purge` | Log entry |
| 3 | Runs `TC_DATA\db_stats.sql` through `sqlplus` if present | Log entry |
| 4 | Volume orphan file audit with `review_volumes` | `orphan_files_report_<date>.txt` |
| 5 | Deletes files older than `RETENTION_DAYS_TEMP` (7) from `TC_DATA\tmp` | Log entry |
| 6 | Exports site preferences with `preferences_manager` | `site_preferences_<date>.xml` in `BACKUP_DIR` |
| 7 | Copies BMIDE data model `*.xml` and `*.zip` from `TC_DATA\model` | `bmide_templates_<date>\` |
| 8 | Extracts "Login successful" lines from the syslogs | `login_summary_<date>.txt` |
| 9 | Disk usage report for all drives | `weekly_report.txt` |
| 10 | Copies `TC_ROOT\pool_manager\confs` | `pool_manager_confs_<date>\` |

## Repository layout

```
tc-weekly-maintenance/
├── src/TC_WEEKLY_MAINTENANCE.bat   # the script
├── config/settings.example.txt     # reference for the variables to edit
├── docs/SCHEDULING.md              # Task Scheduler setup
├── docs/KNOWN_ISSUES.md            # review items before production use
├── CHANGELOG.md
├── LICENSE
└── .gitignore
```

## Requirements

- Windows Server with Teamcenter 2606
- Oracle client with `sqlplus` on `PATH`
- PowerShell 5 or later (`Compress-Archive` and disk report)
- `review_volumes` and `preferences_manager` available in `TC_ROOT\bin` and on `PATH`
- An account with access to the log, archive and backup folders and the database

## Configuration

Edit the variables at the top of the script. `config/settings.example.txt` lists them all. The values are generic placeholders.

**Do not commit real passwords.** The script has `[PLACEHOLDER_PASSWORD]` in three places (steps 3, 4 and 6). See `docs/KNOWN_ISSUES.md`.

## Usage

```bat
src\TC_WEEKLY_MAINTENANCE.bat
```

The script prompts `Are you sure you want to run the weekly maintenance routine? (Y/N)`. Anything other than `Y` cancels. Because of this prompt and the final `PAUSE`, the script needs a small change before it can run unattended (see `docs/SCHEDULING.md`).

## Exit codes

- `0`: completed, or cancelled at the prompt
- `1`: critical environment error (`tc_profilevars.bat` not found)

Individual step failures are written to the log and do not change the exit code.

## Disclaimer

Test in a non-production environment first. The script deletes temporary files and copies, compresses and exports data.

## License

MIT, see `LICENSE`.
