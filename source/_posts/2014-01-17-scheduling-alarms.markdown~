---
layout: post
title: "Scheduling Alarms in Android"
date: 2014-01-17 15:03:49 +0200
comments: true
categories: 
author: Geno Roupsky
published: false
categories:
- Android
---

Alarms in Android are just like normal alarms we use to remind us about events or that wakes us up in the morning. They serve to tell your app that the time has come to do something. 
What you're going to do is up to you and you can use the little note you left on the alarm to distinguish between different events.

So you have your app, and you want to fire up that beautiful notification to remind the user that he hasn't been using it for some time now and you wander how to do it. 
Well you may go and read the official docs over at [Android Developers](http://developer.android.com/training/scheduling/alarms.html) and wonder what the heck, or just scroll down and 
see the gists.

<!-- more -->

## Alarm types

First things first, let's split the two main use cases for Alarms in Android:

- **Timelapse Alarm** - supposed to fire after certain amount of time has elapsed
- **Calendar Alarm** - fires when specific date and time comes

### Timelapse Alarm 

Timelapse alarms are like the eggclock or a countdown timer - you are not interested what time it is when they fire, rather how much time has passed. 
So when you schedule one you just provide after what interval you need to be notified.

```java
    // Schedule an alarm to fire after half an hour
    alarmMgr.set(AlarmManager.ELAPSED_REALTIME,
            AlarmManager.INTERVAL_HALF_HOUR,
            alarmIntent);
```

The important thing is the use of **AlarmManager.ELAPSED_REALTIME** as first argument, which tells the AlarmManager to thread the second parameter as an interval from the current point of time.

### Calendar Alarm

This type of alarms are more suited for events that are tied to specific date and time, like the alarm that wakes you up at 7:00 am in the morning - you're not interested how much time there is until it fires, just that it fires at the specific time.

```java
    // You're better of using some nice lib like jodaTime, 
    // but for the sake of example I'll use the SimpleDateFormat
    SimpleDateFormat parser = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
    Date date = parser.parse("2016-02-29T07:12:00Z");
    alarmMgr.set(AlarmManager.RTC, 
            date.getTime(), 
            alarmIntent);
```

It is important to understand the first argument **AlarmManager.RTC** is what tells the AlarmManager we are providing an exact point in time when the alarm should be fired.

## Repeating Alarms

I wake up every day, can't I just tell the alarm to fire every day instead of scheduling it every time? Well you can, there is a similar method to **AlarmManager.set** that is intented to do just that:

```java
// Hopefully your alarm will have a lower frequency than this!
    alarmMgr.setRepeating(AlarmManager.ELAPSED_REALTIME,
            AlarmManager.INTERVAL_HALF_HOUR,
            AlarmManager.INTERVAL_HALF_HOUR, alarmIntent);
```

If you noticed the third argument now is the one that tells the interval in which the alarm should repeat. This is regardless of the alarm type, 
it's always interval of time in miliseconds, but it's good to have some predefined intervals for easier calculation:

* **AlarmManager.INTERVAL_DAY**
* **AlarmManager.INTERVAL_FIFTEEN_MINUTES**
* **AlarmManager.INTERVAL_HALF_DAY**
* **AlarmManager.INTERVAL_HALF_HOUR**
* **AlarmManager.INTERVAL_HOUR**

All of these constants may be used to calculate a required interval, for example if you want to set an alarm that repeats every Uranus's day (a day on Uranus is 17 hours 14 minutes and 24 seconds) you can do it like this:

```java
    static final long INTERVAL_SECOND = 1000l;
    // we actually calculate 17 hours and 15 minutes minus 36 seconds
    static final long INTERVAL_MARS_DAY = 
        AlarmManager.INTERVAL_HOUR*17
        + AlarmManager.INTERVAL_FIFTEEN_MINUTES
        - INTERVAL_SECOND*36;
```


## Alarm Intent

To create the intent that is send when alarm fires, you need to use one of the many methods on **PendingIntent** like:

```java
    // we need an intent that starts service
    Intent intent = new Intent(getContext(), HandsomeService.class);
    // and we create a PendingIntent so the AlarmService can fire it
    PendingIntent alarmIntent = PendingIntent.getService(getContext(), 0, intent, 0);
```

Beside the normal use of **Intent** and **Context**, the first number is requestCode, but for the sender, which currently according to docs is not used, so just put 0.
Second number is more intersting, it is a set of flags that controls how the pending intent will be created and/or updated:

* **PendingIntent.FLAG_ONE_SHOT** - this one just tells that the intent is for single use, fire it once and forge. 
If you are setting a repeating alarm you can't use this flag, because the second time the alarm will not have a usefull intent to send.
* **PendingIntent.FLAG_NO_CREATE** - will not create a new PendingIntent for the given Intent. 
It will not update the existing PendingIntent unless **PendingIntent.FLAG_UPDATE_CURRENT** is also provided. This is useful if you don't want to 
schedule a new alarm but rather to update the Intent of existing one.
* **PendingIntent.FLAG_CANCEL_CURRENT** - using this flag you will cancel all existing PendingIntents for your Intent, which means that all existing alarms will not be able to fire and you need to schedule a new one.
* **PendingIntent.FLAG_UPDATE_CURRENT** - this flag means to reuse the existing PendingIntent or create a new one, unless **PendingIntent.FLAG_NO_CREATE** is also set in which case if no previous intent exists, none will be created.
Using this flag you can update the existing alarms without rescheduling them again.

You may noticed that there may be several PendingIntents for a single Intent and that you can change the Intent. 
The intents are compared by their action, data, type, class and categories (check the docs for [Intent.filterEquals()](http://developer.android.com/reference/android/content/Intent.html#filterEquals(android.content.Intent))
so if you have two intents which are considered equal, you can update and cancel existing alarms.

## Canceling Alarm

To cancel an alarm you need to create a PendingIntent with Intent that is **Intent.filterEquals()** to the one used when created.

```java
    alarmMgr.cancel(alarmIntent);
```

## To Wake, Or Not To Wake

For the time being mobile devices have a limitted battery and we as application developers are responsible to make a good use of it (a user with dead battery is no user at all).
Android tries to optimize battery usage, by putting the while device in a sleep, and only the bare minimum functions that are necessary are alive and working. 
But because the process of sleeping and waking up takes some time and the user may feel disappointed the sleeping only occurs after some time. So even though the device is not doing anything usefull 
everytime it is awaken it spends some battery power to stay awake before actually going to sleep. Now your alarms may directly affect this process, because you can tell the android that your alarm
is so important that it needs to wake up the device and give you all the power to do your important work. On the other hand if you know that a delay of several minutes is not that important
you may as well tell that to the AlarmManager and allow the android system to wait for a really important work and at that to deliver your alarm.

### Waking

Well both flags **AlarmManager.ELAPSED_REALTIME** and **AlarmManager.RTC** have their counterparts accordingly **AlarmManager.ELAPSED_REALTIME_WAKEUP** and **AlarmManager.RTC_WAKEUP** that 
forces the device to wake up when the time for firing the alarm has come. But be carefull - waking up too often and the battery will start to drop fast and no one likes to keep an application 
that drains the battery!

### Inexacting

Additionally to forcing the device to wake or not, you can specify that the timing of the alarm is not exact and may happen within a specified time after the initial time. This is usefull because Android can 
batch multiple processes to run during a single wake up instead of waking for each one. The provided method is very similar to one used for repeating alarm and in fact can be used for repeating
if the PendingIntent is not marked as **PendingIntent.FLAG_ONE_SHOT**:

```java
    // lets schedule an alarm for after 1 hour with optional 15 minutes delay
    alarmMgr.setInexactRepeating(AlarmManager.ELAPSED_REALTIME,
        AlarmManager.INTERVAL_HOUR,
        AlarmManager.INTERVAL_FIFTEEN_MINUTES,
        alarmIntent);
```

You can use both Calendar and Timelapse alarms here, and you can use it for single alarms or repeating, the gotcha is that if you make it a repeating alarm,
the interval that the alarm can be delayed is equal to the interval of repeating, so it may as well be that you will receive the alarm when the time for next alarm has come.

**Note:** prior to API 19 the inexact repeating is inexact only when using one of the AlarmManager.INTERVAL_XXX constants. 


