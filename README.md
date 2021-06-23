# About
It is a guide how to run [jitouch 2.74](http://www.jitouch.com/) at MacOS Big Sur. 

# Overview
We create user agent that runs at start up and keeps _jitouch_ alive by restarting it if it fails. Idea come from [this](https://v2ex.com/t/661325) terminal script implementation and was adopted to launchd script.

# Process

## Create _launchd_ file
_launchd_ is an Apple approved way to start/stop something at startup. See more [here](https://launchd.info/).
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
cd ~/Library/LaunchAgents
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

# Kill duplicate process
I noticed that second Jitouche process starts after computer wakes. I'm still working on proper way to kill second process. Here is intermediate solution that checks if second process is present every 60 seconds and kills it. 

## Create bash killer script

Bash script will list all processes (`launchctl list`), find 'Jitouch' process (`grep`), get its name (`-o` and regex), and stops it (`launchctl stop`).

```
cd ~/Library/LaunchAgents
touch jitouch_kill_duplicate.sh
echo "#bin/zh" >> jitouch_kill_duplicate.sh
echo "launchctl list | grep -o '[a-zA-Z0-9\.]*Jitouch[a-zA-Z0-9\.]*' | xargs -I {} launchctl stop {}" >> jitouch_kill_duplicate.sh
```

## Create _plist_ to run periodically
Same as with _jitouch_, we create agent that executes killer script every 60 seconds. 

```
cd ~/Library/LaunchAgents
touch local.kill.duplicate.jitouch.plist 
echo '<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">' >> local.start.jitouch.plist
echo '<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">' >> local.kill.duplicate.jitouch.plist
echo '<plist version="1.0">' >> local.kill.duplicate.jitouch.plist
echo '<dict>' >> local.kill.duplicate.jitouch.plist
echo '    <key>Label</key>' >> local.kill.duplicate.jitouch.plist
echo '    <string>local.kill.duplicate.jitouch</string>' >> local.kill.duplicate.jitouch.plist
echo '    <key>ProgramArguments</key>' >> local.kill.duplicate.jitouch.plist
echo '    <array>' >> local.kill.duplicate.jitouch.plist
echo '        <string>Users/'$(whoami)'/Library/LaunchAgents/jitouch_kill_duplicate.sh</string>' >> local.kill.duplicate.jitouch.plist
echo '    </array>' >> local.kill.duplicate.jitouch.plist
echo '    <key>RunAtLoad</key>' >> local.kill.duplicate.jitouch.plist
echo '    <true/>' >> local.kill.duplicate.jitouch.plist
echo '    <key>StartInterval</key>' >> local.kill.duplicate.jitouch.plist
echo '    <integer>60</integer>' >> local.kill.duplicate.jitouch.plist
echo '</dict>' >> local.kill.duplicate.jitouch.plist
echo '</plist>' >> local.kill.duplicate.jitouch.plist
```

## Load launchd script

```
launchctl load  ~/Library/LaunchAgents/local.kill.duplicate.jitouch.plist
```
See Handling load error above if face `Load failed: 5`. 

# How to kill it completely
You won't be able to close _jitouch_ without stopping _launchd_ script. Script will keep on restarting _jitouch_. Do following command:

```
launchctl unload ~/Library/LaunchAgents/local.start.jitouch.plist 
```
