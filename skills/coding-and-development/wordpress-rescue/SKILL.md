---
name: wordpress-rescue
description: Diagnose and fix WordPress errors from the actual debug.log, with a safe rollback procedure. Use when a WordPress site shows a white screen, an HTTP 500 error, a database connection error, a redirect loop, or a plugin or theme conflict, and you need to find the cause before changing anything.
---

# WordPress Rescue

Fix WordPress by reading the log first. Most WordPress errors are diagnosed in minutes from `wp-content/debug.log` and fixed with one change, but only if you find the cause before you start changing things. Guessing from the symptom alone is how a small error becomes a broken site.

## When to Use

- The site shows a white screen, an HTTP 500 error, or "Error Establishing a Database Connection".
- The site redirects in a loop, or the admin area is unreachable while the front end works (or the other way around).
- A plugin or theme update broke the site.
- A user reports "it worked yesterday" and nothing was changed on purpose.
- You have access to `wp-content/debug.log`, WP-CLI, FTP, or the hosting file manager.

## The First Rule: Read the Log

Never guess from the symptom alone. A white screen can be a fatal error in a plugin, a memory limit, a broken theme, or a database failure, and each one has a different fix. The log tells you which one it is.

1. Open `wp-content/debug.log`. If it does not exist, turn on logging first: in `wp-config.php`, set `WP_DEBUG` to `true` and `WP_DEBUG_LOG` to `true`, and set `WP_DEBUG_DISPLAY` to `false` so errors go to the file and not to visitors. Reproduce the error once, then read the log.
2. Read from the bottom up. The last fatal error is the one that stopped the page. Earlier warnings are noise unless they point at the same file.
3. Find the file path in the error. The path tells you the owner: `wp-content/plugins/<slug>/` is a plugin, `wp-content/themes/<slug>/` is the theme, `wp-includes/` or `wp-admin/` is WordPress core.
4. Run the diagnosis script on the log if it is in the premium pack: `python3 diagnose.py wp-content/debug.log`. It names the failing plugin or theme, the file and line, the error class, the fix, and the rollback step.

If the log is empty and the site is still broken, the error is not being logged. Check that `WP_DEBUG_LOG` is true, that the `wp-content` folder is writable, and that the failure happens after the point where WordPress loads `wp-config.php`. A redirect loop and a corrupt `.htaccess` file both leave an empty log, and the decision tree covers them.

If `debug.log` is empty or missing, follow the no-log path: check the server error log first (hosting panel, error log for the same minute as the failure), because it records errors WordPress never sees; then force logging on with the force-logging snippet from the premium pack, or by setting `WP_DEBUG` and `WP_DEBUG_LOG` to `true` and `WP_DEBUG_DISPLAY` to `false` in `wp-config.php`; reproduce the error once by loading the broken page; then re-read the log, or re-run `diagnose.py` on the new log file. The decision tree has a full entry for this (entry 10, no log at all).

## The Safe Rollback Procedure

Do this before any fix. Every fix below ends with a rollback step, and the rollback only works if you prepared it first.

1. Back up before you change anything. Take a full backup of the files and the database, or at minimum copy the exact file you are about to edit and export the database table you are about to touch. Name the backup with the date and the error, for example `backup-2026-11-16-before-plugin-fix`.
2. Make one change at a time. Change one plugin, one setting, or one file, then load the site. If you change three things at once, you will not know which one fixed it or which one broke it further.
3. Verify after every change. Load the front page and the admin login page, and check the log again. The error you fixed should be gone and no new fatal error should appear.
4. If the fix makes it worse, roll back immediately. Restore the file you copied in step 1, reactivate the plugin you deactivated, or restore the database export. Then re-read the log and pick the next fix. Do not keep pushing forward on a fix that made things worse.
5. Write down what you changed. One line per change, with the time. If the site breaks again tomorrow, that list is the first thing you read.

The premium pack has `rollback-checklist.md`, a checkbox version of this procedure to run before any fix.

## The Error Decision Tree

The full tree is in `references/error-decision-tree.md`. It covers ten errors. Each entry has the same four parts: what the log line looks like, the most likely cause, the fix, and the rollback.

1. White screen of death: a fatal error, almost always a plugin or theme. The log names the file.
2. Error Establishing a Database Connection: WordPress cannot reach the database. The log shows the MySQL error.
3. HTTP 500: a fatal error or a server limit. Read the log first, then check the server error log.
4. Redirect loops: the site redirects to itself. The log is usually empty; check `siteurl`, `home`, and `.htaccess`.
5. Plugin conflicts: two plugins, or one plugin and the theme, disagree. Deactivate in sequence to isolate the one.
6. Theme conflicts: the active theme breaks after an update or a PHP change. Switch to a default theme to test.
7. `.htaccess` corruption: a bad rewrite rule or a bad edit. Rename the file and let WordPress regenerate it.
8. PHP version and memory limits: a plugin needs a newer PHP, or the site runs out of memory. The log says which.
9. File permissions: WordPress cannot write uploads or updates. Wrong ownership or a 600 permission on a folder.
10. No log at all: `debug.log` is empty or missing. Check the server error log, force logging on, reproduce the error, then re-read the log.

Start at the symptom, follow the tree to the log line, and only then fix. Never fix a symptom without a log line or a test that names the cause.

## How to Isolate a Plugin Conflict

A plugin conflict means the site works with one plugin off and breaks with it on, or breaks only when two plugins are on together. Isolate it in sequence:

1. Back up first, using the rollback procedure above.
2. Deactivate all plugins. If the site works, the cause is a plugin. If it still fails, the cause is the theme or core; skip to the theme steps.
3. Reactivate plugins one at a time, in the same order they were active. Load the site after each one. The plugin that breaks it is the suspect.
4. If the site only breaks with two plugins active together, the conflict is between those two. Keep both off, then turn one on, then the other, and note the pair.
5. Leave the suspect off and confirm the site works. Then decide: update it, replace it, or contact its author with the exact log line.

With WP-CLI: `wp plugin deactivate --all`, then `wp plugin activate <slug>` one at a time. Without WP-CLI: rename each plugin folder under `wp-content/plugins/` to `<slug>.off`, one at a time, and rename it back to test. Renaming the folder is the same as deactivating it, and renaming it back is the rollback. If the admin area is unreachable and you cannot use WP-CLI or rename folders, the premium pack has `rescue.php`: upload it to the site root, visit it in a browser, confirm, and it disables all plugins and switches to a default theme. Delete it immediately after use; never leave it on a live site.

## How to Isolate a Theme Conflict

1. Back up first.
2. Switch to a default theme (Twenty Twenty-Four or Twenty Twenty-Five). If the site works, the active theme is the cause.
3. If the admin area is unreachable, switch the theme without the admin: with WP-CLI, `wp theme activate twentytwentyfour`. Without WP-CLI, rename the active theme folder under `wp-content/themes/` to `<slug>.off`; WordPress falls back to a default theme.
4. Rollback is switching back to the original theme, or renaming the folder back. `rescue.php` in the premium pack also switches the theme to a default theme; see the plugin section above for the delete-after-use warning.

## Worked Example: A Fatal Error in a Plugin

The sample log `examples/sample-debug-log-fatal-plugin.log` shows a fatal error in a booking form plugin. The diagnosis is in `examples/diagnosis-example.md`, and the steps are:

1. The last line of the log is a PHP Fatal error, `Call to undefined function sample_booking_render()`, in `/wp-content/plugins/sample-booking-form/includes/booking.php` on line 87. The path names the plugin: `sample-booking-form`.
2. The function does not exist, which means the plugin's main file did not load, or the plugin was partly updated. The fix is to deactivate the plugin, then update it fully or reinstall it from a fresh copy.
3. Rollback: reactivate the plugin from the backup copy, or restore the plugin folder from the backup taken before the fix. The site returns to the broken state it was in, and you try the next fix instead of leaving a half-fixed plugin in place.

## Gotchas

- Never edit a plugin or theme file to silence an error. Removing the line that errors hides the symptom and leaves the bug. Fix the cause or deactivate the plugin.
- Never raise the memory limit as the first fix. A memory error in one plugin means that plugin is doing something wrong; raising the limit lets it use more memory and fail later, on a bigger page. Fix the plugin first, raise the limit only if the log shows the site itself needs more.
- Never delete `debug.log` to "clean up" before you have read it. The log is the evidence. Rotate it (rename it to `debug-2026-11-16.log`) so a fresh log starts, and keep the old one until the fix is verified.
- Never run a fix on the live site without a backup. "It is only one line" is the sentence people say before they restore from a backup they did not take.
- A white screen with an empty log is not "no error". It means logging is off, the failure happens before WordPress loads, or the server is hiding the error. Check `WP_DEBUG_LOG`, then the server error log.
- Deactivating all plugins at once on a live shop or membership site changes what visitors see. Do it in a staging copy first if one exists, or warn the owner and work fast.

## Troubleshooting

| Symptom | Likely cause | First step |
|---|---|---|
| White screen, log names a plugin file | Fatal error in that plugin | Deactivate that plugin, verify, then update or replace it |
| White screen, log names a theme file | Fatal error in the theme | Switch to a default theme to test |
| White screen, log empty | Logging off, or failure before WordPress loads | Turn on `WP_DEBUG_LOG` (or the premium force-logging snippet), reproduce, re-read; then check the server error log |
| Database connection error | Wrong credentials, database down, or user permissions | Read the MySQL error in the log, check `wp-config.php` credentials |
| HTTP 500 on every page | Fatal error or server limit | Read the log, then check the server error log for the same time |
| Redirect loop | `siteurl` and `home` disagree, or a bad `.htaccess` rule | Check the two URLs in the database, then rename `.htaccess` and test |
| Site works, admin does not | A plugin that only runs in admin | Deactivate plugins in sequence from WP-CLI or FTP |
| Error after a PHP update | A plugin or theme needs the old PHP version | Read the log for the deprecated or removed function, then update the plugin or pin the PHP version |
| Uploads fail | File permissions or ownership | Check that `wp-content/uploads` is writable by the web server user |

## Delivery Rules

- Diagnose from the log first, every time. A fix without a log line is a guess; say so if you have to guess.
- One change at a time, verify after each, and keep the backup until the fix is verified.
- Never present a log from a real site in a report or an example. Sample logs in this skill are written by hand with fictional plugin names and paths.
- Never claim a fix was tested on a live site unless it was. A diagnosis from a sample log is a diagnosis of the sample, not of a real site.
