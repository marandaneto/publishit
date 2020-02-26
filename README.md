# publishit
Just a sample to explore the `maven-publish` plugin on Gradle 6.x and AGP 3.6.x

# module publishmykotlinlib
It's a JVM lib that runs Java and Kotlin code and it uses the built-in Javadocs/Sources tasks from Gradle 6.x to generate the javadocs, sources and pom file.

```groovy
java {
    withJavadocJar()
    withSourcesJar()
}
```

# module publishmylib
It's an Android lib that runs Java and Kotlin code, it uses the built-in `release` component from AGP 3.6.x.

```groovy
from components.release
```

# required improvements
Ideally, we'd like to have something similar and built-in like java does:

```groovy
android {
    withJavadocJar()
    withSourcesJar()
}
```

as you can see, generating Javadocs and Sources is still verbose and it requires some boilerplate code:

```groovy
// needed because: throwing: javadoc: error - Illegal package name
tasks.withType(Javadoc).all {
    enabled = false
}

task androidJavadocs(type: Javadoc) {
    source = android.sourceSets.main.java.source
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
    classifier = 'javadoc'
    from androidJavadocs.destinationDir
}

task androidSourcesJar(type: Jar) {
    classifier = 'sources'
    from android.sourceSets.main.java.source
}

afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                artifact androidSourcesJar
                artifact androidJavadocsJar

                from components.release
            }
        }
    }
}
```
