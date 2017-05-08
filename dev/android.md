---
layout: page
title: Android
---

* Debugging Native SO: http://blog.dornea.nu/2015/07/01/debugging-android-native-shared-libraries/
* Libraries: http://www.codeproject.com/Articles/1004611/Open-source-Android-libraries-every-programmer-sho
* Android bytecode: https://mariokmk.github.io/programming/2015/03/06/learning-android-bytecode.html

# adb

If adb is not recognizing your device, see https://dl-ssl.google.com/android/repository/latest_usb_driver_windows.zip

The Nexus drivers may be available in android/sdk/extras/google/usb_driver.

# Android Studio

Copy `dev/AndroidStyle.xml` to `~/.AndroidStudioBeta/config/codestyles` and restarted Android Studio.
Select this style from Settings -> Code Style -> Java -> Scheme -> AndroidStyle.

# Intents

To test intents:

    adb shell am start -a android.intent.action.VIEW -n com.google.android.apps.maps/com.google.android.maps.MapsActivity -d "geo:6.88,79.91?q=6.88,79.91 (foo)"
