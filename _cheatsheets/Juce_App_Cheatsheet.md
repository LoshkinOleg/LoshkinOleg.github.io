---
title: "JUCE Applications Building Cheatsheet"
layout: cheatsheet
categories: "cheatsheets"
permalink: /:categories/:title
---

# Headers to add to "Header Search Paths"

Headers required to compile an empty JUCE audio app. You need to add those manually to the "Linux Makefile" Exporter when using Projucer on Linux.
Obviously install the packages that all of those headers come from.

```
/usr/include/freetype2
/usr/include/gtk-3.0
/usr/lib/x86_64-linux-gnu/glib-2.0/include
/usr/include/glib-2.0
/usr/include/pango-1.0
/usr/include/harfbuzz
/usr/include/cairo
/usr/include/gdk-pixbuf-2.0
/usr/include/atk-1.0
/usr/include/webkitgtk-4.1
/usr/include/libsoup-3.0
```
