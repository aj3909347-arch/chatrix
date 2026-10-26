# chatrix
main.dart
name: Build APK
on: workflow_dispatch
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.5'
          channel: stable
      - name: Build
        run: |
          flutter create --org com.chatrix --project-name app --platforms android build_app
          cp main.dart build_app/lib/main.dart
          cp google-services.json build_app/android/app/google-services.json
          cd build_app
          flutter pub add firebase_core:^3.6.0 firebase_auth:^5.3.1 cloud_firestore:^5.4.4
          sed -i 's/id "org.jetbrains.kotlin.android" version \(.*\) apply false/&\n    id "com.google.gms.google-services" version "4.3.15" apply false/' android/settings.gradle
          sed -i 's/id "kotlin-android"/&\n    id "com.google.gms.google-services"/' android/app/build.gradle
          sed -i 's/minSdkVersion flutter.minSdkVersion/minSdkVersion 23/' android/app/build.gradle
          flutter build apk --release
      - uses: actions/upload-artifact@v4
        with:
          name: chatrix-apk
          path: build_app/build/app/outputs/flutter-apk/app-release.apk
