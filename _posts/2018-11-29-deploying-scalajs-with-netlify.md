---
published: false
layout: default
title: Deploying Scala.js to the Web with Netlify
---

Scala has long been my language of choice, but while I was aware of Scala.js, the process to get from Scala code to something that people could access in a web browser was a bit intimidating. Last week I set up a Scala.js project to automatically deploy the `master` branch from github to a website running on Netlify, and was pleasantly surprised at how easy and straightforward it all was.

## 0. Scala code in a Scala.js project using the [Scalor maven plugin](https://github.com/random-maven/scalor-maven-plugin)

I'll start by assuming we've got a Maven project set up that will link JS with the Scalor plugin:

````
C:\Users\md401\eclipse-workspace\plus-minus-zero>mvn install
...
[INFO] --- scalor-maven-plugin_2.12:1.4.2.20180508161326:compile-main (default) @ plus-minus-zero ---
[INFO] [compile-main] Invoking Zinc compiler: Scalor [47422f7f42a63a8f70345c9cf979188c]: 2.12.6
[INFO] [compile-main] [DBUG] No changes
[INFO]
[INFO] --- scalor-maven-plugin_2.12:1.4.2.20180508161326:scala-js-link-main (default) @ plus-minus-zero ---
[INFO] [scala-js-link-main] Full linker build request.
[INFO] [scala-js-link-main] Linker runtime: C:\Users\md401\eclipse-workspace\plus-minus-zero\target\classes\META-INF\resources\webjars\plus-minus-zero\0.2-SNAPSHOT\runtime.js
[INFO] [scala-js-link-main] Linker runtime: C:\Users\md401\eclipse-workspace\plus-minus-zero\target\classes\META-INF\resources\webjars\plus-minus-zero\0.2-SNAPSHOT\runtime.min.js
[INFO] [scala-js-link-main] Closure: 0 error(s), 0 warning(s)
...
````

(I'm sure it's possible to do this stuff with Gradle or SBT, but I find Maven much simpler and easier to use)

## 1. Create a basic HTML file, using resource filtering to point at our compiled JS

I was a little concerned that the path that the javascript is built in will change with the project version, but Maven has builting functionality to substitute in resource files. So I put some substitutable parameters in `src/main/resources/index.html`:

````
<!DOCTYPE html>
<html>
  <body>
    <div id="app"></div>
    <!-- Include Scala.js compiled code -->
    <script type="text/javascript" src="./META-INF/resources/webjars/${project.artifactId}/${project.version}/runtime.min.js"></script>
  </body>
</html>
````

And enabled filtering for my resources in `pom.xml`:

````
<resource>
  <directory>src/main/resources</directory>
  <filtering>true</filtering>
</resource>
````

In the same file I also made sure my application's `main` method was in the initializer list in the scalor plugin configration:

````
<linkerMainInitializerList>
  <initializer>com.github.m50d.plusminuszero.PlusMinusZero.main(build=${project.artifactId},stamp=${maven.build.timestamp})</initializer>
</linkerMainInitializerList>
````

To check everything's working, run `mvn process-classes` and then navigate to `target/classes/index.html` in your web browser. You should see your app running locally!

