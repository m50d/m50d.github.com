---
layout: default
title: Second Logging System
published: false
---

I've often thought [SLF4J](http://www.slf4j.org) is a clear-cut case of [second-system effect](http://c2.com/cgi/wiki?SecondSystemEffect). It features strict separation of interface and implementation, complete with at least five different backends and a really elegant trick for determining which is actually in use of runtime... for logging. Something that's built into the JVM (though few seem to actually use java.util.logging these days, probably for good reason), or at least had a fully functional monolithic implementation in [log4j](http://logging.apache.org/log4j/1.2/); I suspect a large proportion of slf4j users are simply using the slf4j-log4j backend (my previous employer mandated the use of Logback for a while, but given that none of us knew how to configure it and its supposed advantages never materialized, eventually sense was seen), meaning all it gives us is an extra layer of indirection.