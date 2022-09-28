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

# Adding visual elements to your application.

- When you make a member component inheriting from juce::Component, don't forget to declare it with JUCE:
```
class MyComponent : public juce::Component{
public:
	void paint(juce::Graphics&) override;
	void resize() override;
private:
	JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR(MyComponent)
};

class MyJuceWindow : public juce::Component
MyJuceWindow::MyJuceWindow()
{
public:
	MyJuceWindow()
	{
		addAndMakeVisible(myComponent_);
	}

	void paint(juce::Graphics&) override;
	void resize() override;
private:
	MyComponent myComponent_;

	JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR(MyJuceWindow)
};
```