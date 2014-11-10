---
layout: post
title: Internet Explorer Testing VMs
date: 2013-02-01 09:17:00
author: Adam Presley
status: Published
tags: testing os browser
slug: internet-explorer-testing-vms
---

I'm not sure how long this site has been up but I'm just now finding it.
<http://www.modern.ie/> is a developer site with tools and resources to
help test and detect issues in older IE versions, as well as provide a
cloud-based testing suite offering. The particular feature I find most
cool is the ability to download VM appliances for IE versions 6-10. Not
only are VMs available for Windows hosts but they also provide Linux
hosts!  
  
To start visit <http://www.modern.ie/virtualization-tools>. On the right
hand side of the page you find a section named **Local virtualization**.
Let's say you want to test your web application in IE 6 on a Linux host.
In that section there is a box labelled **Select Desired Testing OS**
that you would click on and select *Linux*. This opens up another box
labelled **Select Virtualization Platform** where you would select
*VirtualBox for Linux*. This will open up a whole list of IE versions
you can download. In this example you would select **IE6 - WinXP**. This
will download a VM appliance that you can import into Oracle VirtualBox.
To do that:  
  
1. Extract the appliance file (.OVA) from the ZIP file you downloaded
* Open VirtualBox
* Go to File -\> Import Appliance
* In the dialog click on Choose... and select the .OVA file then click Next
* The settings for this appliance will be listed, then you can just click Import to continue

Once these steps are complete you will have a new VM called **IE6 -
WinXP**. If you start the VM now you might find that IE6 won't be able
to browse to any websites. This is most likely due to how the network
adapter for this VM is setup. To address this follow these steps:  

1. Right click on the VM named **IE6 - WinXP** and select *Settings...*
* In the setting dialog select *Network* from the list on the left
* In the tab labelled **Adapter 1** you will find it's *Attached to* setting has **NAT** selected. Change this to **Bridged adapter**
* Click **OK** to accept your changes

Now you are ready to start the VM appliance. Select it and click
**Start** to run the virtual machine. Once inside you can start running
IE6 for awesome testing goodness! Cheers!
