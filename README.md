name: Build APK
on: [push, workflow_dispatch]
permissions:
  contents: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: unzip -o smart_health.zip
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: '17' }
      - uses: subosito/flutter-action@v2
        with: { flutter-version: '3.27.4', channel: stable }
      - working-directory: smart_health
        run: |
          flutter create --platforms=android --org pk.smarthealth --project-name smart_health .
          python3 tool/patch_android.py
          flutter pub get
          dart run flutter_launcher_icons
          dart run flutter_native_splash:create
          flutter build apk --release
          cp build/app/outputs/flutter-apk/app-release.apk ../SmartHealth.apk
      - uses: softprops/action-gh-release@v2
        with:
          tag_name: v1.0.${{ github.run_number }}
          name: Smart Health v1.0.${{ github.run_number }}
          files: SmartHealth.apk
