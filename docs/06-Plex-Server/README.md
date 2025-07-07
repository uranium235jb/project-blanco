## 6. Deploying Plex Media Server

**Goal:** Install Plex Media Server on the `ubuntu-practice` VM to organize and stream personal media, leveraging the foundational home lab infrastructure.

**Key Technologies:**
* Ubuntu Server (Guest OS)
* `apt` package manager (for software installation)
* `curl`, `apt-transport-https` (for repository management)
* `plexmediaserver` package
* `systemctl` (for service management)
* `nano` (text editor for configuration)

**Process - Installation:**

1.  **Pre-installation Updates:**
    * Ensured the `ubuntu-practice` VM's package lists were updated and upgraded:
        ```bash
        sudo apt update
        sudo apt upgrade -y
        ```

2.  **Installing Necessary Dependencies:**
    * Installed `curl` (for downloading files) and `apt-transport-https` (for secure repository access):
        ```bash
        sudo apt install curl apt-transport-https -y
        ```

3.  **Adding Plex's Official Repository:**
    * **Added Plex's GPG Key:** This establishes trust for packages from Plex's repository:
        ```bash
        curl [https://downloads.plex.tv/plex-keys/PlexSign.key](https://downloads.plex.tv/plex-keys/PlexSign.key) | sudo apt-key add -
        ```
        *(Note: Encountered a deprecation warning for `apt-key`, but the key was successfully added.)*
    * **Adding Plex's Repository File:**
        **What Went Wrong:** The initial `echo ... | sudo tee ...` command to create `/etc/apt/sources.list.d/plexmediaserver.list` failed with a "No such file or directory" error. This was due to `sudo` not correctly applying permissions to the redirection part of the command.
        **How I Fixed It:** Used a more robust method to ensure the file was created with proper root privileges:
        ```bash
        sudo sh -c 'echo deb [https://downloads.plex.tv/repo/deb](https://downloads.plex.tv/repo/deb) public main > /etc/apt/sources.list.d/plexmediaserver.list'
        ```
        The content of the file was verified with `cat /etc/apt/sources.list.d/plexmediaserver.list`.
        **Result:** `apt update` subsequently confirmed the Plex repository was correctly recognized, showing `Get https://downloads.plex.tv/repo/deb public InRelease`.

4.  **Updating Package Lists (for New Plex Repo):**
    * Refreshed the package list again to include the newly added Plex repository:
        ```bash
        sudo apt update
        ```

5.  **Installing Plex Media Server Software:**
    * Installed the main Plex Media Server package:
        ```bash
        sudo apt install plexmediaserver -y
        ```

6.  **Verifying Plex Service Status:**
    * Confirmed the `plexmediaserver` service was running automatically after installation:
        ```bash
        sudo systemctl status plexmediaserver
        ```
        Output showed `Active: active (running)`.

**Process - Initial Web Setup:**

1.  **Accessing Plex Web UI:**
    * Accessed the Plex Media Server's web interface from a local browser: `http://10.0.0.101:32400/web` (using the `ubuntu-practice` VM's IP address and Plex's default port).
2.  **Plex Account Sign-in/Sign-up:**
    * Signed in with an existing Plex account or created a new free account.
3.  **Initial Server Setup Wizard:**
    * Named the Plex server (e.g., "Blanco Films").
    * **Crucially:** Left "Allow me to access my media outside my home" **unchecked** for initial local setup.
    * Skipped adding media libraries at this stage, as dedicated storage was not yet prepared.
    * Skipped Plex Pass offers.
    * Finalized the setup, landing on an empty Plex dashboard.

**Lessons Learned:**
* **Linux Repository Management:** Gained practical experience adding third-party `apt` repositories securely.
* **Troubleshooting File Permissions:** Learned a robust method (`sudo sh -c`) for creating system files with correct permissions when `sudo tee` fails with redirection.
* **Basic Plex Server Setup:** Understood the initial server naming and wizard flow.
