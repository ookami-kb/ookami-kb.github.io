---
layout: post
title:  "Adding flavor-specific tasks to Gradle"
date:   2016-01-13 11:00:00 +0200
categories: android kotlin view
---

Suppose you have a multi-flavored project. Each flavor is a separate app, so you have to use separate `google-services.json` file for each flavor.

The problem is that `google-services.json` file must be placed in the `app/` directory (_update:_ as of version `2.0.0-alpha3` of the plugin support was added for build types), so we have to copy this file to `app/` directory on every flavor build process.

If we want to automate this process, we can add flavor-specific tasks to our `build.gradle` file and make them run depending on what flavor we are building.

Here is my way of achieving this.

First of all, add this code to `productFlavors` section:

```groovy
all { flavor ->
    task("${flavor.name}GoogleServices", type: Copy) {
      description = 'Switches to google-services.json depending on flavor'
      from "src/${flavor.name}"
      include "google-services.json"
      into "."
    }
}
```

This will create tasks `flavor1GoogleServices`, `flavor2GoogleServices` and so on. This task will copy `google-services.json` file from `src/flavor1/` folder to `app/` folder.

Now add this code to the end of the build.gradle file:

```groovy
afterEvaluate {
    android.productFlavors.all {
      flavor -> 
        tasks."generate${flavor.name.capitalize()}DebugResources".dependsOn "${flavor.name}GoogleServices"
        tasks."generate${flavor.name.capitalize()}ReleaseResources".dependsOn "${flavor.name}GoogleServices"
    }
}
```

This code runs after a project is evaluated. It adds dependencies to tasks `generateFlavor1DebugResources`, `generateFlavor2DebugResources` etc. on our tasks.

That’s all. After switching build variant in Android Studio the right task for this variant will run and update json file.
