J2CL Maven plugin
=================

[J2CL](https://github.com/google/j2cl) is a powerful Java-JavaScript transpiler used in Gmail,
Google Docs, and even a [Quake II Port](https://github.com/treblereel/quake2-gwt-port).
Unfortunately, the official J2CL is tightly coupled with the Bazel build system. It doesn't
run on Windows. And it lacks features like Source Maps for debugging.

This project provides J2CL as a Maven plugin. It supports hot-reloading,
debugging of Java code in the browser, and fast builds by way of efficient caching.
The latest Java version supported is the same as J2CL, Java 9.

## Installation

Run this command to generate a starter project:

```bash
tbd maven command...
```

----

## Goals

The plugin has four goals executable with `mvn j2cl:<insert goal>`:

1. `build`: executes a single compilation, typically to produce a JS application or library.

2. `test`: compiles and executes j2cl-annotated tests, once.

3. `watch`: monitor source directories, and when changes happen that affect any `build` or `test`, recompile the 
required parts of the project. Tests are not (presently) run, but can be run manually by loading the html pages
that exist for them. Make sure to have run the `build` goal at least once before `watch`. Source Map debugging
is enabled with the `enableSourcemaps` property (`mvn j2cl:watch -DenableSourcemaps=true`). Multi-Module projects
should run `watch` on the "parent" in the reactor, so that any changed modules are detected and recompiled correctly,
rather than depending on stale sources.
  
<!--   * `watch-test`: only rebuild things that affect `test` executions, useful when iterating on tests and avoiding
    building the application itself
   * `watch-build`: only rebuild things that affect `build` executions, may save time if tests aren't currently
    being re-run               -->
  
4. `clean`: cleans up all the plugin-specific directories

Optional goal arguments can be viewed with:
```bash
mvn help:describe -DgroupId=com.vertispan.j2cl -DartifactId=j2cl-maven-plugin \
-Dversion=0.16-SNAPSHOT -Ddetail
```

----

## Persistent, shared caching

To help make builds faster on your own machine, you can move the cache directory out of `target/`, and into
somewhere global like `~/.m2/` so that it doesn't get deleted every time you need to clean your maven project.
Similarly, this will result in all GWT 3 projects built on your machine sharing the same cache, so that as long
as the same version of the compiler is run on the same sources, output can be reused rather than building it
again.

To do this, in your `~/.m2/settings.xml` file, you can add a profile like this:
```xml
      <profile>
        <id>shared-gwt-cache</id>
        <activation>
          <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
          <gwt3.cache.dir>/home/myusername/.m2/gwt3BuildCache</gwt3.cache.dir>
        </properties>
      </profile>
```

Since compiled output is stored based on the hash of the inputs, this should be safe, but from time to time
you may find the need to remove some cache entries. There are a few tools for this:
 
 * `mvn j2cl:clean` - this will find all the artifacts in the current reactor project and remove any cache entries
 found in the specified directory. 
 * `mvn j2cl:clean -Dartifact=some-artifact-id` - deletes any artifact that was built with this plugin from the
 cache which has an artifactId matching the given parameter. Currently does not support a groupId in the name
 of making it easier to quickly specify a cache entry, and to make the contents of the cache directory slightly
 easier to traverse manually.
 * `mvn j2cl:clean -Dartifact=*` - deletes all contents in the cache directory. If you find yourself doing this 
 a lot, file a bug describing whatever is going wrong frequently, and consider leaving the cache directory in the
 target directory where it defaults to, so that it can be cleaned automatically.
 
----

## Further Reading
* [JS Interop By Example](https://github.com/google/j2cl/blob/master/docs/jsinterop-by-example.md#j2cl-jsinterop-by-example)
* [J2CL Compatibility Limitations](https://github.com/google/j2cl/blob/master/docs/limitations.md#limitations-of-j2cl---java-compatibility)
* [Elemental2 Browser APIs Github](https://github.com/google/elemental2) and [Javadoc (for DOM module)](https://javadoc.io/doc/com.google.elemental2/elemental2-dom/latest/index.html)

----

## Credits

This plugin includes the code original developed as

        com.vertispan.j2cl:build-tools

built from here:

    https://github.com/gitgabrio/j2cl-devmode-strawman

------------------------
