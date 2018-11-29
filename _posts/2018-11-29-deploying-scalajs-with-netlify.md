---
published: false
layout: default
title: Deploying Scala.js with Netlify
---

Last week I set up a Scala.js project to automatically deploy the `master` branch from github onto Netlify, and was pleasantly surprised at how easy and straightforward it all was.

## 0. Working Scala.js project using the [Scalor maven plugin](https://github.com/random-maven/scalor-maven-plugin)

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
