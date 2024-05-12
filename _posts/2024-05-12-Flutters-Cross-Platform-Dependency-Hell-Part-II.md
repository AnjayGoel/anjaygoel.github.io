---
layout: post
title: "Flutter's Cross Platform Dependency Hell - Part II"
date: 2024-05-12 00:00:00 +0530
category: Blog
tags: [ flutter ]
desc: "---"
description: "---"
---

In the last blog post, I wrote about how flutter includes all the platform-specific implementations in the dependency
tree even when you don't target those platforms. Thus causes unnecessary dependency conflicts. In this post, I will show
another annoying issue about how flutter handles native dependencies, particular in iOS.

If you add the following packages in your project and do a `flutter pub get`:

```
flutter_facebook_app_links: ^3.0.2
flutter_facebook_auth: ^6.2.0
```

It will resolve successfully and even run perfectly on android. But when you try to build for iOS you will get the
following error:

```
[!] CocoaPods could not find compatible versions for pod "FBSDKCoreKit":
  In Podfile:
    flutter_facebook_app_links (from `.symlinks/plugins/flutter_facebook_app_links/ios`) was resolved to 3.0.0, which depends on
      FBSDKCoreKit (~> 16.0)

    flutter_facebook_auth (from `.symlinks/plugins/flutter_facebook_auth/ios`) was resolved to 6.2.0, which depends on
      FBSDKLoginKit (~> 17.0.0) was resolved to 17.0.0, which depends on
        FBSDKCoreKit (= 17.0.0)
```

What's happening here is that both the packages depend on different major versions of native `FBSDKCoreKit` cocoapod (
The native facebook iOS SDK). Since flutter is only resolving the dart dependencies, it didn't even know about the
dependency conflict in the native packages.

Such issues can break your builds even without you changing anything. Lets say you use two packages in your app:

```
package_a: ^1.0.0
package_b: ^1.0.0 
```

Both of them depending on a cocoapod (native ios library) say `native_lib: 10.0.0`. Your app will build fine because
both flutter & cocoapod resolve perfectly.

Now `package_a` updates the major version of `native_lib` to `11.0.0` in a minor release `1.1.0`. Flutter would
resolve `package_a` to the newer version `1.1.0` causing a conflict with `package_b` which it's still
on `native_lib: 10.0.0`. The only solution would be to manually pin `package_a` to `1.0.0` till `package_b` updates
their native dependencies to the latest version.
