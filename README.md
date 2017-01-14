This repository contains various systemd unit files to manage automatic daily
backups using borg(1). Both user and systemwide configuration is supported.

## Run daily backups as a system service

To run daily backups as a system service, you first have to create the necessary
configuration files. Copy the contents of `autobackup-example` to
`/etc/borg/autobackup` (you are going to have to create that directory).

    # mkdir -p /etc/borg/autobackup && cp autobackup-example/* /etc/borg/autobackup

Then tweak the configuration there to your liking.
- `environment` contains general options like the location of your repository,
  the ssh key to use to connect to it and the passphrase (you should probably
  also chmod this file to 600 if you use a passphrase).
- `excludes` contains a list of directories to exclude from the backup.
- `sourcedirs` contains a list of directories or files to back up

Now you have to link the system units into your system with

    # systemctl link $(pwd)/system/borgbackup.service
    # systemctl link $(pwd)/system/borgbackup.timer

The `$(pwd)` is necessary because `systemctl link` arbitrarily refuses to work
with relative paths (it would be as simple as passing it to `readlink -f` but
systemd doesn't want to do that).

Anyway, you can now enable and start the timer unit

    # systemctl enable borgbackup.timer
    # systemctl start borgbackup.timer

And that should be it.

## Run daily backups as a user service

The procedure is basically the same as the system service except that the
configuration is stored in `~/.config/borg/autobackup` and you have to add
`--user` to all systemctl commands (and run them as your user instead of root).

## Mount backup repository

This repo also contains a user service to mount the repo which you can link and
start with

    $ systemctl --user link $(pwd)/user/borgmount.service
    $ systemctl --user start borgmount.service

After you are done inspecting the contents of the repository, you can unmount it
again with

    $ systemctl --user stop borgmount.service

I would not advise keeping the backup mounted for a longer time since the mount
creates a lock on the repository which causes the creation of new archives to
fail. I should probably look at borgs source code and figure out if there is a
better way to keep the mount synchronized but I am lazy and have already spent
more time on this project than I wanted.
