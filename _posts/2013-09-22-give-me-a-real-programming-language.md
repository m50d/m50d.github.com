---
published: false
title: Give Me a Real Programming Language
---

The modern programming language is a complex and beautiful thing. Decades of research have gone into making it, and great accolades await anyone who can make even the slightest of improvements. The idea of "visual programming", expressing programs not as simple text files but some kind of rich, graphical, colourful view (usually based on some kind of "flowchart"), has been a perennial favourite, fruit for numerous PhD theses. Of course, each year no more than a handful of new languages gain any kind of traction, and not one visual programming language has ever been widely adopted.

So why do so many companies seem to think they can do better? All too often a system requires a turing-complete configuration, but we users are made to express it in some custom, tool-specific language - or worse, by dragging and dropping in some GUI. The worst case I've seen is the ESB that IBM tried to sell to my previous company - a software company, no less. The idea of a tool that would let non-programmers program is appealing, for obvious reasons - but it proves too much; if there were such a tool, companies would be using it already for all their programming, not just for this particular tool. The recent [case against cucumber](http://blog.8thlight.com/kevin-liddle/2013/09/18/a-case-against-cucumber.html) is a good example of the general problem with this kind of tool.

Even tools by and for real programs aren't immune from these problems. Consider ant, which used an XML syntax which rapidly expanded into a full-blown programming language, complete with <if> and <for> tags - but not the kind of language a sane developer would ever use.


XSLT
sendmail M4