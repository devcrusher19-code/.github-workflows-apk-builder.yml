# .github-workflows-apk-builder.yml
.github/workflows/apk-builder.yml
devcrusher19-code (devcrusher19-code)
devcrusher19-code

name: Build Epil-vip debug APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Locate project ZIP (if present) and unzip
        run: |
          if ls *.zip 1> /dev/null 2>&1; then
            echo "Found zip files, extracting..."
            if [ -f "epil-vip-android-project-updated.zip" ]; then
              unzip -q epil-vip-android-project-updated.zip -d project_root
            else
              unzip -q $(ls *.zip | head -n 1) -d project_root
            fi
            PROJECT_DIR=$(find project_root -maxdepth 1 -type d ! -path project_root | head -n 1)
            if [ -z "$PROJECT_DIR" ]; then
              PROJECT_DIR="project_root"
            fi
            echo "EXTRACTED_DIR=$PROJECT_DIR" >> $GITHUB_ENV
          else
            echo "No zip detected. Building directly from repository root."
            echo "EXTRACTED_DIR=." >> $GITHUB_ENV
          fi

      - name: Setup Java 11
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: 11

      - name: Install dependencies
        run: |
          sudo apt-get update -y
          sudo apt-get install -y unzip zip gradle

      - name: Build debug APK
        working-directory: ${{ env.EXTRACTED_DIR }}
        run: |
          ls -la
          if [ -f "./gradlew" ]; then
            chmod +x ./gradlew
            ./gradlew assembleDebug --no-daemon
          else
            gradle assembleDebug --no-daemon
          fi

      - name: Upload APK artifact
        uses: actions/upload-artifact@v4
        with:
          name: epil-vip-apk
          path: |
            ${{ env.EXTRACTED_DIR }}/app/build/outputs/apk/debug/app-debug.apk
