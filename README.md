# Wat

Expose your clipboard over TCP. This sounds ridiculous, especially if you use a password manager, until you realise that this allows you to access
your clipboard while `ssh`ed into a box.

NOTE: you MUST trust `root` and anyone able to log in as your user on the remote and local systems if you use this, since they can connect to the
TCP socket and pull the contents of your clipboard. In addition, anyone with packet capture privileges on the remote end will see your traffic in
cleartext.


# Set-up

Scenario: you have a Mac that you use as a client with a remote dev box running Linux.

Place `rclip` somewhere convenient on your local Mac, we'll go with `/usr/local/bin`.

Generate a shared secret by running `rclip generate-secret`.

Edit your SSH config file (`~/.ssh/config`), and add the following:

```
Host my.dev.box
	PermitLocalCommand yes
	LocalCommand /usr/local/bin/rclip ensure-server 9110 &
	RemoteForward 9110 localhost:9110
```

On your remote dev box, put `rclip` somewhere in your `PATH` (such as `/usr/local/bin`).

Run `rclip set-secret` on the remote box and paste the secret from above.

`rclip paste` on the remote box will now paste the contents of your local box's clipboard.

It's recommended that you symlink `rclip` as `pbcopy` and `pbpaste` so that (a) your muscle memory continues to work,
and (b) neovim etc. pick it up and use it as your clipboard manager.

```
cd /usr/local/bin
ln -s rclip pbcopy
ln -s rclip pbpaste
```

`pbpaste` on the remote box functions the same as `rclip paste`, and the same applies for `pbcopy` and `rclip copy`.


## Set-up: launchd edition

Conceptually, it is preferable to launch your rclip server using launchd, but that increases the latency of copying & pasting by around 500ms.
I consider waiting nearly a second for a paste to be annoying.

If you don't, then skip the `LocalCommand` directives in your SSH config file,
drop `io.rahulg.rclip.plist` from this repo into `~/Library/LaunchAgents/`, and run
`launchctl load -w ~/Library/LaunchAgents/io.rahulg.rclip.plist`.


# CLI Details

```
rclip [mode] [port]
```

`mode` and `port` are optional. `port` defaults to `9110`, and `mode` defaults to `server`.

Valid modes:
- server (runs the server)
- secret (prints your secret)
- generate-secret (generates a new secret)
- set-secret (sets your secret)
- copy (runs pbcopy on the server)
- paste (runs pbpaste on the server)
- ensure-server (checks if a server is running, otherwise runs one)

Any symlinks to rclip ending in `copy` or `paste` to automatically call the respective command.
