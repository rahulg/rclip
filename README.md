# Wat

Expose your clipboard over TCP. This sounds ridiculous, especially if you use a password manager, until you realise that this allows you to access
your clipboard while `ssh`ed into a box.

Note: you MUST trust all users on the remote and local systems if you use this, since they can pull the contents of your clipboard.


# Set-up

Scenario: you have a Mac that you use as a client with a remote dev box running Linux.

Place `rclip` somewhere convenient on your mac, we'll go with `/usr/local/bin`.

Edit your SSH config file (`~/.ssh/config`), and add the following:

```
Host my.dev.box.local
	PermitLocalCommand yes
	LocalCommand /usr/local/bin/rclip ensure-server 9110 &
	RemoteForward 9110 localhost:9110
```

On your Linux dev box, put `rclip` somewhere in your `PATH` (such as `/usr/local/bin`).

`rclip paste` on the remote box will now paste the contents of your local machine.

It's recommended that you symlink `rclip` as `pbcopy` and `pbpaste` so that (a) your muscle memory continues to work,
and (b) neovim etc. pick it up and use it as your clipboard manager.

```
cd /usr/local/bin
ln -s rclip pbcopy
ln -s rclip pbpaste
```

`pbpaste` on the remote box functions the same as `rclip paste`.

## Set-up: launchd edition

Conceptually, it is preferable to launch your rclip server using launchd, but that increases the latency of copying & pasting by around 500ms.
I consider waiting nearly a second for a paste to be annoying. If you don't, then skip the `LocalCommand` directives
in your SSH config file, and drop the following into `~/Library/LaunchAgents/io.rahulg.rclip.plist` instead.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>io.rahulg.rclip</string>
	<key>ProgramArguments</key>
	<array>
		<string>/usr/local/bin/python3</string>
		<string>/usr/local/bin/rclip</string>
		<string>server</string>
		<string>9110</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
</dict>
</plist>
```

And run `launchctl load -w ~/Library/LaunchAgents/io.rahulg.rclip.plist`.

# CLI Details

```
rclip [mode] [port]
```

`mode` and `port` are optional. `port` defaults to `9110`, and `mode` defaults to `server`.

Valid modes:
- copy (runs pbcopy on your local machine)
- paste (runs pbpaste on your local machine)
- server (runs the server)
- ensure-server (checks if a server is running, otherwise runs one)

Any symlinks to rclip ending in `copy` or `paste` to automatically call the respective command.
