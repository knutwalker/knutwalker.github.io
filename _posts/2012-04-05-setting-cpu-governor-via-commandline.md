---
layout: post
title: Setting the CPU governor via commandline
tags:
- shell
---

To set the CPU governor to `ondemand`, use the following:

``` sh
echo "ondemand" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

<br>

the governors available on most systems are : (check `cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_available_governors`)

<dl>
  <dt>ondemand</dt>
  <dd>the CPU is running at the lowest possible frequency and only scales up as needed</dd>

  <dt>consercative</dt>
  <dd>like ondemand, but the scaling happens slowly, one step a time</dd>

  <dt>performance</dt>
  <dd>the CPU is running at the highest possible frequency</dd>

  <dt>powersave</dt>
  <dd>the CPU is running at the lowest possible frequency</dd>

  <dt>userspace</dt>
  <dd>the frequency can be set by the user, as long as the CPU supports the desired frequency (that is, you cannot overclock using the userspace governor)</dd>
</dl>


<br>

As a important note, especially for laptop users:
The governor with the least power consumption is almost every time the *ondemand* governor and not the *powersave* governor.
Unlike *powersave*, *ondemand* allows the CPU to fall back to idle which consumes far less power than a CPU running at the slowest speed.


To scale the CPU in userspace mode, you can write the CPU frequency in specific files.

To get the possible frequencies, call
``` sh
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_available_frequencies
```



To get the current setting, use
``` sh
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq
```



and to set the frequency, use a command similar to the opening one:
``` sh
echo "1000000" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_setspeed
```
