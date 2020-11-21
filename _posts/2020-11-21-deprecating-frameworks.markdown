---
layout: post
title:  "Deprecating Frameworks"
date:   2020-11-21 20:10:15 +0200
categories: programming, framework, xcode
---
I can imagine that if Framework was an API we would probably see it marked as `@available(*, deprecated, message: "Use XCFramework now!")` when XCFrameworks was introduced in last year's WWDC. 

Although I couldn't find direct meantion of it in documentation or during presentations everything points into that direction.

## XCFramework
 
XCFramework is only way to support Mac Catalyst since Fat Framework don't contain multiple x86 slices, but since Catalyst is something new and only fraction of apps supports it isn't priority to support - for example Carthage [request for Catalyst support](https://github.com/Carthage/Carthage/issues/2799) is opened since June 2019.

Then Swift 5.3 brings binary dependencies capability to SPM and those can only be XCFrameworks. But again - not everyone uses SPM and one could stick to Framework till this point. But things have changed when Apple Silicone Macs were introduced last month. 
	
## Status quo

Since the first iPhoneOS SDK when we were building for a real device we used `iphoneos` sdk and ARM architecture, when we select a simulator we would use `iphonesimulator` sdk and x86 architecture. Then we could use `lipo` to combine them into Fat Framework that would work with both device and simulator. This is how Carthage works and most precompiled frameworks are distributed.

To illustrate - we have:

* ARM  ↔️ iphoneos
* x86 ↔️ iphonesimulator
 
This is where Apple Silicone break this order adding third pair:
 
* ARM ↔️ iphonesimulator


## Xcode 12

One thing changed in Xcode 12, one that is not clearly visible and many developers was surprised to discover. 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Did anyone encounter/found a fix for the issue where you build for Catalyst in Xcode 12.2 and Xcode just MUST build an arm64 version, ignoring the &quot;build active architecture only” setting? I can work around by always building all archs, but dang it so slow. <a href="https://t.co/tjnSzNinxa">pic.twitter.com/tjnSzNinxa</a></p>&mdash; Peter Steinberger (@steipete) <a href="https://twitter.com/steipete/status/1324297285489119237?ref_src=twsrc%5Etfw">November 5, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Let's build a Framework target for simulator and investigate it:

{% highlight shell %}

lipo -archs  Sample.framework/Sample
x86_64 arm64

{% endhighlight %}

By default Xcode will build for both ARM and x86. This new behavior will break Carthage that uses `lipo` under the hood to produce Fat Framework.

{% highlight shell %}

$lipo -create iphoneos/Sample.framework/Sample iphonesimulator/Sample.framework/Sample 

fatal error: Sample.framework/Sample have the same architectures (arm64) and can't be in the same fat output file

{% endhighlight %}

Fortunatly there is [document](https://github.com/Carthage/Carthage/blob/master/Documentation/Xcode12Workaround.md) describing how to exclude ARM architecture from building and basically restore how it worked in Xcode 11. Of course that will only work on Intel Macs, Apple Silicone users should wait for [XCFramework support](https://github.com/Carthage/Carthage/pull/3071) to land.


## Wrap up

If you distribute framework for other developers - please provide them with proper XCFramework with ARM simulator version so they can use simulator on there new Macs. I wish Apple would clearly state the need for migration to XCFrameworks so it would be done sooner - not when developers would start having compilation errors. 
