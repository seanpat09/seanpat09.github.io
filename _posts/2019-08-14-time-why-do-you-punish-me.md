---
layout: post
title: Time(zones), why do you punish me?
date: '2019-08-14-T00:00:00.000-07:00'
author: Sean Cuevo
description: Date fields are timezone agnostic until they are not
tags:
- salesforce
- time zones
- apex
- bugs
---

When it comes to Date values, we typically think of them as timezone agnostic. However, when comparing a Date value to Today's date, the timezone does matter. When "Today" ends depends on where you live and if you're not careful, you could be locking out users from a deadline sooner than you expect.

For example, let's say you are accepting applications until 7/31/2019 and your Apex controller validates this using System.today:

```
Boolean deadlineHasPassed(Date deadline) {
    return deadline < System.today();
}
```

If your applicant's user is in the GMT timezone, that means they will be locked out if they apply any time after 5pm PST on 7/31. Meanwhile, anyone with the PST timezone will have an extra 7 hours to apply. To give everyone an equal time period, you may want the deadline to actually be until midnight on 7/31 in PST.

You could easily fix this by changing the deadline to a Datetime field type so that you can take timezones into account, but that's not always an option. Your deadline could be based on an standard field like EndDate on Campaign and you typically don't want to replace a standard field with a custom one. You run the risk of other integrations relying on the standard field being populated, so you put yourself in a position where you have to constantly keep your custom field in sync with the standard one.

As a proxy, you could use the timezone of the org to figure out when midnight is and compare that to the current time. Sounds easy, right? Well as I quickly found out, there aren't any standard Apex methods that allow you to get a time based on a specified timezone. So here's what I needed to do:

* Figure out the current time using the `Datetime.now().getTime()` method, which returns you the number of milliseconds January 1, 1970, 00:00:00 GMT. This is an absolute time that is unaffected by timezone.

* Instantiate a DateTime object using the midnight of the deadline given the org's timezone, convert that to GMT, and then use getTime() to compare it to the current time.

I had some trouble find the code on how to do this, so here's mine:

```
Boolean deadlineHasPassed(Date deadline) {
    Timezone orgTimezone = Timezone.getTimeZone([SELECT TimeZoneSidKey FROM Organization][0].TimeZoneSidKey);
    Time midnight = Time.newInstance(23,59,59,999);

    //Get the number of seconds offset between GMT and the org's timezone
    Integer offsetSeconds = orgTimezone.getOffset(deadline) / 1000;

    Long deadlineTime =
        Datetime.newInstanceGMT(endDate, endTime) //Get midnight in GMT
            .addSeconds(-offsetSeconds) //Add the offset seconds so that you can get midnight of your org's timezone in GMT time
            .getTime();

    Long currentTime = Datetime.now().getTime() // get the current time;

    return deadlineTime < currentTime;
}
```

Now you can compare the current time with the end of a day without having to worry about timezones! I hope this saves someone some time (pun definitely intended).