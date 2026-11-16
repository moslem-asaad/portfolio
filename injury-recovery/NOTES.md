# Injury Recovery browser demo

This is a browser preview of the Injury Recovery app, a 2024 university team project for physiotherapy clinics. It uses the original Hebrew app screens, layout, logo, and treatment artwork. The frame around the app lets you switch roles.

- Select **Try the demo** to open the sample patient profile. Open a treatment, switch between the two exercise videos, mark exercises complete, and send a text feedback request.
- Select **System manager** to view feedback, reply to a pending request, or create a treatment with videos from the sample library. Switch back to **Patient view** to see the plan or reply.
- All records are fictional. Changes stay in the open tab and reset on reload. The demo makes no Firebase calls. Account forms and uploads are disabled. It uses two bundled illustrated exercise videos. The browser may fetch a static font for Hebrew text.

The original app uses Firebase Authentication for email and password sign-in, Cloud Firestore for profiles, treatments, exercise records and feedback, and Cloud Storage for videos. Firebase Analytics is listed as a dependency. The original upload code expects a device file path and a mobile video player. The unused mobile-only thumbnail package stays outside the demo flow.

## Rebuild and test

The app source is on the local `portfolio-web-demo` branch. Build with Flutter 3.22.3.

```sh
cd ~/devs/Injury-Recovery
export PATH="$HOME/devs/flutter-sdks/3.22.3/flutter/bin:$PATH"
flutter pub get
flutter build web --release --target lib/portfolio_demo.dart --base-href /portfolio/injury-recovery/ --web-renderer canvaskit --dart-define=FLUTTER_WEB_CANVASKIT_URL=canvaskit/ --pwa-strategy=none
rsync -a build/web/ ~/devs/portfolio/injury-recovery/

mkdir -p /tmp/injury-portfolio-preview
ln -sfn "$HOME/devs/portfolio" /tmp/injury-portfolio-preview/portfolio
python3 -m http.server 8765 --bind 127.0.0.1 --directory /tmp/injury-portfolio-preview
```

Open [the local demo](http://127.0.0.1:8765/portfolio/injury-recovery/). Pages after the `#` in the address work on static hosting and can be reloaded. The `screenshots/` folder contains six images of the main screens.

## Publish after testing

GitHub Pages publishes the portfolio from the root of `main`. After approving the preview, run these commands in the portfolio repository.

```sh
cd ~/devs/portfolio
git switch injury-recovery-demo
git add -- injury-recovery/
git commit -m "Add Injury Recovery web demo"
git switch main
git merge --ff-only injury-recovery-demo
git push origin main
```

Open [the published demo](https://moslem-asaad.github.io/portfolio/injury-recovery/) when GitHub Pages finishes.

## Firebase console checks for the original app

Publishing this demo requires no Firebase changes. The original project’s Authentication configuration responded. The public Firestore check received a document-missing response for a made-up document ID in `Treatments`. Review that read rule before connecting any public app. Cloud Storage returned HTTP 402 and reported the Spark billing plan.

1. Open [Firebase Console](https://console.firebase.google.com/). Select **Injury Recovery**. In **Project settings → General**, confirm the project ID is `injury-recovery` and check the web app configuration under **Your apps**.
2. In **Authentication → Sign-in method**, verify **Email/Password**. In **Authentication → Users**, verify that a private test account has a matching profile at `Users/{uid}` in Firestore. Test sign-in with that account privately. In **Authentication → Settings → Authorized domains**, add `moslem-asaad.github.io` only if you later connect a Firebase sign-in flow there.
3. In **Firestore Database → Rules**, review the rules for the default database. Remove broad public access. Check that patients can only read their own profile, treatments and feedback. Keep manager access tied to a role set by an administrator. Prevent users from changing their own manager role. Use the Rules Playground or emulator to test signed-out, patient and manager access before publishing rule changes. Some original app queries may need changes for these rules. Realtime Database is unused.
4. In **Storage → Files**, check for `injury-recovery.appspot.com`. Firebase requires Blaze to access the original bucket. In **Storage → Rules**, restrict videos to the intended clinic users and block public uploads. If private uploads are needed later, restrict them to the uploader’s folder, accept video only and set a file limit, for example 20 MB. Use the Rules Playground or emulator to test access. A file size limit alone still allows repeated uploads.

## Rules for a future Firebase demo

If you later need a Firebase-backed demo, create a separate Firebase project. Add only fictional records. If using anonymous Firebase sign-in, turn on **Authentication → Sign-in method → Anonymous** and use a new local session for each visitor. Place fictional Firestore documents under a top-level `Demo` collection. Apply these rules under **Firestore Database → Rules** in the separate project, then test and publish them there.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /Demo/{document} {
      allow read: if request.auth != null
                  && request.auth.token.firebase.sign_in_provider == 'anonymous';
      allow write: if false;
    }
  }
}
```

Bundle the video with the site. If a Storage bucket exists in the separate project, apply this under **Storage → Rules** to block all client access there.

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{file=**} {
      allow read, write: if false;
    }
  }
}
```

Firebase reads can still use project quota. These example rules are for the separate demo project. The original clinic app needs rules matched to its collections and queries.

## App files changed

- `lib/features/presentation/services/service_layer.dart`
- `lib/features/presentation/views/customer/customer_profile.dart`
- `lib/features/presentation/views/customer/treatment/feedback_request.dart`
- `lib/features/presentation/views/customer/treatment/ro_feedback_request.dart`
- `lib/features/presentation/views/customer/treatment/treatment_view.dart`
- `lib/features/presentation/views/customer/treatment/videos_view.dart`
- `lib/features/presentation/views/customer/treatments.dart`
- `lib/features/presentation/views/login_view.dart`
- `lib/features/presentation/views/system_manager/create_treatment.dart`
- `lib/features/presentation/views/system_manager/exercise_videos.dart`
- `lib/features/presentation/views/system_manager/main_view.dart`
- `lib/features/presentation/views/system_manager/users_feedbacks.dart`
- `lib/features/presentation/widgets/my_video_player.dart`
- `macos/Flutter/GeneratedPluginRegistrant.swift`
- `pubspec.lock`
- `pubspec.yaml`
- `fonts/Roboto-Bold.ttf`
- `fonts/Roboto-Regular.ttf`
- `fonts/Roboto_LICENSE.txt`
- `images/demo-exercise-hold.mp4`
- `images/demo-exercise.mp4`
- `lib/features/presentation/services/demo_service.dart`
- `lib/portfolio_demo.dart`
