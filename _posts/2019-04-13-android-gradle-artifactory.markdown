---
layout: post
title:  "Android + Gradle + Artifactory"
date:   2019-04-13 22:58:40 +0200
categories: android gradle artifactory update
---

This page shows how to integrate gradle builds of android libraries with a private maven repository hosted on artifactory.

You'll need:

1. The library identifier:  
    `<your-group-id>:<your-artifact-id>:<your-version-number>`  
    example: `com.example.test:mylibrary:1.0.0`
2. An Artifactory server with a repository:  
    `https://<your-server>/<your-base-path>/<your-repository>`  
    example: `https://example.com/artifactory/gradle-release-local`
3. A user account with sufficient access rights consisting of `<your-username>` and `<your-password>`.

## Producer and Consumer

Both, producer and consumer must create a properties file containing the artifactory username and password.
The file should be created as `~/.gradle/gradle.properties`.
The content should be:

```ini
artifactory_username=<your-username>
artifactory_password=<your-password>
```

## Producer side

Project level `build.gradle` file:

```groovy
buildscript {
    dependencies {
        classpath "org.jfrog.buildinfo:build-info-extractor-gradle:4.7.1"
    }
}
```

Module level `build.gradle` file:

```groovy
apply plugin: 'com.jfrog.artifactory'
apply plugin: 'maven-publish'
apply plugin: 'com.android.library'

// define name and version
def libraryGroupId = '<your-group-id>'
def libraryArtifactId = '<your-artifact-id>'
def libraryVersion = '<your-version-number>'

// add the android and dependencies sections here...

publishing {
    publications {
        aar(MavenPublication) {
            groupId libraryGroupId
            version id.libraryVersion
            artifactId libraryArtifactId

            artifact("$buildDir/outputs/aar/${artifactId}-release.aar")

            // Build a pom file which will be published, so the consumer does not
            // need to declare all dependencies manually.
            pom.withXml {
                final dependenciesNode = asNode().appendNode('dependencies')

                ext.addDependency = { Dependency dep, String scope ->
                    if (dep.group == null || dep.version == null || dep.name == null || dep.name == "unspecified")
                        return // ignore invalid dependencies

                    final dependencyNode = dependenciesNode.appendNode('dependency')
                    dependencyNode.appendNode('groupId', dep.group)
                    dependencyNode.appendNode('artifactId', dep.name)
                    dependencyNode.appendNode('version', dep.version)
                    dependencyNode.appendNode('scope', scope)

                    if (!dep.transitive) {
                        // If this dependency is transitive, we should force exclude
                        // all of its dependencies from the POM
                        final exclusionNode = dependencyNode.appendNode('exclusions').appendNode('exclusion')
                        exclusionNode.appendNode('groupId', '*')
                        exclusionNode.appendNode('artifactId', '*')
                    } else if (!dep.properties.excludeRules.empty) {
                        // Otherwise add specified exclude rules
                        final exclusionNode = dependencyNode.appendNode('exclusions').appendNode('exclusion')
                        dep.properties.excludeRules.each { ExcludeRule rule ->
                            exclusionNode.appendNode('groupId', rule.group ?: '*')
                            exclusionNode.appendNode('artifactId', rule.module ?: '*')
                        }
                    }
                }
                // Map all gradle dependencies to maven dependencies (the compile dependency of gradle is only
                // present to support legacy gradle build files)
                configurations.compile       .getAllDependencies().each { dep -> addDependency(dep, "compile") }
                configurations.api           .getAllDependencies().each { dep -> addDependency(dep, "compile") }
                configurations.implementation.getAllDependencies().each { dep -> addDependency(dep, "runtime") }
            }
        }
    }
}

artifactory {
    contextUrl = 'https://<your-server>/<your-base-path>'
    publish {
        repository {
            repoKey = '<your-repository>'

            username = artifactory_username
            password = artifactory_password
        }
        defaults {
            publications('aar')
            publishArtifacts = true

            properties = [
                'qa.level': 'basic',
                'q.os'    : 'android',
                'dev.team': 'core']
            publishPom = true
        }
    }
}
```

Deploying the artifact is only a matter of invoking `gradlew assembleRelease artifactoryPublish`

## Consumer side

Add the repository to the `repositories` section:

```groovy
repositories {
    google()
    jcenter()
    maven {
        url "https://<your-server>:443/<your-base-path>/<your-repository>"
        credentials {
            username = "${artifactory_username}"
            password = "${artifactory_password}"
        }
    }
}
```

Add the library with its given maven coordinates to the `dependencies` section:

```groovy
dependencies {
    implementation '<your-group-id>:<your-artifact-id>:<your-version-number>'
}
```
