# Additional Setup

The following documents some Fedora setup that wasn't automated with ansible.

## Snapper

Snapper is used to create snapshots with the BTRFS filesystem for root and home
directories. I'd like to make these snapshots available at grub with 
[grub-btrfs](https://github.com/Antynea/grub-btrfs), but I've found that 
akmod-nvidia breaks it. Snapper is setup with:

```bash
sudo btrfs filesystem label / *FTL ship name*

# Make /var/log subvolume
sudo mv -v /var/log /var/log-old
sudo btrfs subvolume create /var/log
sudo cp -arv /var/log-old/. /var/log/
sudo restorecon -RFv /var/log
sudo rm -rvf /var/log-old

# Add /var/log to fstab
sudo vi /etc/fstab
# UUID=<drive uuid> /var/log  btrfs subvol=var/log,compress=zstd:1 0 0
sudo systemctl daemon-reload
sudo mount -va

# Create snapper configs
sudo snapper -c root create-config /
sudo snapper -c home create-config /home

# Allow users to perform snapshots
sudo snapper -c root set-config ALLOW_USERS=$USER SYNC_ACL=yes
sudo snapper -c home set-config ALLOW_USERS=$USER SYNC_ACL=yes
sudo chown -R :$USER /.snapshots
sudo chown -R :$USER /home/.snapshots

# Add / and /home to fstab
sudo vi /etc/fstab
# UUID=<drive uuid> /.snapshots      btrfs subvol=.snapshots,compress=zstd:1 0 0
# UUID=<drive uuid> /home/.snapshots btrfs subvol=home/.snapshots,compress=zstd:1 0 0
sudo systemctl daemon-reload
sudo mount -va

# Show resulting subvolume structure
sudo btrfs subvolume list /
```

## Wireguard Client

Wireguard is nice for a home vpn and [pivpn](https://pivpn.io/) makes it easy.

1. Create client on server and copy resulting `.conf` file to local machine
2. Import to networkmanager with:
```bash
nmcli connection import type wireguard file <conf file from pivpn>
```
3. Turn on/off from nm-applet

## Mount network drives

I find fstab messing about more troubule than it is worth. Mount network drives 
when needed with the following command:

```bash
linux-mount-<network drive name>
```

## Taskopen for taskwarrior

[taskopen](https://github.com/jschlatow/taskopeni) is easier to install 
manually at this point since it isn't packaged and uses nim. Might get this 
automated in the future.

```bash
curl https://nim-lang.org/choosenim/init.sh -sSf | sh # install nim
git clone https://github.com/jschlatow/taskopen.git
cd taskopen
make PREFIX=/usr
sudo make PREFIX=/usr install
```

## Syncthing 

Syncthing is used to sync folders between various computers and android. The 
ansible script should setup and run the service, but shares must be setup
via the web gui. Currently four shares exists:
- `.warrior` - `.task` and `.timewarrior` folders to sync taskwarrior tasks.
These two folders are symlinked to the home folder where taskwarrior/timewarrior 
expects them.
- `warrior` - contains text files associated with taskwarrior (mostly from
taskopen).
- `phone photos` - personal photos synched from android.
- `phone screenshots` - personal screenshots synced from android.
- `ssh_keys` - contains ssh keys for git remotes (~/.ssh/keys)
- `vimwiki` - contains text files associate with my personal vimwiki.

## Lxappearance

My GTK theme is pulled down by chezmoi, but isn't active by default. This can
be fixed with the lxappearance gui (for X sessions).

## Git SSH for personal and work

...
