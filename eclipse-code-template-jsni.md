---
title:Steve's first page
date:2011-10-20
---
# Eclipse code template for GWT jsni methods

GWT JavaScriptObject's and JavaScript Overlays are very cool. I've written about them before. It can get a bit tiresome to write jsni methods though, especially all those fiddly slashes, braces and asterisks at the end!

Here's a simple Eclipse code template that does the hard work for you, and lets you quickly tab-and-type through to completing your jsni method without ever having to type those fiddles again:

	public final native ${return_type} ${newName}()/*-{ 
	    ${cursor} 
	}-*/;

To add this in Eclipse:

1. Open the preferences dialog (Window -> Preferences)
2. Type "templates" into the filter box (it should have focus already so you can just start typing)
3. Select the bold entry under "Java -> Editor -> Templates"
4. In the right hand panel you'll see all the existing templates, and some buttons ... Click "New"
5. The "name" is also the shortcut for your new template, so enter "jsni"
6. Paste in the template text (copy from the code block above).
7. Click ok to return to prefs, and ok again to return to the editor and try it out ...

To use your new template:

1. Type "jsni" (without quotes) into a java editor and hit ctrl-space (or whatever your auto-complete shortcut is)
2. The first option in the auto-complete suggestions should be your "jsni" template - hit return to use it
3. Your template appears, and eclipse focuses the first part (the return type) for you to modify
4. Modify each part of the template in turn and press tab to move to the next part
5. Enjoy!
