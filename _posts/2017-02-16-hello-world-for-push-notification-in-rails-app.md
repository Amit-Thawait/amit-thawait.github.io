---
title: Hello World for Push Notifications in a Rails App 
layout: post
published: true
category: programming
tags: [ruby, iOS, push notification]
comments: true
---

I was working for a product which comprised of 2 parts :

1. Rails App
2. iOS App

For every update in Rails App I needed to notify the iOS app about the change.

Since there was only an iOS app and no Andriod app hence I was looking for a library that can help me with this task.

After looking into various available libraries I found [Houston](https://github.com/nomad/houston){:target="_blank"} to be most suitable for my usecase.

But to get my work started I needed 2 things from my iOS dev :

1. p12 certificate with passphare
2. device token(specific per App per device) to which I will send a push

The p12 certificate shared by iOS dev needs to be converted to pem certificate for Houston gem to be able to read it. Command to do the same is shown below :

<script src="https://gist.github.com/Amit-Thawait/8a4b19e4617954f2512164f77d7e6d10.js"></script>

Next you just need to connect with the iOS device via device token and send an alert(push) to the device via `push` method as shown below. The below code can be tried on irb or rails console to see it in action.

<script src="https://gist.github.com/Amit-Thawait/d1ce8a318449b079dc55de1b91bb7a72.js"></script>

After the last command you will see a push received on your iOS device.

Simple. Isn't it.