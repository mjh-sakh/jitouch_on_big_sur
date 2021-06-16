# About
It is a guide how to run [jitouch 2.74](http://www.jitouch.com/) at MacOS Big Sur. 

# Overview
User agent is created that runts at start up and keeps _jitouch_ alive and directs discards all errors. 

# Process
There are X parts

## Create script file
Script starts jitouch and discards all error messages.

Navigate to `~/Library/LaunchAgents`. Create `.sh` file with following content:

```
#bin/zh
~/Library/PreferencePanes/Jitouch.prefPane/Contents/Resources/Jitouch.app/Contents/MacOS/Jitouch > /dev/null 2>&1
```

Explanation is [here](https://stackoverflow.com/a/10508862/12488601).

## Make script executable
Assumption is that script file name is `jitouch_run_script.sh`.

Execute follwing command:
```
chmod +x jitouch_run_script.sh
```

## Create _launchd_ file
_launchd_ is an Apple approved way to start/stopt something at startup. See more [here](https://launchd.info/).
Basically MacOS loads scripts from Daemons and Agents folders and execute them as specified. We will create script for user that launches above created bash script and keep it alive all the time. 

Create `.plist` file with following content:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>local.start.jitouch</string>
    <key>ProgramArguments</key>
    <array>
        <string>sh</string>
        <string>-c</string>
        <string>'~/Library/LaunchAgents/jitouch_run_script.sh'</string>
    </array>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```
Note: script can be shorter if aboslute path provided to the `.sh` file. Just us it inside single `string` tag. 

## Change owner and group for _launchd_ script file
Assuming file name is `local.start.jitouch.plist`, execute following commands:

```
sudo chown root local.start.jitouch.plist
sudo chgrp wheel local.start.jitouch.plist
```

Explanation is [here](Explanation is [here](https://stackoverflow.com/a/10508862/12488601).

## Close _jitouch_ to avoid duplicate instances running
Execute following command:
```
pkill -f "Jitouch"
```

## Load _launchd_ script
This will activate the script and jitouch will start:
```
launchctl load ~/Library/LaunchAgents/local.start.jitouch
```

### Handling load error
If you see error `Load failed: 5: Input/output error` at the end, then some work around needed to load `.plist` file for the first tiem. See [here](https://www.reddit.com/r/MacOS/comments/kbko61/launchctl_broken/). The easiest way for me was via [LaunchControl](https://www.soma-zone.com/LaunchControl/).

# All commands together
You can run below if you trust me 😁

```
cd ~//Library/LaunchAgents
touch jitouch_run_script.sh
echo "#bin/zh\n~/Library/PreferencePanes/Jitouch.prefPane/Contents/Resources/Jitouch.app/Contents/MacOS/Jitouch > /dev/null 2>&1" > jitouch_run_script.sh
chmod +x jitouch_run_script.sh
touch local.start.jitouch.plist
echo '<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">\n<plist version="1.0">\n<dict>\n    <key>Label</key>\n    <string>local.start.jitouch</string>\n    <key>ProgramArguments</key>\n    <array>\n        <string>sh</string>\n        <string>-c</string>\n        <string>Users/mjh-ao/Library/LaunchAgents/jitouch_run_script.sh</string>\n    </array>\n    <key>KeepAlive</key>\n    <true/>\n</dict>\n</plist>' > local.start.jitouch.plist
pkill -f "Jitouch"
launchctl load local.start.jitouch
```

See Handling load error above if face `Load failed: 5`. 
