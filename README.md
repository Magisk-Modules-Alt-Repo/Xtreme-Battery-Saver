
# XtremeBS

## There is currently an app in development

This module is for those who want to really stretch their battery and is designed to be highly configurable.

The main goal is to maximize battery life as much as possible while still remaining functional.

Most of the options are safe, meaning 

1. You dont have to worry about bricking or hard bootloops.

2. Some options may produce lag, late mesaages, missed alarms, or SystemUI crashes. 

You should probably start slow and activate 1 at a time with a little testing.

Everything has been built and tested on a Pixel 5 running ProtonAOSP.

## Supported Root Managers
- Magisk
- KernelSU Next (Thank you NanKill)

Probably KSU and APatch. I would love confirmation on these.

## Disclaimer
I am NOT responsible for any damages or data loss this software may cause. By using this software you accept the risks of using it.

## What can this do?
- kill, reprioritize, or suspend apps.
- put CPU cores into powersave mode.
- completely disable CPU cores.
- disable Google Play Services(gms).
- manage process priorities.
- force your device into doze mode.
- completely disable WiFi
- pretend the device has low ram

## Installation

1. Install using Magisk Manager and reboot.
2. Configure the options in the config file `/data/local/tmp/XtremeBS/XtremeBS.conf`

## Configuration
After you reboot from installation, XtremeBS drops a premade config file in `/data/local/tmp/XtremeBS/XtremeBS.conf`.

Every time you change the config, you must reboot or reload using `XBSctl reload`.

As of v1.0.6 there is a webui available at http://127.0.0.1:8081 to make editing the config easier
action.sh will launch it

### Config Options

The v1 config used a simple `key=value` format

The v2 config, uses a different format.
It replaces the `trigger` option with what i like to call "event blocks"
There are a few hardcoded ones (boot, screen_off, charging, manual, low_power) all of the hardcoded ones except manual, hook into the OS and use those as triggers. Then you have user-defined events. The user-defined events are named anything else and are defined exactly like the hardcoded ones.
they look like this:


    my_event={
    #some config options here
    }


NOTE: the `my_event={` and `}` have their own line. THIS IS VERY IMPORTANT

This change adds more dynamicism to XtremeBS, making it much more powerful
and allows you to change the balance of perfomance and powersaving
based on certain events. You may choose to stay on the v1 config if
you dont need this type of functionality, as it is backwards compatible.

you may name it whatever you wish, as long as you only use alphanumeric characters, underscores, or dashes.
No spaces
No $
No =
No { or }

Custom events are started using 

    XBSctl start my_event

And stopped using

    XBSctl stop my_event

The user-defined events are not called automatically by the daemon and are only known to the daemon if they are called like so. 

In any of the event blocks, you can leave them blank and they do nothing.

So if you wanted cpu cores 6 and 7 disabled during screen off and cpu cores 2-5 disabled on low_power, you can set those like so

    screen_off={
    disable_cores=cpu6 cpu7
    }
    low_power={
    disable_cores=cpu2 cpu3 cpu4 cpu5
    }

Each will take effect when they are triggered by the daemon but if both low_power is active and the screen is off, then both take effect. Meaning cpu cores 2-7 are disabled.

Now if you have defined cpu5 in each of those event blocks, then which ever event is called first, will disable cpu5 and whichever event deactivates first like the screen turning on, will re-enable cpu5.

Low power is triggered when you turn on your devices battery saver.

Screen off, charging, and boot are self explanitory.


Manual is a special case, it is started using

    XBSctl start

And stopped using

    XBSctl stop

Without any options and is like a user defined event, its really only for those who dont need more than 1 custom event. However you can specify it using XBSctl like any other custom event and it does the same thing.

Why events, DethByte64?
What if i told you, that you could do certain things when your battery saver turned on and do a different thing when your screen turned off, or do other things when your device begins charging?
Thats exactly what its for. Not to mention, you can use different allowlists with the `handle_apps` option per event, or literally any combination of config options per event.
This allows for many more possibilities than just "on" or "off".


#### trigger
This option specifies when to trigger power saving and is only necessary when using the v1 config.

Options:

`trigger=boot` The Battery saver starts at boot and is manually controlled using `XBSctl`. SAFE, BUT NOT RECCOMMENDED.

`trigger=auto` RECCOMMENDED/DEFAULT. This will trigger when Androids "Battery Saver" is toggled.

`trigger=manual` For manual control using XBSctl

#### delay

It sets a delay in seconds between checking for the trigger or commands to not spam the CPU.

Default is 3.

`delay=3`

#### keep_on_charge
This is for if you wish to keep the powersaver on while charging
added in v1.0.4
This leads to faster charging speeds
You only need to set this if you are using `trigger=auto`

`keep_on_charge=true` Default
 
`keep_on_charge=false`

#### allowlist
This takes a path to a file.

Default is `allowlist=/data/local/tmp/XtremeBS/apps.allow`

This file is not created, you must create it yourself and add **package** names to it, 1 per line.

As a bare minimum, you will need: your keyboard app, magisk manager (which is probably hidden, so wont be com.topjohnwu.magisk), and a terminal emulator (Termux).

That would look something like this

`com.topjohnwu.magisk
com.termux
com.google.android.inputmethod.latin`

You can find the package name of an app by long pressing the app you want on your home screen or app drawer, then selecting app info, and scroll to the bottom of that page.

If you set `handle_apps=suspend` you should probably add whatever you know youre going to use because you wont be able to open any other apps when the trigger is active.

If you fuck this up and dont add things to the allowlist for suspend mode, you will have to use adb from a PC.
`adb shell XBSctl safe`

#### denylist
By design, handle_apps only deals with user installed apps and not system apps to maintain stability.
If you so desire, you can use this denylist to deal with system apps.

default is `denylist=/data/local/tmp/XtremeBS/apps.deny`

You will have to create it to use it. It isnt mandatory.


#### handle_apps
This setting turns on or off the handling of apps.

Options:

`handle_apps=suspend` RECCOMMENDED
This will suspend every app. You cannot open suspended apps.
Be sure to have a whitelist in place.

`handle_apps=nice` DEFAULT

`handle_apps=kill` this just force stops all apps.
They can be restarted by other apps and you. 
This may take up more battery because of wasted cpu cycles.


#### handle_cores
This is like `handle_apps` but with CPU cores instead.

You might be asking, "this seems kind of risky. *how are* cores handled?". Thats a good question, with a simple answer. If this setting is turned on, it changes the CPU governors to powersave if its supported.

`handle_cores=auto` RECCOMMENDED, XtremeBS will figure it out for you.

`handle_cores=false` DEFAULT

`handle_cores="cpu4 cpu5"` a manual selection of the cores you want to put in powersave mode.

#### disable_cores
Which CPU cores to disable, if any.

`disable_cores=false` DEFAULT. Turns the setting off

`disable_cores=auto` RECCOMMENDED.
This automagically disables only the high power cores which *might* produce a tiny bit of lag, if youre okay with that.

`disable_cores="cpu6 cpu7"` you can manually set the cores you want to disable.

#### handle_gms
This decides how to handle Google Play Services, which is a battery hog.

`handle_gms=false` sets to not even bother with gms

`handle_gms=nice` DEFAULT. Sets the nice level to 19, the lowest priority.

`handle_gms=kill`
WARNING! If you use this option, it will break google apps until you change the option. Also, YOU WILL FAIL SAFETYNET AND PLAY INTEGRITY. It just disables the package.

#### handle_proc
Whether we should renice system daemons and processes. This is touchy. Whatever you put in the proc_file, will be reniced with a value of 10. You might get messages late, you might miss alarms, who knows. Just be careful.

`handle_proc=false` DEFAULT

`handle_proc=true`

#### proc_file
This file is where you put the processes you want to use less CPU power on. Personally, i put netd and system_server in here. One process per line.

`proc_file=/data/local/tmp/XtremeBS/proc.list` DEFAULT

Here is an example of what my proc_file looks like:
`/system/bin/netd 19
system_server 10`

the nice numbers were added in v1.0.4

##### A note on niceness
The numbers beside the process names are nice levels 0 (normal) - 19 (very nice). If you choose to not give them a nice level, the nice level will be 10.
If you dont know what "niceness" is, its pretty simple.
The nicer a process is, the less CPU time it gets, like a line in a grocery store. a really nice program will let others by so they can get done first.
When there are no more customers (processes) in line, then the nicer processes take their turn. the higher the niceness is, the more likely it will let the others go first.
There are negative numbers too. Negative numbers are NOT NICE and will get CPU time first. if you have an app that is really important, you can assign a negative number to it.
these range from -1 (politely aggressive) to -20 (hyper aggressive).
you can use not nice numbers in here but i dont recommend it.

If you want to find processes that use high cpu, you can use this command and it will sort the highest to the bottom. `su -c ps -eo "%cpu pid cmd" | sort -n -k1,1`
or if that doesnt work, you can use this one
`su -c "ps -A -o PCY,PID,USER,%CPU,ARGS" | sort -k4 -nr`

#### low_ram
This sets the low_ram flag to true, so the system tries to use less and therefore save power.

`low_ram=true` DEFAULT

`low_ram=false`

#### doze
This enables Doze Mode (added in v2.0.0)

`doze=false` DEFAULT

`doze=light`

`doze=deep`

Doze may cause issues with alarms

#### kill_wifi
This will disable the wifi radios (added in v2.0.0)
Not even the switch in Settings will re-enable it

`kill_wifi=false` DEFAULT

`kill_wifi=true`

You might be asking, "Why would anyone want to do this?" Well, WiFi is very power hungry and disabling it will save a ton of power.
Dont worry, it will be re-enabled when your trigger gets called.

#### notify
This option is to stop those annoying notifications that make your phone go bzzz when an event is changed

`notify=true` DEFAULT

`notify=false` Turns off all notifications

#### safemode
This option is for an all else fails and the system wont accept a XBSctl command.
This has never been used, nor was it requested, its just in here for persistence and if something really bad happens.
If all else fails, you can do this from your recovery mode.

`safemode=1` Enables Safemode

`safemode=0` Disables Safemode and removes its presence from the config.

You may also exit safemode the normal way as this is linked to XBSctl and vice-versa.

## XBSctl
This is how you control the daemon. 

commands are:

`start` Starts XtremeBS or an event in v2

`stop` Stops XtremeBS or an event in v2

`pause` Pauses trigger handling.

`resume` Resumes trigger handling or gets out of safe mode.

`reload` Reloads the config file. Only needed for v1

`safe` Safemode. If you mess up, use this. It stops XtremeBS and waits until you tell it to Resume, Then reloads the config.

## FAQ

Q: It softloops my device/systemui crash. What do i do?


**A: You tried too many options! Start slow, see what works and what doesnt.
   Note: Some Samsung devices dont like the CPU cores tampered with. Dont use any core options.
   If you have a Oneplus, don't use low_ram**

Q: Will this break my device?


**A: Probably not. If you move slow with your config options and test it, you should have a beefy config that does well.**

Q: Do i have to configure the module or is it plug n' play?


**A: Youre going to have to get your hands dirty and configure it yourself. I have thought about an "autoconfig tool/mode", its a concept ive been playing with for awhile now, but its a tough one to pull off while maintaining stability, not to mention, the codebase will be about 5x-10x what it is now, and a full rewrite would have to be done, just for that feature.**

Q: Will there be an app for this, to configure more easily?


**A: Yes! I am currently developing an app that will be packaged with it. Upon release, the webUI will likely be removed and compatibility mode (for v1 configs) will be removed.**

Q: How strong is this module?


**A: Its not for everybody, thats how strong it is. Scale of 1-10? 8.5-9.5. Its pretty stout. With enough tweaking, you could probably get device uptime to at least 5x what it was at stock. No other module is as strong as this.**

Q: When is the battery prediction update coming out?


**A: It was going to integrate with the WebUI, but ive decided it will come out with the app update**

Q: Will you write a config for me?


**A: No! You shouldnt ask anyone for a config, ever! Every device is different, your acceptable amount of lag is not another persons acceptable amount of lag, not to mention the security concerns.**

