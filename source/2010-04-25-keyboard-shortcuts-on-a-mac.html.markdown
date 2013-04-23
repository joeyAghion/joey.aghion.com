---
title: Keyboard shortcuts on a Mac
date: 2010-04-25 7:43
tags: 
---

I've been working exclusively on Macs for a while now, but am in the process of configuring my first brand-new machine. Sadly, I'm still not as fluid with Mac OSX keyboard shortcuts as I was on Windows. As a result, I'm much more mouse-dependent (and so slower) on the Mac. That ends now.

Luckily, **Command+Tab** for switching among applications works similarly to Windows' Alt+Tab. **Command-\`** (back-tick) usually allows you to switch between windows of a single application, but the occasional application uses **Ctrl+Tab** (like Windows) instead.

**Fn+F9** will show you (and, again, allow you to switch between) all open windows. 

**Command+M** will minimize the current window to the dock, although that's hardly ever useful to me since one of the first customizations I make to a machine is usually to **Automatically hide and show the Dock** (_System Preferences_ > _Dock_). 

Want to maximize a window the way you might on Windows? Not so fast. The green plus-sign (**+**) button looks like it might maximize the window, but in fact it chooses a best-fit based on the window's content. I don't know of a keyboard shortcut for triggering this effect, but the following post from Simon Dorfman provided an adequate solution:

[http://www.macosxhints.com/article.php?story=20051227001809626](http://www.macosxhints.com/article.php?story=20051227001809626)

Adapted slightly: 
1. Go to _System Preferences_ > _Universal Access_ and check _Enable access for assistive devices_
2. Paste the following script into a file called _Maximize.scpt_ in your _~/Library/Scripts_ directory:

    tell application "System Events"
      set FrontApplication to (get name of every process whose frontmost is true) as string
      tell process FrontApplication
        click button 2 of window 1
      end tell
    end tell

3. Download and install Quiksilver: [http://quicksilver.en.softonic.com/mac](http://quicksilver.en.softonic.com/mac)
4. Open Quiksilver _triggers_ (**Ctrl+Space** to open Quiksilver, then **Command-'** to open triggers)
5. Add a HotKey pointing to your _Maximize.scpt_ script, and assign it a keystroke (e.g., **Ctrl+Option+Z**)

Now, your custom keystroke should toggle between window sizes the way clicking the green **+** button would.

On Windows, hitting the Alt key to move focus to the menu bar is a convenient fall-back if you don't know the shortcut for a particular function. On Macs, hitting **Ctrl+fn+F2** does the same. (Note that this applies to the laptop-sized keyboards... full-sized ones probably don't require the fn.)

One final time-saver: Under _Keyboard_ > _Keyboard Shortcuts_, I updated the _Full Keyboard Access_ setting from _Text boxes and lists only_ to _All controls_. This way, tabbing between form fields selects radio buttons and checkboxes in addition to the usual text fields.

More shortcuts available at [http://support.apple.com/kb/ht1343](http://support.apple.com/kb/ht1343). Learn one a day!
