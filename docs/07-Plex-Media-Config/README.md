## 7. Configuring Plex Media Libraries & Troubleshooting Media Issues

**Goal:** To prepare the necessary storage for Plex Media Server, integrate media files, and diagnose/resolve common issues encountered during initial media playback.

**Key Technologies:**
* Linux File System (Directories, Permissions)
* `mkdir`, `chown`, `chmod` (Linux commands for directory/permission management)
* `systemctl` (for restarting Plex service)
* WinSCP (SFTP client for file transfer)
* HandBrake (.NET Desktop Runtime dependency, video encoding software)
* `ls -ld` (Linux command for directory listing/permissions)

**Process & Key Steps:**

1.  **Preparing Media Storage on Ubuntu VM:**
    * Created a dedicated directory structure for Plex media within the `ubuntu-practice` VM:
        ```bash
        mkdir /home/jesse/PlexMedia
        mkdir /home/jesse/PlexMedia/Movies
        ```
    * **Initial Permissions & Ownership Setup (Attempt):**
        * Ran initial commands to adjust ownership and permissions, aiming for Plex to access files:
            ```bash
            sudo chown -R jesse:jesse /home/jesse/PlexMedia
            sudo chmod -R 755 /home/jesse/PlexMedia
            sudo chmod g+s /home/jesse/PlexMedia
            ```
        * Restarted Plex Media Server (`sudo systemctl restart plexmediaserver`).

2.  **Transferring Test Media File:**
    * Used **WinSCP** (an SFTP client) to securely transfer a small test video file (`batcave.mp4`) from my local computer to the `ubuntu-practice` VM's `/home/jesse/PlexMedia/Movies/` directory.

**What Went Wrong & How I Fixed It (Media Folders Not Visible in Plex - Persistent Permissions!):**
Despite initial permission commands, the `PlexMedia` and `Movies` folders were not visible when Browse for media in the Plex Web UI. This indicated Plex (running as the `plex` user) lacked the necessary read/execute permissions.

* **Diagnosis:**
    * Used `ls -ld /home/jesse/PlexMedia/` in the SSH terminal.
    * The output showed `drwx--Sr-x` permissions for `PlexMedia/`. This indicated that while the `plex` group was the owner, the crucial execute permission (`x`) was missing for the group, preventing traversal.
* **Solution:**
    * Explicitly set the correct permissions on the `PlexMedia` directory, granting full read/write/execute to both the owner (`jesse`) and the group (`plex`):
        ```bash
        sudo chmod 775 /home/jesse/PlexMedia
        ```
    * Verified the permissions with `ls -ld`, confirming `drwxrwxr-x`.
    * Restarted `plexmediaserver` service again (`sudo systemctl restart plexmediaserver`).
* **Result:** The `PlexMedia` and `Movies` folders became visible in the Plex Web UI file browser.

3.  **Adding Media Library in Plex:**
    * In the Plex Web UI, navigated to "Libraries" and selected "Add Library."
    * Chose "Movies" as the library type.
    * Named the library "My Test Movies."
    * **Crucially:** Pointed Plex to the correct media folder: `/home/jesse/PlexMedia/Movies/`.
    * Added the library.

**What Went Wrong & How I Fixed It (Stretched/Distorted Video Playback):**
After the library was added, the `batcave.mp4` test video played back stretched and distorted in the Plex player.

* **Diagnosis:**
    * Initial troubleshooting (checking Plex player settings, refreshing metadata) did not resolve the issue.
    * Inspected the source video's properties in HandBrake: `Source Dimensions: 1080x1920`. This confirmed the video was recorded in **portrait (vertical) orientation**.
    * The stretching was a result of the Plex player attempting to fit a tall (portrait) video into a standard widescreen (landscape) display without proper orientation correction.
* **Solution:**
    * **Installed .NET Desktop Runtime:** HandBrake required this prerequisite. Downloaded and installed it on the local machine.
    * **Used HandBrake to Re-encode the Video:**
        * Opened `batcave.mp4` in HandBrake.
        * Selected a standard preset (e.g., `Fast 1080p30`).
        * **Crucial Fix:** In the **"Dimensions" tab**, manually set the **"Rotation"** filter to **`90 degrees (clockwise)`** or **`270 degrees (clockwise)`** (experimented until the preview showed correct landscape orientation). This effectively rotated the video's orientation in the file itself.
        * Ensured "Anamorphic" was `None` and "Keep Aspect Ratio" was checked (for the now-rotated video).
        * Saved the new file with a distinct name (e.g., `batcave_fixed.mp4`) to avoid overwriting the original.
        * Started the encoding process.
    * **Transferred New File to VM:** Used WinSCP to transfer `batcave_fixed.mp4` to `/home/jesse/PlexMedia/Movies/`.
    * **Updated Plex Library:** Performed a "Scan Library Files" in Plex to pick up the new version. (Optionally deleted the original stretched file from the VM).
* **Result:** The `batcave_fixed.mp4` video played back correctly in Plex, with proper aspect ratio and no stretching.

**Lessons Learned from Media Setup:**
* **Linux Permissions are Key:** File and directory permissions are paramount for applications (like Plex) to access data. `chmod` and `chown` are essential tools.
* **Debugging with `ls -ld`:** Learned to use `ls -ld` to precisely inspect directory permissions.
* **Video Encoding Fundamentals:** Understood common video issues like aspect ratio/orientation mismatch and how to correct them using tools like HandBrake.
* **Persistence in Troubleshooting:** Complex issues often require multiple diagnostic steps and iterative solutions.
