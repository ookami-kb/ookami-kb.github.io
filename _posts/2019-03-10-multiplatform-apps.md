---
layout: post
title:  "Multi-platform apps: are we there yet?"
date:   2019-03-10 11:00:00 +0200
categories: flutter
---

The weekend was rather rainy, so I was playing with Flutter. Last time I’ve tried it, it was still in beta (or even alpha?), so I was interested if it finally became useful for building multi-platform apps.

I’ve decided to re-make our MileFarClub app (it’s our internal side project for tracking steps and competing with colleagues). Here’s what I was able to achieve while investing into it several hours for 2 days:

- Initial screen with authorization by google account, requesting permissions from Google Fit on Android and Health Kit on iOS.
- Main screen with displaying user information and total/today steps count.
- Rank screen with the list of users and their total results.

All the data are read-only for now, but it’s not a problem to add cross-platform updating as well. Due to the fact that we’re using different sources of activity information (Google Fit vs Health Kit) it’s not possible to make it totally cross-platform, so this part needs to be implemented separately. Same with background notifications and updates, as the process differs a lot between these 2 platforms.

Overall impressions from Flutter is rather positive, I was able to share 100% of UI code and some high-level logic. Integrating with native code was rather smooth as well, so it’s not a problem to write some mini-plugin to use native solution for each platform. Even Dart has become more usable as a language 😃.

I definitely like it more than React Native, because e.g.:
- it really compiles into native code without using bridge to JavaScript, so performance is the same as of native app, especially regarding animations;
- it uses its own widget library, so it’s not a problem to create brand-oriented UI that looks the same on both platforms (at the same time it’s not a problem to use different look&feel on each platform due to the standard library including both Material and Cupertino widgets) — while React Native uses standard widgets of each platform.

Yes, the project is way too simple to say something for sure, so I will definitely invest more time in exploring it, but for now it looks like the most promising solution for the near future, when comparing to the other ones:

- React Native — I’ve already wrote about it.
- Xamarin — it’s really cool, but after MS abandons Windows Phone I’m slightly afraid of Xamarin being abandoned as well;
- Kotlin Multiplatform — looks very good (and it’s Kotlin ❤️), but it’s too raw and not ready for production yet, it needs at least several more years to become production ready.

Besides that there’re rumours about Google replacing Android with Fuchsia, and Flutter is the only native framework for it. Although I’m pretty sure that Google will provide some tool to convert Android app into Fuchsia app or there will be some VM to run Android apps (even Google isn’t crazy enough to kill OS with ~90% of market), but still adopting native solution in its early stage can be useful.

_P.S._ For a really long time I hope to find some solution to build high-quality cross platform apps and at the same time being rather sceptical about available options. I think I’ve tried almost all of them: Titanium, Phonegap, Ionic, React Native, Kotlin Multiplatform, Xamarin, Flutter… you name it. There always was some discomfort with them, so every time I was returning to native development. But I guess now I’ve found the reason of this discomfort: when they say, that with This-One-Universal-Framework-To-Rule-Them-All you don’t need to have experience with iOS/Android native development at all — no, it won’t work. It can be useful only when you team has experience with developing native apps. So this equation:

```
2 Flutter guys = 2 Android guys + 2 iOS guys
```

is totally wrong, but this one:

```
1 Flutter guy + 1 Android guy + 1 iOS guy = 2 Android guys + 2 iOS guys
```

looks more like a truth for me.

And you still have a benefit, because 3-person team will be as productive as 4-person one (well, yeah, it’s only my theory for now), but you need to have developers in you team who know native development for each platform.

And I believe that cross-platform will eventually win, mainly because (yes, I’m quoting an official Flutter FAQ, so you can argue about these points being slightly opinionated, but I agree with them):

- Modern app design trends point towards designers and users wanting more motion-rich UIs and brand-first designs. In order to achieve that level of customized, beautiful design, Flutter is designed to drive pixels instead of the OEM widgets.
- By using the same renderer, framework, and set of widgets, it’s easier to publish for both iOS and Android concurrently, without having to do careful and costly planning to align two separate codebases and feature sets.
- By using a single language, a single framework, and a single set of libraries for all of your UI (regardless if your UI is different for each mobile platform or largely consistent), we also aim to help lower app development and maintenance costs.

> Originally published at [developers.mews.com](https://developers.mews.com/multiplatform-apps-are-we-there-yet/)
