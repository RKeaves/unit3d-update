# UNIT3D Update & Upgrade Tutorial

![Upgrade](https://img.shields.io/badge/Upgrade-UNIT3D%20v8.3.3%20%E2%86%92%20v9.0.1%20%7C%20PHP%208.3%20%E2%86%92%20PHP%208.4-yellowgreen)

<p align="center">
  <img src="https://ptpimg.me/6o8x8j.png" alt="UNIT3D Logo" style="width: 12%;">
</p>

_Whether you're upgrading from UNIT3D v8.3.3 to v9.0.1 or updating your PHP environment from 8.3 to 8.4, This tutorial offers a detailed guide to the entire process.​_

---

<div style="border: 2px solid #e74c3c; background-color: #f9e6e6; padding: 10px; border-radius: 5px; margin: 15px 0;">
  <strong>🚨 IMPORTANT:</strong> Before starting the update, always create a backup of your current installation! In this tutorial, we assume you’ve already created and securely stored your latest backup.
</div>


## Create a Backup

Regular backups protect your UNIT3D installation. Use PHP Artisan to securely back up your files and database.

### Step-by-Step Backup Process

1. **Navigate to Your Project Directory:**

   ```bash
   cd /var/www/html
   ```

2. **Run the Artisan Backup Command:**

   ```bash
   php artisan backup:run
   ```

3. **View Available Backups:**

   To list all generated backups, execute:

   ```bash
   php artisan backup:list
   ```

### unit3d-backup-restore

[![GitHub Backup & Restore Tutorial](https://img.shields.io/badge/UNIT3D%20Backup%20%26%20Restore-Tutorial-blue?style=flat-square)](https://github.com/RKeaves/unit3d-backup-restore)  

> For a more detailed guide on creating and managing backups, you may want to check out this other [tutorial](https://github.com/RKeaves/unit3d-backup-restore)  🚀 


---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Enter Maintenance Mode & Prepare Meilisearch](#step-1-enter-maintenance-mode--prepare-meilisearch)
- [Step 2: Update PHP to 8.4](#step-2-update-php-to-84)
- [Step 3: Update UNIT3D](#step-3-update-unit3d)
- [Step 4: Database Migration Fix](#step-4-database-migration-fix)
- [Step 5: Final Reset & Cleanup Commands](#step-5-final-reset--cleanup-commands)
- [Notes](#notes)
- [Acknowledgements](#acknowledgements)

---

## Prerequisites

- **Backup:** A complete backup of your current UNIT3D installation  (for example, stored in `~/tempBackup`).
- **Sudo Access:** Required for system and package management commands.
- **PHP & UNIT3D Versions:**
  - Current UNIT3D version: **v8.3.3**
  - Target UNIT3D version: **v9.0.1**
  - PHP version update from **8.3 to 8.4**
- **Tools:** `curl`, `apt`, `nginx`, `mysql` (or your chosen database client), and basic text editors like `nano` or `micro`.

---

## Step 1: Enter Maintenance Mode & Prepare Meilisearch

Before making any updates, put your site into maintenance mode. This ensures users do not experience issues during the upgrade.

```bash
cd /var/www/html
php artisan down
```

**Explanation:**
- `php artisan down` sets your application to maintenance mode.

Next, update Meilisearch as the new version requires a newer version. First, update Meilisearch by stopping the current service, installing the new version, and clearing its data.

```bash
sudo apt update
sudo apt upgrade
sudo systemctl stop meilisearch
sudo curl -L https://install.meilisearch.com | sudo sh
sudo mv ./meilisearch /usr/local/bin/
sudo chmod +x /usr/local/bin/meilisearch
sudo rm -rf /var/lib/meilisearch/data
```

**Explanation:**
- `sudo apt update && sudo apt upgrade`: Refreshes package lists and upgrades available packages.
- `sudo systemctl stop meilisearch`: Stops the current Meilisearch service.
- `sudo curl -L https://install.meilisearch.com | sudo sh`: Downloads and installs the latest Meilisearch.
- `sudo mv` and `sudo chmod +x`: Move the binary to `/usr/local/bin/` and make it executable.
- `sudo rm -rf /var/lib/meilisearch/data`: Clears previous Meilisearch data to avoid conflicts.

Proceed with the following command:

```bash
sudo chmod +x /usr/local/bin/meilisearch
sudo systemctl start meilisearch
```

> **Note:**
> This step had to be repeated every time.

**Explanation:**
- These commands ensure that Meilisearch has the correct executable permissions and is restarted.

---

## Step 2: Update PHP to 8.4

Now, update your PHP installation to version 8.4.

1. **Add PHP Repository & Update Package Lists:**

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

**Explanation:**
- `sudo add-apt-repository ppa:ondrej/php` adds a repository that contains updated PHP packages.
- `sudo apt update` refreshes the package list after adding the new repository.

2. **Backup Current PHP Packages & Install PHP 8.4:**

```bash
sudo dpkg -l | grep php | tee ~/tempBackup/packages.txt
sudo apt install php8.4-common php8.4-cli php8.4-fpm php8.4-{redis,bcmath,curl,dev,gd,igbinary,intl,mbstring,mysql,opcache,readline,xml,zip}
```

**Explanation:**
- `sudo dpkg -l | grep php | tee ~/tempBackup/packages.txt` creates a backup list of currently installed PHP packages.
- The `sudo apt install` command installs PHP 8.4 along with required modules.

3. **Update Nginx Configuration:**

3.1. Open the Nginx configuration file:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

3.2. Update the `fastcgi_pass Directive` to Use PHP 8.4 Socket

> **Note:**
> Find the line with `fastcgi_pass unix:/var/run/php/***.sock;` and update it to point to the PHP 8.4 socket (e.g., `fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;`).


Locate the line containing:
   ```nginx
   fastcgi_pass unix:/var/run/php/***.sock;
   ```

   and update it to:
   
   ```nginx
   fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
   ```

3.3. Save and exit the editor.

3.4. Test the Nginx configuration:
   ```bash
   sudo nginx -t
   ```


> **Note:**
> Replace `***` with your specific site identifier (e.g., `unit3d` or `php8.4`).



4. **Restart Nginx and PHP-FPM, and Remove PHP 8.3:**

```bash
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl restart php8.4-fpm 
sudo systemctl stop php8.3-fpm
sudo apt purge '^php8.3.*'
php -v
```

**Explanation:**
- `sudo nginx -t` tests the Nginx configuration for errors.
- Restarting Nginx and PHP-FPM ensures that your web server uses the new PHP version.
- Purging PHP 8.3 cleans up outdated packages.
- `php -v` verifies that the installed PHP version is correct.

---

## Step 3: Update UNIT3D

With PHP updated, proceed with the UNIT3D update:

```bash
cd /var/www/html
php artisan git:update

```
During the update process, UNIT3D differences all files and prompts the user to choose whether to keep the current version or update to the new one; typically, it is recommended to update all files, but before beginning the update, the most recent backup should be copied to `~/tempBackup`, and after the update, the list of file conflicts is saved as `~/tempBackup/fileConflicts.txt` so that, once the process is complete, these files can be reviewed to determine which changes should be merged or discarded.

**Explanation:**
- `php artisan git:update` initiates the UNIT3D update process by comparing your current files with the updated version.
- During the update, you will be prompted to choose whether to keep or update files. It is recommended to update all files. For best practices, copy the most recent backup to `~/tempBackup` before proceeding.

After the update, it's a good idea to save any file conflicts for review:

```bash
# (Example command to copy the conflict list; adjust as needed)
cp ~/tempBackup/fileConflicts.txt ~/tempBackup/fileConflicts_$(date +%Y%m%d%H%M%S).txt
```

---

## Step 4: Database Migration Fix

During the update, you might encounter an error related to the tickets table:

```
2025_02_17_074140_update_columns_to_boolean ......................................................................................... 38.50ms FAIL

In Connection.php line 825:                                                                                                              
  SQLSTATE[22004]: Null value not allowed: 1138 Invalid use of NULL value (Connection: mysql, SQL: alter table `tickets` modify `staff_read` tinyint(1) not null default '0')                                                                                                                 

In Connection.php line 571:                     
  SQLSTATE[22004]: Null value not allowed: 1138 Invalid use of NULL value
```

**Resolution:**

1. **Log in to MySQL:**  
   Open a terminal and enter the following command (replace `your_username` with your actual MySQL username):

   ```bash
   mysql -u your_username -p
   ```

   When prompted, enter your MySQL password. Once logged in, select the appropriate database by running:

   ```sql
   USE your_database_name;
   ```

2. **Fix the Null Values:**  
   Run the following SQL command to update any null entries in the `staff_read` column to `0`:

   ```sql
   UPDATE tickets SET staff_read = 0 WHERE staff_read IS NULL;
   ```

3. **Exit MySQL:**  
   After executing the command, type:

   ```sql
   exit;
   ```

4. **Complete the Migrations:**  
   Back in the terminal, run:

   ```bash
   php artisan migrate
   ```

**Explanation:**  
- Logging in to MySQL allows direct interaction with the database.  
- The SQL command updates the `staff_read` column by replacing any `NULL` values with `0`.  
- Finally, `php artisan migrate` applies the remaining database migrations.
---

## Step 5: Final Reset & Cleanup Commands

After updating and migrating, run the following commands to clear caches, reinstall dependencies, rebuild assets, and restart services:

```bash
sudo -u www-data composer install --prefer-dist --no-dev -o && \
sudo php artisan cache:clear && \
sudo php artisan queue:clear && \
sudo php artisan auto:email-blacklist-update && \
sudo php artisan auto:cache_random_media && \
sudo php artisan set:all_cache && \
bun install && \
bun run build && \
sudo php artisan migrate && \
sudo systemctl restart php8.4-fpm && \
sudo php artisan queue:restart && \
sudo supervisorctl reread && \
sudo supervisorctl update && \
sudo supervisorctl reload && \
sudo php artisan scout:sync-index-settings && \
sudo php artisan auto:sync_torrents_to_meilisearch --wipe && \
sudo php artisan auto:sync_people_to_meilisearch
```

**Explanation:**
- **Composer Install:** Reinstalls PHP dependencies.
- **Cache & Queue Clear:** Clears application cache and queued jobs.
- **Auto-Update Commands:** Update email blacklist, cache random media, and set all caches.
- **Bun Commands:** Reinstall Node.js dependencies and rebuild assets.
- **Migrate & Restart:** Ensures database migrations are complete and PHP-FPM is restarted.
- **Supervisor & Scout:** Reloads supervisor configurations and synchronizes search index settings.

Finally, finish the update and bring your site back online:

```bash
sudo php artisan scout:sync-index-settings && \
sudo php artisan auto:sync_torrents_to_meilisearch --wipe && \
sudo php artisan auto:sync_people_to_meilisearch && \
sudo php artisan set:all_cache && \
sudo systemctl restart php8.4-fpm && \
sudo php artisan queue:restart && \
sudo php artisan up
```

**Explanation:**
- These final commands ensure that search indexes are in sync, caches are set, PHP-FPM and queues are restarted, and the site is taken out of maintenance mode with `php artisan up`.

---

## Notes

- **Backup First:** Always backup your installation before starting an update.
- **Log Files:** Check Laravel and system logs for detailed error messages if something goes wrong.

---

## Acknowledgements

This project was made possible thanks to airclay aka [ericlay](https://github.com/ericlay).

---

_If you have questions or run into issues, feel free to open an issue or pull request with suggestions._
