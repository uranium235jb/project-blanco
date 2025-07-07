## 8. Enabling Plex Remote Access & Testing

**Goal:** To make the Plex Media Server accessible securely from outside the home network, allowing friends and family to stream content remotely.

**Key Technologies:**
* Plex Web UI (Remote Access settings)
* UPnP (Universal Plug and Play, router feature)
* Home Router (Nighthawk)
* Internet Connection
* Plex Client Apps (mobile, web browser)
* Email (for invitations)

**Process & Key Steps:**

1.  **Enabling Remote Access within Plex:**
    * Navigated to Plex Web UI (`http://10.0.0.101:32400/web`).
    * Went to "Settings" (wrench icon) -> "Remote Access."
    * Checked the "Enable Remote Access" box.
    * **Result:** Plex successfully configured the router, reporting "Fully accessible outside your network" within seconds. This indicated UPnP was active and working correctly on the Nighthawk router, automatically handling the necessary port forwarding (typically TCP port 32400).
    * **Lesson Learned:** UPnP can simplify router configuration for common services, though manual port forwarding is an alternative if UPnP fails or is disabled for security.

2.  **Inviting a Friend to Access the Library:**
    * To test remote access with an external user, invited a friend via Plex's sharing features.
    * **Process:**
        * In Plex Web UI -> "Settings" -> "Manage Library Access" -> "Grant Library Access."
        * Entered friend's Plex account email address.
        * Selected "My Test Movies" library for sharing.
        * Sent the invitation.
    * **Friend's Role:** Friend received the email invitation, accepted it, and logged into their Plex account on their mobile phone's native Plex app.

**What Went Wrong & How I Fixed It (Mobile App Payment Prompt for Friends):**
Upon attempting to play the video in the native Plex mobile app, my friend encountered a payment prompt, leading to confusion as Plex's core service is free.

* **Diagnosis:** This is a specific behavior of Plex's native mobile apps (iOS/Android). While the Plex Media Server and its web interface are free, native mobile apps often require a one-time "mobile app unlock" purchase (or a Plex Pass subscription) for full streaming functionality beyond a short trial. This is a monetization strategy for Plex's client development.
* **Solution:** Advised friend to use their phone's **web browser** and navigate to `https://app.plex.tv/desktop`. After logging in there, they were able to stream the video successfully without any payment prompts.
* **Result:** Confirmed the server was indeed remotely accessible and functional, overcoming the mobile app's payment barrier via the web interface.
* **Lessons Learned:**
    * Understanding the nuances of Plex's free vs. paid client model, particularly for native mobile apps.
    * The importance of a free web-based client (`app.plex.tv/desktop`) as an alternative or troubleshooting step.
    * Effective troubleshooting requires adapting solutions to the specific client/platform.

**Conclusion of Plex Deployment:**
The Plex Media Server is now fully deployed on Project Blanco, successfully organizing, streaming, and providing remote access to personal media. This phase highlighted critical aspects of server application deployment, network accessibility, user management, and client-side troubleshooting. The next steps for Plex will involve integrating larger storage solutions for a full media library.
