---
layout: post
title:  "TIL: SwiftUI on Mac"
date:   2020-05-07 23:30:15 +0200
categories: programming
---
WWDC is near by, is good to catch up with stuff that is had on my TODO list - SwiftUI. I was tempted to try it on Mac to see how it manages to hide all of the complexity of AppKit.

I have mixed filling - right now SwifUI is great way to describe a layout, view hierachy and state managment. But if you need to do even a simple platfrom specific thing - let's say to change windows title - there is no simple way. Things like `navigationBarItems` are tided to UIKit only plaftorm so that would force to switch to `Mac Catalyst`.

I'm eagier to find out what WWDC 2020 will bring - death of AppKit or more complex SwiftUI integration.

Anyway I was able to create simple tool to inspect frameworks to check it's type and architecure. You can check it out in [Github](https://github.com/PycKamil/FrameworkInspector).
