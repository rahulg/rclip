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
