---
layout: post
title: "Diagram of a Activity's Life Cycle"
date: 2015-10-05 11:31:52 +0800
comments: true
published: true
sharing: true
footer: true
categories: android
---

A diagram present Activity's life cycle

Two Activity class named `A` and `B`, `A` is app entry.

Calling order is A call B , then B back to A.

1) Action: Launch App
  onCreate(A) -> onStart(A) -> onResume(A)

2) Action: startActivity(B)
  onPause(A) -> onCreate(B) -> onStart(B) -> onResume(B) -> onStop(A)

3) Action: goBack to Activity A
  onPause(B) -> onRestart(A) -> onStart(A) -> onResume(A) -> onStop(B) -> onDestory(B)

  > If B has been hold, it would not be onDestory(B)

![lifecycle](/images/refered/activity_life_cycle.png)
