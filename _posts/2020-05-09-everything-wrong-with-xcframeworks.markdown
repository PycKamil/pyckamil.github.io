---
layout: post
title:  "Everything wrong with XCFrameworks"
date:   2020-05-09 11:30:15 +0200
categories: programming, xcframework, xcode
---
Good old frameworks are dated back to time where there was only one platfrom and one language - Obj-C. Today we have multiple version of Swift and different platforms to support. 

When XCFrameworks was introduced in last years WWDC I was really excited:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">&quot;XCFrameworks make it possible to bundle a binary framework or library for multiple platforms —including iOS devices, iOS simulators, and UIKit for Mac — into a single distributable .xcframework bundle that your developers can use within their own applications.&quot; 😲</p>&mdash; Kamil Pyć (@KamilPyc) <a href="https://twitter.com/KamilPyc/status/1135628413279100929?ref_src=twsrc%5Etfw">June 3, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I would like to share my experience with it and what roadblocks you can hit trying to replace Frameworks with XCFrameworks in you project. 

Note: all of code and examples was run on Xcode 11.4.1. using Swift 5.2.

## Creating XCFrameworks

Lets start from the begining - how to create XCFramework. With Frameworks that just a normal target workflow as you may know from creating app or test targets. Just select "New..." and pick Framework as target.

![Framework in Xcode](/assets/images/framework_creation_xcode.png)

I would expect that from this point one can select which architectures to include and then export `.xcframework`. But it doesn't work this way. 

In order to produce xcframework we need to export all of the frameworks and then combine them with `-create-xcframework` command.

{% highlight shell %}

 xcodebuild -create-xcframework \
            -framework ${FRAMEWORK_SIMULATOR_DIR}/${1}.framework \
            -framework ${FRAMEWORK_DEVICE_DIR}/${1}.framework \
            -framework ${FRAMEWORK_MAC_DIR}/${1}.framework \
            -output ${OUTPUT_DIR_PATH}/xcframeworks/${1}.xcframework

{% endhighlight %}

You can check out full [create script on Github](https://github.com/bielikb/xcframeworks/blob/master/scripts/create_xcframeworks_catalina.sh) created by Boris Bielik - it's over 100 lines of code. 

I think that this over complication can be one of the reasons why XCFrameworks are [still not supported by Carthage](https://github.com/Carthage/Carthage/pull/2801).
	
## Xcode

On a first glance usage isn't different than how we work with standard frameworks. Just add it to target's "Frameworks and Libraries" section in general tab.
	
![Framework in Xcode](/assets/images/xcframework-general-tab.png)

After building app in build log we can find out that XCFrameworks get different treatment than Frameworks. 

There is a special `ProcessXCFrameworkLibrary` step that's extracts correct `.framework` from all architectures contained in xcframework bundle based on target architecture. 

{% highlight shell %}

ProcessXCFrameworkLibrary PromisesObjC.xcframework/ios-i386_x86_64-simulator/PromisesObjC.framework /Users/kamilpyc/Library/Developer/Xcode/DerivedData/NotchIt-axosvlxgvlvyerffinztzmvrhkfp/Build/Products/Debug-iphonesimulator/PromisesObjC.framework (in target 'TargetB' from project 'NotchIt')

{% endhighlight %}

Because of that Xcode need to have explicit list of xcframeworks to process and `FRAMEWORK_SEARCH_PATHS` does not work.
Xcode wants to optimise the usage of `ProcessXCFrameworkLibrary` and ther is only one call per single `xcframework`. That's will make build to fail in pretty common usage case.
Let’s say we have target `ModelA.framework` and `ModelB.framework` that links same `ModelCore.xcframework`. Xcode selects one of this targets to include the `ProcessXCFrameworkLibrary` step and if this selected target will start after another target utilising same xcframework build will simply fail with `Framework not found ModelCore`. I took me some time to figure out this race going on when my builds fails randomly 50% of time.
	
**WWDC2020 update:**
Xcode 12 fixes the race condition bug: 

{% highlight text %}

Fixed a race condition where multiple targets using the same XCFramework could result in non-deterministic build failures.
	
{% endhighlight %}

## Swift
In order to produce Swift libary that's support module stability, frameworks needs to be created with `BUILD_LIBRARY_FOR_DISTRIBUTION` turned on: 

{% highlight shell %}

xcodebuild archive \
    -workspace XCFrameworks.xcworkspace \
    -scheme ${1} \
    -destination "${2}" \
    -archivePath "${3}" \
    SKIP_INSTALL=NO \
    BUILD_LIBRARY_FOR_DISTRIBUTION=YES
{% endhighlight %}


Under the hood this will pass `-enable-library-evolution` flag to Swift Compiler. That have couple of implications:
* Code is compiled differently. When I tried to replace [Apollo](https://github.com/apollographql/apollo-ios) framework with precompiled xcframework version app started crashing with `swift_getEnumCaseMultiPayload.cold.1`
* If you have any enums you either have to mark them `@frozen` or handle default case on usage. Unfortunalty that make it impossible to simply replace `xcframework` with `.framework` in case any code needs to be debugged or profiled.
* Generated `.swiftinterface` file is corrupted. This was is [already reported](https://forums.swift.org/t/generated-swiftinterface-has-wrong-content/28543/3) but not solved, so you cannot build SwiftyJSON or RxSwift as xcframework.

## Wrap up
In summary, there are few things that can block developers from using XCFrameworks expecially with Swift projects. That could explain why they are not widely popular on iOS. We are one month ahead of WWDC and I hope those get issues got resolved, because XCFrameworks are great step ahead of Framework.
