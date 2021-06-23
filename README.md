# About
It is a guide how to run [jitouch 2.74](http://www.jitouch.com/) at MacOS Big Sur. 

# Overview
User agent is created that runs at start up and keeps _jitouch_ alive by restarting it if it fails. Idea come from [this](https://v2ex.com/t/661325) terminal script implementation.

# Process

## Create _launchd_ file
_launchd_ is an Apple approved way to start/stopt something at startup. See more [here](https://launchd.info/).
Basically MacOS starts Daemons and Agents scripts (`.plist`). Agent may be a simple start of an app. We will create a script for user that launches __jitouch__ and keeps it alive all the time. 

Create `.plist` file with following content:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>local.start.jitouch2</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Users/[YOUR USER NAME]/Library/PreferencePanes/Jitouch.prefPane/Contents/Resources/Jitouch.app/Contents/MacOS/Jitouch</string>
	</array>
	<key>StandardErrorPath</key>
	<string>/dev/null</string>
	<key>StandardOutPath</key>
	<string>/dev/null</string>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```
！replace ___[YOUR USER NAME]___ with your user name. 

### Optional: Change owner and group for _launchd_ script file
This is not required if you run _jitouch_ for youself only. Keeping it here just in case as it took me some time to found. Assuming file name is `local.start.jitouch.plist`, execute following commands:

```
sudo chown root local.start.jitouch.plist
sudo chgrp wheel local.start.jitouch.plist
```

Note that you need to place scripts to different folder to run scripts for all users. 

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
If you see error `Load failed: 5: Input/output error` at the end, then some work around needed to load `.plist` file for the first time only. See [here](https://www.reddit.com/r/MacOS/comments/kbko61/launchctl_broken/). The easiest way for me was via [LaunchControl](https://www.soma-zone.com/LaunchControl/).

# All commands together
You can run below if you trust me 😁

```
cd ~//Library/LaunchAgents
touch local.start.jitouch.plist
echo '<?xml version="1.0" encoding="UTF-8"?>' >> local.start.jitouch.plist
echo '<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">' >> local.start.jitouch.plist
echo '<plist version="1.0">' >> local.start.jitouch.plist
echo '<dict>' >> local.start.jitouch.plist
echo '    <key>Label</key>' >> local.start.jitouch.plist
echo '    <string>local.start.jitouch2</string>' >> local.start.jitouch.plist
echo '    <key>ProgramArguments</key>' >> local.start.jitouch.plist
echo '    <array>' >> local.start.jitouch.plist
echo '        <string>/Users/'$(whoami)'/Library/PreferencePanes/Jitouch.prefPane/Contents/Resources/Jitouch.app/Contents/MacOS/Jitouch</string>' >> local.start.jitouch.plist
echo '    </array>' >> local.start.jitouch.plist
echo '    <key>StandardErrorPath</key>' >> local.start.jitouch.plist
echo '    <string>/dev/null</string>' >> local.start.jitouch.plist
echo '    <key>StandardOutPath</key>' >> local.start.jitouch.plist
echo '    <string>/dev/null</string>' >> local.start.jitouch.plist
echo '    <key>KeepAlive</key>' >> local.start.jitouch.plist
echo '    <true/>' >> local.start.jitouch.plist
echo '</dict>' >> local.start.jitouch.plist
echo '</plist>' >> local.start.jitouch.plist
pkill -f "Jitouch"
launchctl load local.start.jitouch
```

See Handling load error above if face `Load failed: 5`. 

# How to kill it
You won't be able to close _jitouch_ without stopping _launchd_ script. Script will keep on restarting _jitouch_. Do following command:

```
launchctl unload ~/Library/LaunchAgents/local.start.jitouch.plist 
```
