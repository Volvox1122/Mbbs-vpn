# MBBS VPN — Full Project

A real WireGuard-based VPN Android app with a Firebase admin panel, AdMob ads,
and extra study tools for MBBS students in Russia. Everything below can be
done from your **phone only** — no laptop, no Android Studio.

---

## What you have

- `app/` — the Android app source code (Kotlin)
- `admin-panel/index.html` — a webpage you host for free that lets you add/edit
  VPN servers and control ads, from any phone browser
- `.github/workflows/build-apk.yml` — tells GitHub's free cloud servers how
  to compile your APK
- `firestore.rules` — security rules so random people can't rewrite your server list

---

## STEP 1 — Get 2-3 VPN servers (WireGuard)

You need actual servers for people to connect through. Cheapest path:

1. Rent a VPS from any provider (Contabo, Hetzner, DigitalOcean, Vultr — all
   have phone-friendly web dashboards, from ~$4-5/month). Pick 2-3 different
   countries.
2. On each VPS, install WireGuard. The fastest way — SSH into the server
   (you can SSH from your phone using an app like Termux or JuiceSSH) and run
   the official installer script:
   ```
   curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
   bash wireguard-install.sh
   ```
   Follow the prompts. At the end it gives you:
   - Server public key
   - A client config with: client private key, client address, server endpoint
3. Write these down for each server — you'll paste them into the admin panel
   in Step 4.

**Note on scale**: this setup uses one shared client key per server (simplest
for a small friend group / classmates). It's not bank-grade multi-tenant
security — fine for personal/small-group use, not for thousands of anonymous
users. Say the word if you eventually want proper per-user key provisioning
(Cloud Functions) — that's a bigger build.

---

## STEP 2 — Create your Firebase project (free)

1. From your phone browser, go to https://console.firebase.google.com
2. Create a project (e.g. "mbbs-vpn")
3. Add an **Android app**: package name must be exactly `com.mbbs.vpnstudent`
4. Download the `google-services.json` it gives you
5. In your GitHub repo (Step 3), replace `app/google-services.json.SAMPLE`
   with this real file, renamed to exactly `google-services.json`
6. Also in Firebase Console, add a **Web app** (for the admin panel) — copy
   the `firebaseConfig` object it shows you
7. Paste that config into `admin-panel/index.html` where it says
   `YOUR_API_KEY` etc.
8. In Firebase Console → Build → Firestore Database → Create database
   (production mode) — then go to the Rules tab and paste in the contents
   of `firestore.rules` from this project, click Publish
9. In Firebase Console → Build → Authentication → get started → enable
   **Email/Password** → Users tab → Add user (this is YOUR admin login for
   the panel — nobody else can log in without this)

---

## STEP 3 — Push this project to GitHub and build the APK

1. Create a free GitHub account if you don't have one (github.com, works
   fine from phone browser)
2. Create a new repository (e.g. `mbbs-vpn`)
3. Upload all these files to it — easiest way on phone: use GitHub's
   "Add file → Upload files" in the web UI, or install the **GitHub mobile
   app** which lets you browse/edit/commit files directly
4. Once you've replaced `google-services.json.SAMPLE` with your real
   `google-services.json`, push/commit everything
5. Go to the **Actions** tab of your repo — you'll see "Build MBBS VPN APK"
   running automatically. Wait ~3-5 minutes.
6. When it finishes (green check), click into the run → scroll down to
   **Artifacts** → download `mbbs-vpn-debug-apk` → this is a zip containing
   your real, installable `.apk` file
7. On your phone, extract the zip, tap the `.apk`, allow "install unknown
   apps" if prompted — done, it's installed like any app

Every time you push a code change, GitHub automatically rebuilds it for you.

---

## STEP 4 — Add your servers from the admin panel

1. Host `admin-panel/index.html` for free — easiest: in your GitHub repo,
   go to Settings → Pages → set source to your branch, and it'll be live at
   `https://yourusername.github.io/mbbs-vpn/admin-panel/`
2. Open that link on your phone, log in with the admin email/password you
   created in Firebase Step 2.9
3. Fill in the "Add / Edit VPN Server" form using the values you got from
   Step 1 for each of your servers, tap Save
4. Open your VPN app — the servers appear instantly (live from Firestore,
   no app update needed)

From this same panel you can later: enable/disable any server instantly,
edit the ad frequency, turn all ads off, post an announcement banner, and
see how many people are connected to each server in real time.

---

## STEP 5 — Turn on real ads (AdMob)

Right now the app uses **Google's official test ad IDs** — safe to build
and test, but they don't earn real money.

1. Go to https://apps.admob.com, sign in, create an app (Android,
   `com.mbbs.vpnstudent`)
2. Create a **Banner** ad unit and an **Interstitial** ad unit — copy their IDs
3. In the project, replace these two spots with your real IDs:
   - `app/src/main/AndroidManifest.xml` → `APPLICATION_ID` meta-data value
   - `app/src/main/res/layout/activity_main.xml` → `app:adUnitId` on the AdView
   - `app/src/main/java/com/mbbs/vpnstudent/MainActivity.kt` → the interstitial
     ad unit ID string
4. Push the change — GitHub rebuilds automatically
5. AdMob pays out via bank transfer once you hit their payment threshold
   (usually $100+, varies by country) — payment setup is inside apps.admob.com

---

## What's already built into the app

- **Real WireGuard VPN tunnel** (official Google/WireGuard Android library —
  this is genuine encrypted VPN traffic, not a fake/demo tunnel)
- Live server list pulled from Firestore — add/remove/edit servers with zero
  app updates
- Banner + interstitial ads (AdMob), frequency controlled remotely from the
  admin panel
- "Premium" server flag (ready for you to gate behind a rewarded ad or paid
  unlock later)
- Study Tools screen for MBBS students: 25-minute focus timer, medical unit
  converter (kg/lb, mg/g, °C/°F), Russia emergency numbers (112/103/102/101)
  with one-tap dial, offline quick notes
- Maintenance-mode & announcement banner, controlled remotely
- Clean gradient/card-based Material 3 UI

## Natural next features (tell me if you want these built too)
- Rewarded video ads ("watch an ad to unlock premium server for 1 hour")
- Per-user WireGuard key provisioning (real multi-tenant security, needs a
  small Cloud Function)
- Login system so you can track individual users
- Dark mode toggle
- In-app "report a broken server" button that alerts you
