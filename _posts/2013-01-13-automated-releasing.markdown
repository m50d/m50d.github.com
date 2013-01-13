---
layout: default
published: false
title: Automated Releasing with Maven
---

One of the proofs that the software industry is still immature is how much effort is still duplicated. I've now worked the same job in three very different industries, and while there has been plenty of business-specific stuff, there are some things I've found myself writing again each time: a type-safe [Hibernate](http://www.hibernate.org) query language, a responsive AJAX layer for [Wicket](http://wicket.apache.org), and a release system for [Maven](http://maven.apache.org).

I've witnessed more hate for Maven than any other piece of software; in my view this couldn't be more wrong. Simply put, Maven is an *opinionated* build system; it forces your project to bend to meet its expectations, not the other way around. Many users respond by trying to fight Maven, and come out with either hundreds of lines of fragile XML that will break the first time they add another plugin, or give up and go back to ant. I'm not sure I've actually programmed in anything that qualifies as a "bondage and discipline language", but Maven seems to meet the spec.

It's wonderful. This program is on a mission to eliminate "snowflake" builds (like  [snowflake servers](http://server.dzone.com/articles/martin-fowler-snowflake) - I wish I could find the parody article claiming that devops was destroying years of craftmanship in hand-building servers). And within the Java world it's basically succeeded, meaning I or any vaguely competent Java developer can pick up a new project and *not have to think about* how it's built. Like with code formatting standards, the value is not that Maven's standards are particularly good - more important is that we have a standard and all our projects follow it.

With this in mind, the obviously correct way to create stable builds of a project is with Maven's "release" plugin. I feel this is something of a misnomer; it's 