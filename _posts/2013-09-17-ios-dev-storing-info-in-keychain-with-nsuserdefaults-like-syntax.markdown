---
layout: post
title: "IOS Dev - Storing Info in Keychain with NSUserDefaults like syntax"
date: 2013-09-17 17:55
comments: true
tags: [ios]
---

Recently, i decided to improve the security of one of my applications by storing some of the information that i was previously saving in NSUserDefaults in the Keychain. Basically, this doesn't make that much of a difference if your device is jailbroken. However, if your device is not jailbroken then it is possible to fetch the information saved by NSUserDefaults but not from the Keychain. This is because NSUserDefaults saves the information in an unencrypted format in a plist file inside the application sandbox which can be easily fetched. It can also be fetched from an iTunes backup. Hence, saving info in the Keychain adds an extra layer of security in that case. I started looking for some Keychain Wrappers and bumped into this amazing library from Github named [PDKeychainBindingsController](https://github.com/carlbrown/PDKeychainBindingsController). I found it ridiculously simple to use. First i added the relevant files to my project. ![1]({{site.baseurl}}/images/posts/keychain/1.png) Then i imported **PDKeychainBindings.h** in whichever file i wanted to use it and changed the implementation from NSUserDefaults which was like this...

<!-- more -->

<pre> NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
 [[Model sharedModel] currentUser] setAuthToken:[defaults objectForKey:@"authToken"]];
</pre>

to this...

<pre>PDKeychainBindings *bindings = [PDKeychainBindings sharedKeychainBindings];
[[[Model sharedModel] currentUser] setAuthToken:[bindings objectForKey:@"authToken"]];
</pre>

To save objects in the keychain, use this ..

<pre>PDKeychainBindings *bindings = [PDKeychainBindings sharedKeychainBindings];
[bindings setObject:@"XYZ" forKey:@"authToken"];
</pre>

As Simple as that ! I would recommend this library to anyone who wants to save information in their keychain because of its ease of use. If you still face any issues, write down a comment and i will get back to you :-)