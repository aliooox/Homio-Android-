# Homio Android + Google Sheets setup

## What is included
- Android Studio project (Java + WebView UI)
- Your original Homio wordmark remains in the app header; the supplied rounded-square icon is cropped and configured separately as the Android launcher icon
- Responsive Homio dashboard, bookings, teams, expenses and payments screens
- Google Apps Script backend in `scripts/Code.gs`
- GitHub Actions workflow to build a debug APK
- The APK is **not prebuilt** in this ZIP because this environment does not have the Android SDK/Gradle build toolchain installed.

## 1) Prepare your Google Sheet
Open your spreadsheet:
https://docs.google.com/spreadsheets/d/1Z2d8ZGfAgID2IPcO1cidwSlBN6rGmcochSp2_A7r1po/edit

1. In the spreadsheet, open **Extensions → Apps Script**.
2. Replace the default script with the contents of `scripts/Code.gs`.
3. Save the script.
4. Select `setupHomioSheets` in the function dropdown and click **Run**. Approve the requested spreadsheet permissions.
5. Confirm the `Bookings`, `Teams`, `Expenses`, and `Payments` tabs exist. Existing tabs with these names are preserved; only blank tabs receive headers.

## 2) Deploy the backend
1. In Apps Script, select **Deploy → New deployment → Web app**.
2. Choose the execution identity and access restriction deliberately. Restrict access to your own Google account or your trusted Google Workspace users wherever possible.
3. Do **not** use unrestricted public access for a write-enabled business endpoint. The simple included backend is a starter integration and does not itself verify each caller's Google identity or role. For multi-user production, add authenticated identity/role checks or place it behind a proper authenticated API.
4. Deploy and copy the URL ending in `/exec`.

Important: Google Apps Script web-app access choices can vary by account type. If you cannot restrict access to the intended users while still allowing the app to call it, do not loosen access to public; use an authenticated backend/OAuth integration instead.

## 3) Build and connect the Android app
1. Open this folder in Android Studio.
2. Let Gradle sync. Install Android SDK Platform 35 if prompted.
3. Run the app on your Android device/emulator.
4. Open **Settings**, paste the Apps Script `/exec` URL and tap **Save connection**.
5. Tap **Test connection & sync**.
6. Confirm the dashboard loads the rows from your sheet.

## 4) Build a debug APK
In Android Studio: **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
The output is normally under `app/build/outputs/apk/debug/app-debug.apk`.

Or push this project to a GitHub repository. The included workflow `.github/workflows/android.yml` builds a debug APK on each push and makes it available as a workflow artifact.

## Sheet schema
- `Bookings`: bookingId, customerName, phone, address, date, time, service, price, team, status, paid, notes, createdAt
- `Teams`: team, lead, members, status
- `Expenses`: expenseId, amount, category, team, date, method, bookingId, notes, createdAt
- `Payments`: paymentId, bookingId, customerName, amount, method, date, reference, createdAt

## Important limitations before production
- This is an MVP starter, not a security-audited financial application.
- The Apps Script starter endpoint needs proper authentication/authorization for every caller before multi-user deployment.
- Payment records are ledger entries only; no money is moved by this app.
- Team availability is a basic estimate; add a conflict-aware calendar before relying on it for dispatch.
- Back up the spreadsheet and test with non-critical sample records first.
