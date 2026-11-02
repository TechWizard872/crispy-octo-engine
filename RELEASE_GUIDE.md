# SkillGo: GitHub Pages Web Distribution & Release Guide 🚀

This guide explains how to host SkillGo for free on **GitHub Pages** and how to release in-app updates directly to users outside the Google Play Store.

---

## 1. GitHub Pages Configuration & URL Reference

Before publishing, configure your repository details.

### Your Configuration Placeholders:
```
GITHUB_USERNAME = "YOUR_USERNAME"
GITHUB_REPOSITORY = "YOUR_REPOSITORY"
```

### Expected Public URLs:
* **Website (Home & Download Page):**  
  `https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/`
* **Update Manifest (Read by the App):**  
  `https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/update-manifest.json`
* **Dynamic Content Feed (Curriculum & Announcements):**  
  `https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/content-feed.json`
* **Direct APK Download:**  
  `https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/downloads/skillgo-latest.apk`

*(Note: These URLs become active once you replace the placeholders and push the repository to GitHub).*

---

## 2. Setting Up GitHub Pages (One-Time Setup)

1. **Push your repository** to your GitHub account:
   ```bash
   git add .
   git commit -m "Initialize SkillGo with GitHub Pages web distribution"
   git push origin main
   ```
2. On GitHub, go to your repository:
   * Click **Settings** (tab at the top).
   * In the left sidebar, click **Pages**.
   * Under **Build and deployment** → **Source**, select:
     **`GitHub Actions`** (do not select "Deploy from a branch").
3. The included workflow `.github/workflows/deploy-pages.yml` will automatically build and publish the `/website/` directory to GitHub Pages on every push to `main`!

---

## 3. Configuring the Android App for Your GitHub Pages Site

Open `app/src/main/java/com/example/data/config/DistributionConfig.kt` and replace:
```kotlin
const val GITHUB_USERNAME = "YOUR_USERNAME"
const val GITHUB_REPOSITORY = "YOUR_REPOSITORY"
```
with your actual GitHub username and repository name.

Once set:
* SkillGo will periodically check your GitHub Pages `update-manifest.json` for updates.
* When a newer version is detected, users get an in-app update notification with your release notes.
* Tapping **"Update Now"** downloads the new APK from your GitHub Pages site and prompts Android's system package installer!

---

## 4. Releasing a New App Version (Step-by-Step)

Follow these steps whenever you add new app features or bug fixes:

### Step 1: Update App Version in `app/build.gradle.kts`
Increment `versionCode` and update `versionName`:
```kotlin
defaultConfig {
    // ...
    versionCode = 2          // Increment by 1 for every release
    versionName = "1.0.1"    // Or "1.1.0" for feature updates
}
```

### Step 2: Build the New APK
* In AI Studio: Export or download the built project APK.
* Or using Gradle in your terminal:
  ```bash
  gradle assembleDebug
  ```
  *(Output location: `app/build/outputs/apk/debug/app-debug.apk`)*

### Step 3: Copy the New APK to the Website
Copy and overwrite the APK at `website/downloads/skillgo-latest.apk`:
```bash
cp app/build/outputs/apk/debug/app-debug.apk website/downloads/skillgo-latest.apk
```

### Step 4: Update `website/update-manifest.json`
Update `latestVersion`, `latestVersionCode`, `releaseDate`, and your `releaseNotes`:
```json
{
  "latestVersion": "1.0.1",
  "latestVersionCode": 2,
  "minimumVersion": "1.0.0",
  "minimumVersionCode": 1,
  "downloadUrl": "https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/downloads/skillgo-latest.apk",
  "websiteUrl": "https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/",
  "releaseNotes": [
    "Improved learning streak stability.",
    "Resolved audio playback edge cases on older Android versions.",
    "Performance improvements across practice challenges."
  ],
  "releaseDate": "2026-10-15",
  "apkSizeMb": 30.0,
  "isEnabled": true
}
```

*(Note: The `index.html` website dynamically reads `update-manifest.json` via JavaScript, so the website's displayed version, date, and "What's New" list synchronize automatically!)*

### Step 5: Commit and Push to GitHub
```bash
git add app/build.gradle.kts website/update-manifest.json website/downloads/skillgo-latest.apk
git commit -m "Release SkillGo v1.0.1"
git push origin main
```

### Step 6: Verify Deployment & In-App Update
1. Open the GitHub Actions tab in your repository and wait for the `Deploy SkillGo Website to GitHub Pages` workflow to finish (~30 seconds).
2. Visit `https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/` and verify the download link and version display.
3. Open an existing installation of SkillGo (v1.0.0) on your device:
   * Go to **Profile → About SkillGo** → tap **Check for updates**.
   * You will see the update notification: **"SkillGo 1.0.1 is available! 🚀"**.

---

## 5. Concrete Release Examples

### Example A: Patch Release (1.0.0 → 1.0.1)
* **Reason:** Bug fix or performance patch.
* **`app/build.gradle.kts`:**
  ```kotlin
  versionCode = 2
  versionName = "1.0.1"
  ```
* **`website/update-manifest.json`:**
  ```json
  "latestVersion": "1.0.1",
  "latestVersionCode": 2,
  "minimumVersion": "1.0.0",
  "minimumVersionCode": 1,
  "releaseNotes": ["Fixed question timer reset glitch", "Updated lesson explanations"]
  ```
* **User Experience:**
  * Since `minimumVersion` is `1.0.0`, users on `1.0.0` can choose **"Update Now"** or **"Later"**.

---

### Example B: Minor Feature Release (1.0.1 → 1.1.0)
* **Reason:** New courses, redesigned lesson interactive engine, or major features.
* **`app/build.gradle.kts`:**
  ```kotlin
  versionCode = 3
  versionName = "1.1.0"
  ```
* **`website/update-manifest.json`:**
  ```json
  "latestVersion": "1.1.0",
  "latestVersionCode": 3,
  "minimumVersion": "1.0.0",
  "minimumVersionCode": 1,
  "releaseNotes": [
    "Added 2 new courses: Robotics Fundamentals & Cloud Architecture",
    "Enhanced Gemini AI Tutor explanations",
    "Audio feedback toggle added to Settings"
  ]
  ```

---

## 6. Updating Content Without Releasing an APK

If you only want to push new announcements, change the daily brain teaser challenge, or add AI coach prompt suggestions:
1. Edit `website/content-feed.json`.
2. Commit and push:
   ```bash
   git add website/content-feed.json
   git commit -m "Update daily challenge and announcement"
   git push origin main
   ```
3. Installed apps will automatically receive the new challenge and announcements on their next launch **without requiring users to download an APK update**!
