---
layout: post
title:  "Generating Android asset files with Gradle"
date:   2021-07-03 17:00:00 +0200
categories: Development
tags: Android Gradle Kotlin
---

Recently, I've come across the problem that I need to include a generated file in the asset files of my Android application. More specifically, the report of a build-time dependency license validation (using the [Licensee][licensee] Gradle plugin) should be included in the application. Obviously, this should be possible with a flexible build system like Gradle. The solution I came up with has three steps.

1. Add a new asset directory where we can place the generated asset files.
2. Add a task to Gradle that generates the asset files.
3. Add dependencies to the Task to ensure that it is called automatically.

First we register the output directory of our asset generation task as a new asset directory. This ensures, that its contents are picked up by the `mergeDebugAssets` and `mergeReleaseAssets` tasks and included in the output apk file.

```kotlin
sourceSets["main"].assets.srcDir(
    layout.buildDirectory.dir("generated/dependencyAssets/"))
```

Next, we need to tell Gradle, how the asset files should be generated. This is done for each build variant independently, because the asset files we generate might differ between variants:

```kotlin
applicationVariants.configureEach {
    val variant = this

    // New tasks are configured here.
}
```

For each variant we create a new task to generate the asset files. In my case this generation task simply copies the output of a Gradle plugin that walks the dependency tree and collects all license files.

```kotlin
val copyArtifactsTask =
    tasks.register<Copy>("copy${variant.name.capitalize()}ArtifactList") {
        from(
            project.extensions.getByType(ReportingExtension::class.java)
                .file("licensee/${variant.name}/artifacts.json")
        )
        into(layout.buildDirectory.dir("generated/dependencyAssets/"))
    }
```

For cases where you have more complex asset generation code you can create a task that executes arbitrary Kotlin code, for example as follows:

```kotlin
val customTask = tasks.register("generate${variant.name.capitalize()}AssetFiles") {
    doLast {
        // Write your code here.
    }
}
```

Finally, we need to ensure, that the task of ours is actually called before the assets are merged. This can be done with the `dependsOn` function of the task class. We want to execute our task prior to the `merge{variant}Assets` tasks. This can be done with the following line:

```kotlin
tasks["merge${variant.name.capitalize()}Assets"].dependsOn(copyArtifactsTask)
```

If your asset generation depends on some other output you need to declare this dependency, too. In this example, the dependency report needs to be generated prior to copying it, so we need to add a dependency to our task as follows:

```kotlin
copyArtifactsTask.dependsOn("licensee${variant.name.capitalize()}")
```

The full example follows:

```kotlin
android {
    
    // ... Your other build stuff

    // Register our custom output directory as an asset folder that will be
    // used for asset merging.
    sourceSets["main"].assets.srcDir(
        layout.buildDirectory.dir("generated/dependencyAssets/"))

    // Individual tasks for release and debug builds
    applicationVariants.configureEach {
        val variant = this

        // Define a new task to generate the asset files. Here the file is only
        // copied, but you could do anything else instead.
        val copyArtifactsTask =
            tasks.register<Copy>("copy${variant.name.capitalize()}ArtifactList") {
                from(
                    project.extensions.getByType(ReportingExtension::class.java)
                        .file("licensee/${variant.name}/artifacts.json")
                )
                into(layout.buildDirectory.dir("generated/dependencyAssets/"))
            }

        // This dependency is only necessary if the asset generation depends on
        // something else, the output of the licensee plugin in my case.
        copyArtifactsTask.dependsOn("licensee${variant.name.capitalize()}")

        // Add a dependency between the asset merging and our generation task.
        // This is necessary to ensure that the assets are generated prior to
        // the merging step.
        tasks["merge${variant.name.capitalize()}Assets"].dependsOn(copyArtifactsTask)
    }
}
```

## Useful links

- Licensee: <https://github.com/cashapp/licensee>

<!-- References -->
[licensee]: https://github.com/cashapp/licensee
