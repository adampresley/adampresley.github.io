---
layout: post
title: Logical Drives and WMI
date: 2007-02-01 07:39:00
author: Adam Presley
status: Published
tags: development c#
slug: logical-drives-and-wmi
---
In an effort to support removable media devices in SyncXpress I've had
to do a bit of learning about WMI in .NET. It is actually quite cool how
much information you can get from WMI about ANY piece of hardware
attached to your system, and even INSIDE your system. So in my case I
want to find out information about a logical drive device in Windows.
Using the *System.Management* namespace we can access the
*ManagementObject* classes that will allow us to query the WMI services
to get information about our device. Here is an example of getting
information about your **C:\\** drive.

{% highlight csharp %}
ManagementObject wmi;
string serialNumber, name, freeSpace;

wmi = new ManagementObject("Win32_LogicalDisk.DeviceID=\"C:\"");

serialNumber = wmi["VolumeSerialNumber"].ToString();
name = wmi["Name"].ToString();
freeSpace = wmi["FreeSpace"].ToString();
{% endhighlight %}

For more information on what data can be queried on a logical device see
<http://msdn2.microsoft.com/en-us/library/aa394173.aspx>
