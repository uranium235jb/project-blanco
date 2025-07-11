## 9. Deploying Your Personal Cloud with Nextcloud

**Goal:** To establish a private, self-hosted cloud storage and collaboration platform using Nextcloud, accessible from within your home network. This enhances Project Blanco's capabilities for personal data management and sharing.

**Key Technologies:**
* Nextcloud Hub (Application Software)
* Apache2 (Web Server)
* MariaDB (Database Server)
* PHP (Scripting Language)
* Linux Command Line Interface (CLI)
* `nano` (Text Editor)
* `systemctl` (Service Manager)
* `mysql` (MariaDB Client)

**Process & Key Steps:**

1.  **Phase 1: Prepare Ubuntu VM & Install Core Web Server/Database Packages**
    * **Objective:** Install foundational software for web hosting and database management.
    * **Actions:**
        * Updated `ubuntu-practice` VM: `sudo apt update && sudo apt upgrade -y`
        * Installed Apache2, MariaDB-server, and `libapache2-mod-php`: `sudo apt install apache2 mariadb-server libapache2-mod-php -y`
    * **Verification:** Confirmed `apache2` and `mariadb` services were `active (running)` via `sudo systemctl status`. Verified Apache's default "It works!" page at `http://10.0.0.101/`.

2.  **Phase 2: Configure MariaDB Database**
    * **Objective:** Secure MariaDB and create a dedicated database and user for Nextcloud.
    * **Actions:**
        * **Secured MariaDB:** Ran `sudo mysql_secure_installation`.
            * **What Went Wrong:** Initially, accidentally typed 'n' for "Set root password?"
            * **How I Fixed It:** Re-ran `mysql_secure_installation`, ensuring 'Y' was selected for setting the root password, and 'Y' for all subsequent security hardening questions (removing anonymous users, disallowing remote root login, removing test database, reloading privileges).
            * **Result:** MariaDB `root` user password set securely.
        * **Created Nextcloud Database & User:** Logged into MariaDB as `root` (`sudo mysql -u root -p`) and executed SQL commands.
            * `CREATE DATABASE nextcloud_db;`
            * `CREATE USER 'nextcloud_user'@'localhost' IDENTIFIED BY 'YOUR_DB_PASSWORD';` (This was later changed due to security)
                * **What Went Wrong (1):** Initially copied `YOUR_DB_PASSWORD` literally, leaving the placeholder as the password.
                * **How I Fixed It:** Immediately used `ALTER USER 'nextcloud_user'@'localhost' IDENTIFIED BY 'YOUR_NEW_STRONG_PASSWORD_HERE';` to set a *new, strong, unique, and unshared* password for `nextcloud_user`.
                * **What Went Wrong (2):** Forgot to enclose the password in single quotes (`'`) in the `ALTER USER` command, leading to SQL syntax errors.
                * **How I Fixed It:** Re-executed `ALTER USER` with the password correctly enclosed in single quotes.
                * **What Went Wrong (3):** `nextcloud_user` initially had `GRANT USAGE` but no actual privileges on `nextcloud_db` (Error 1044).
                * **How I Fixed It:** Executed `GRANT ALL PRIVILEGES ON nextcloud_db.* TO 'nextcloud_user'@'localhost';` to correct the user's database privileges.
            * `FLUSH PRIVILEGES;`
            * `quit;`
            * **Result:** `nextcloud_db` created, `nextcloud_user` created with correct password and full privileges on `nextcloud_db`.

4.  **Phase 3: Install PHP and Required Modules**
    * **Objective:** Install PHP extensions and tune `php.ini` for Nextcloud.
    * **Actions:**
        * Installed PHP modules: `sudo apt install php-xml php-zip php-curl php-gd php-json php-mysql php-mbstring php-imagick php-intl php-bcmath php-gmp php-ldap -y`
        * Configured `php.ini` (located at `/etc/php/8.3/apache2/php.ini` for PHP 8.3):
            * `upload_max_filesize = 1G`
            * `post_max_size = 1G`
            * `memory_limit = 512M`
            * `max_execution_time = 360`
            * `date.timezone = America/Chicago` (uncommented and set)
        * Restarted Apache2: `sudo systemctl restart apache2`
    * **Verification:** Confirmed Apache2 was `active (running)`.

5.  **Phase 4: Download & Place Nextcloud Files**
    * **Objective:** Obtain Nextcloud software and place it in the web server's accessible directory.
    * **Actions:**
        * Downloaded `latest.zip` from `download.nextcloud.com`: `curl -o nextcloud.zip ...`
        * Installed `unzip`: `sudo apt install unzip -y`
        * Extracted Nextcloud to `/var/www/`: `sudo unzip nextcloud.zip -d /var/www/` (creating `/var/www/nextcloud/`).
        * Set correct ownership for Apache: `sudo chown -R www-data:www-data /var/www/nextcloud/`
    * **Verification:** Confirmed files extracted and ownership set.

6.  **Phase 5: Configure Apache Web Server for Nextcloud**
    * **Objective:** Create a virtual host configuration for Apache to serve the Nextcloud application.
    * **Actions:**
        * Created `nextcloud.conf` at `/etc/apache2/sites-available/`:
            ```apache
            <VirtualHost *:80>
                ServerAdmin webmaster@localhost
                DocumentRoot /var/www/nextcloud/
                <Directory /var/www/nextcloud/>
                    Require all granted
                    AllowOverride All
                    Options FollowSymLinks MultiViews
                    <IfModule mod_dav.c>
                        Dav off
                    </IfModule>
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined
            </VirtualHost>
            ```
            * **What Went Wrong:** Initially missed the opening `<` on `<VirtualHost *:80>`.
            * **How I Fixed It:** Manually edited the file in `nano` to add the missing character.
        * Enabled `nextcloud.conf`: `sudo a2ensite nextcloud.conf`
        * Enabled required Apache modules: `sudo a2enmod headers env dir mime setenvif`
        * Disabled default Apache site: `sudo a2dissite 000-default.conf`
        * Restarted Apache2: `sudo systemctl restart apache2`
    * **Verification:** `sudo apache2ctl configtest` showed `Syntax OK`. `ls -l /etc/apache2/sites-enabled/` confirmed `nextcloud.conf` was enabled and `000-default.conf` was not.

7.  **Phase 6: Run Nextcloud Web Installer (Initial Setup)**
    * **Objective:** Complete the final Nextcloud setup via the web interface.
    * **Actions:**
        * **Accessed Nextcloud Web UI:**
            * **What Went Wrong:** Initially attempted to access at `http://10.0.0.101/nextcloud`, resulting in a "404 Not Found."
            * **How I Fixed It:** Realized `DocumentRoot /var/www/nextcloud/` meant the base URL was simply `http://10.0.0.101/`. Corrected the URL.
            * **Result:** The Nextcloud setup page loaded successfully at `http://10.0.0.101/`.
        * **Completed Setup Wizard:**
            * Created Nextcloud admin account (username, strong password).
            * Left Data Folder as default (`/var/www/nextcloud/data`).
            * Selected `MariaDB/MySQL` for database.
            * Provided `nextcloud_user` for database user.
            * Provided the *new, strong, and unshared* password for `nextcloud_user`.
            * Provided `nextcloud_db` for database name.
            * Left `localhost` for database host.
            * Click `Install`.
            * **What Went Wrong:** Encountered "Access denied for user 'nextcloud_user'@'localhost' to database 'nextcloud_db'" (Error 1044).
            * **How I Fixed It:** Logged back into MariaDB as `root` and executed `GRANT ALL PRIVILEGES ON nextcloud_db.* TO 'nextcloud_user'@'localhost';` to correct the user's database privileges. Flushed privileges and re-attempted installation.
            * **Result:** Nextcloud installation completed successfully, followed by installation of recommended apps.

**Key Outcomes / Results:**
* **Fully functional Nextcloud Hub instance** deployed on Project Blanco, accessible via web browser at `http://10.0.0.101/`.
* Successfully configured Apache, MariaDB, and PHP to support Nextcloud.
* **Seamless mobile app integration:** Downloaded the official Nextcloud mobile app (Android/iOS) and successfully logged in using the server address `http://10.0.0.101/` and the Nextcloud admin credentials. The app works flawlessly for file Browse and management.

**Lessons Learned from Nextcloud Deployment:**
* **Database User Privileges:** Understanding the difference between `GRANT USAGE` (connect only) and `GRANT ALL PRIVILEGES` (full access) for database users, and the critical need for correct database user passwords.
* **Web Server `DocumentRoot` vs. URL Path:** The crucial distinction between the server's file system root and the URL path.
* **PHP Configuration (`php.ini`):** Tuning PHP settings for application performance and file handling.
* **Apache Virtual Host Syntax:** Meticulous attention to detail in configuration files (`<VirtualHost>` tag).
* **Importance of Internal Access:** Successfully deployed and verified a complex web application for internal network use.
* **Mobile App Integration:** Experienced the ease of connecting standard mobile apps to a self-hosted service.
* **Persistence in Troubleshooting:** Overcame multiple, varied errors (database access, syntax, URL paths) through systematic diagnosis and correction.
