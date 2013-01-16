---
layout: default
published: false
title: Generic Contexts, or: Don't Mention the M Word
---

I took a [Category Theory course](http://gopher.srcf.net:70/users/md401/pt3notes/category.pdf) at university, but in truth I never understood more than the basics. [Lampson's (or Wheeler's) Law](http://c2.com/cgi/wiki?ButlerLampson) applies just as strongly in mathematics, and I hit my personal indirection limit about a third of the way through.

Fortunately, in software we know how to avoid this: Agile. So let's talk through (an anonymized version of) a real-life problem I had at work, and the abstraction that can be used to solve it - no theory, just programming.

Suppose we need to do something inside a "context". 