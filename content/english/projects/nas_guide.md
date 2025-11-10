---
title: 'Guide to Building Your Own Home Server'
hideDate: true
draft: false
ShowToc: true
TocOpen: true
---


## About

This guide contains various pieces of information about building your own home server. I arrived at these solutions mostly by poking around here and there, and I tried to include links to external resources if you want to learn more about any specific aspect. In particular, this guide details how to:

  - achieve a satisfactory hardware solution for a home server
  - set up a storage array with redundancy
  - set up a Samba server accessible on the local network to store your data
  - synchronize your data between this storage array and an off-site "cold storage"
  - access your server via SSH
  - set up a VPN to access this server from outside
  - install a Jellyfin media server on this server
  - serve this Jellyfin server on the web *via* a Cloudflare tunnel

**Note:** This guide assumes you have a basic knowledge of the Linux command line. If not, you can find plenty of resources online, starting with [this one](https://ubuntu.com/tutorials/command-line-for-beginners). Also, most links are archived when relevant, see [this page](about/links) for more details.

## Why reinvent the wheel?

Instead of following this guide, you can absolutely use one of the following solutions:

  - **Buy a Synology NAS**: Synology is probably not the only brand doing this, but they offer [NAS devices with multiple drive bays and turnkey software](https://web.archive.org/web/20250504110217/https://www.synology.com/fr-fr/products/DS923+), especially for remote access. However, you're limited to the features Synology has implemented, and it's not very robust if the company changes its policies or goes bankrupt: it's a proprietary black box, whereas most of the solutions below are Open Source and/or have alternatives.
  - **Use TrueNAS Scale**: [TrueNAS Scale](https://www.truenas.com/truenas-scale/) is a Debian-based OS that offers a number of default features for managing a storage server. Again, you're limited by their app solutions, but since it's Linux under the hood, you can always take control *via* the terminal.

The point of building your own storage server *from scratch* is multi-fold. It allows you to:

  - Use the machine however you want and install apps that aren't provided with turnkey solutions: **scalability**.
  - Have control over every step of the process and not depend on any particular company: **resilience**.
  - Learn and understand what you're doing as you go: **education**.

Personally, I learned a ton by building this server, both about how Linux works and about networking in general. It's a great way to get a feel for system services, routing, *etc.*, which are the foundation of computing and the internet.

# Part I: Choosing the Platform

## What hardware for a home server?

### Power consumption

The server will run 24/7, so power consumption is one of the main things to watch out for. A desktop tower can easily consume around 100 W at idle (sources: [here](https://web.archive.org/web/20250504101356/https://www.reddit.com/r/watercooling/comments/16szgmr/whats_your_idle_power_usage/?rdt=41155) and [here](https://web.archive.org/web/20250504101310/https://www.techpowerup.com/forums/threads/high-power-consumption-in-idle.314694/)), while a recent laptop or Mac mini can go below ten watts (see [here](https://web.archive.org/web/20250504101801/https://www.reddit.com/r/linuxquestions/comments/zqolh3/normal_power_consumption_for_laptop/?rdt=32814) and [here](https://web.archive.org/web/20250504101903/https://www.reddit.com/r/linux/comments/zwar52/haha_suck_on_dat_windows_finally_got_idle_power/?rdt=37144)), but those are potentially (very) expensive (even used, a Mac mini M*x* costs a few hundred euros). That said, in the coming years, an old laptop or Mac mini will become attractive alternatives...

Meanwhile, for a super cheap solution, *thin clients* are an interesting option because they cost next to nothing if refurbished. With 16 GB RAM and 240 GB SSD, a Dell Wyse 5070 costs only 110€ on sites like Remarkt (bought in January 2025). Prices fluctuate a lot, but there are often good deals, and you can build a server for about a hundred euros. The advantage of this kind of machine, even if the CPU is slow, is a power consumption between 4 and 10 W. For reference, **10 W at 20 cents per kWh means about twenty euros per year in electricity bill**.

### Hardware decoding capability

If you want to build a media server and use the streaming feature from [Jellyfin](https://jellyfin.org/), it might be worth investing in a beefier setup, capable of transcoding a 4K h265 stream. The catch is that this easily adds a few hundred euros of hardware (tower format, GPU cost) and increases the electricity bill (about 200€ / year for a tower that uses around 100 W).

**Note:** if you disable [transcoding](https://web.archive.org/web/20250505104453/https://jellyfin.org/docs/general/post-install/transcoding/) in your Jellyfin settings, the original video stream will be sent as-is. This means much less work for your server, but the client must have the right hardware/codecs to decode the video stream. For example, my DELL Wyse 5070 is maxed out (all cores at 100% and playback stutters) on a 1080p h265 @9.2 Mbps stream with transcoding, while direct play *without transcoding* uses less than 5% of the cores... But the client (PC/smartphone/TV) must be able to decode the received video streams!

### CPU architecture

Most of the tools used in this guide, and the vast majority of sysadmin documentation, are for Linux systems. Also, since Linux support on Apple ARM architectures is still a bit shaky, it's better to go with a standard x86 platform, so no Mac minis or MacBooks M*x* (even though things are improving, see [here](https://web.archive.org/web/20250508083522/https://asahilinux.org/)).

### Connectivity

Potentially important if you plan to *work* on data stored on the server (for video editing, for example), make sure you have a fast enough Ethernet port. At least a Gigabit port (~110 MB/s real), but a 2.5GbE or even 10GbE (~1 GB/s) port (or PCI-Express expansion card) can be worth it for [about sixty euros](https://web.archive.org/web/20250504103958/https://www.amazon.fr/TP-Link-TX401-Ethernet-ultra-faible-Compatible/dp/B08GFGG888).

### Storage

#### HDD or SSD?

In January 2025, the cost per terabyte is about [25€ for HDD storage](https://web.archive.org/web/20250504111740/https://www.amazon.fr/Seagate-IronWolf-Disque-interne-ST12000VNZ008/dp/B08147LZFD) vs. [60€ for SSD storage](https://web.archive.org/web/20250504111639/https://www.amazon.fr/Technology-Comprend-Western-Digital-Migration/dp/B0D7MLB76V). Also, long-term storage is not recommended on SSDs ([data volatility](https://web.archive.org/web/20250504112057/https://storedbits.com/ssd-data-retention-period-without-power/)), so HDDs remain the go-to for long-term, high-volume storage, despite the noise (power consumption isn't necessarily higher for HDDs than SSDs, see [here](https://superuser.com/questions/1171866/) and [here](https://web.archive.org/web/20250504113158/https://theoverclockingpage.com/2024/02/25/review-ssd-acer-fa200-2tb-the-fastest-qlc-ssd-we-ever-tested/?lang=en)).

#### What kind of enclosure?

If you go with a tower, you can put the drives inside. If you go with a laptop or barebone, you'll need to buy an external enclosure. No need for RAID in the enclosure, since ZFS (see below) makes that unnecessary.

However, cheap HDD docks have a bad reputation (hence the nickname "disk fryer"), so it might be worth spending a bit more on something like a [QNAP TR-004](https://web.archive.org/web/20250504113940/https://www.qnap.com/fr-fr/product/tr-004).

#### How many drives, what capacity?

This depends on the size of your bay (number of drives), your budget, and how much storage you want. If you go for a 4-bay setup and RAIDZ (ZFS equivalent of RAID 5), the usable capacity will be about three times the capacity of a single drive (you lose 30%). So, for 4x12TB drives, you'll end up with about 34TB of usable storage.

**Note:** If you're not familiar with RAID, read [this](https://en.wikipedia.org/wiki/RAID) or [that](https://web.archive.org/web/20250504115209/https://eshop.macsales.com/blog/56056-a-beginners-guide-to-understanding-raid/). For a RAIDZ capacity calculator, go [here](https://wintelguy.com/raidcalc.pl).

## Operating system

As mentioned above, for tooling reasons, we'll go with a Linux distribution. There are [plenty](https://en.wikipedia.org/wiki/List_of_Linux_distributions), and I personally chose **Lubuntu**, which is an Ubuntu variant using LxQt, so you get a Debian system with all its advantages, and a system RAM footprint among the lowest possible (see [here](https://web.archive.org/web/20250508133609/https://www.androidauthority.com/linux-distro-least-ram-3489365/) and [here](https://web.archive.org/web/20250320065859/https://www.reddit.com/r/Lubuntu/comments/1g2gmp8/general_appreciation_lubuntu_is_a_welloptimised/)).

Having Lubuntu also lets you connect a screen to the server if needed, and still have a GUI. This can also be handy if you want to use the server as both a NAS **and** a media server connected to your TV, for example. If you only plan to access the server via the command line, you can just install Ubuntu Server, which should be even lighter on RAM usage than Lubuntu.

# Part II: Setting up the different services

From this point on, this guide assumes you have a working Linux system, connected to the internet, and to a number of hard drives, whether internal or external. The following commands are to be run in the terminal (I haven't added `sudo` everywhere, but it's often necessary) on the machine itself, or via SSH. In this latter case, we can thus start by configuring an SSH server on our machine, so that we can put it in the corner of a room without needing to connect a screen / keyboard / mouse to it, and do all our administration remotely.

## Creating an SSH server

Follow these steps (see the [doc](https://web.archive.org/web/20250504214212/https://documentation.ubuntu.com/server/how-to/security/openssh-server/index.html)):

  - Install `open-ssh` with `sudo apt-get install openssh-server`
  - (Re)start the service with `sudo systemctl restart ssh.service`, you can also check if the service is enabled with `sudo systemctl status ssh.service`, and do a `sudo systemctl enable ssh.service` if it's not.

**Notes:**

  - You shouldn't have to open port 22 in the firewall as it's open by default. Otherwise, `sudo ufw allow 22/tcp`.
  - If you need to change the default configuration, edit `/etc/ssh/sshd_config` and restart the service.
  - **Warning** if you want to be able to connect via SSH from outside networks (*i.e.* the internet). You can do [port-forwarding](https://en.wikipedia.org/wiki/Port_forwarding) to send a port from your public IP to port 22 of your server, but it's recommended in that case *not* to use port 22 on your public IP and to disable password login in favor of a [certificate-based solution](https://web.archive.org/web/20250505100403/https://goteleport.com/blog/how-to-configure-ssh-certificate-based-authentication/).

## Storage: ZFS arrays and disk health

First, you can read up on the concept of data backup in general, and the [3-2-1 backup strategy](https://en.wikipedia.org/wiki/Backup#3-2-1_Backup_Rule) in particular. In short, it means having 3 copies of your data, on 2 different types of storage, with 1 off-site (in case of fire or whatever). Also, keep in mind that [**RAID is NOT a backup**](https://web.archive.org/web/20250428002448/https://www.raidisnotabackup.com/).

That said, you're building a personal server and the above considerations, while ideal for institutional data backup, aren't always feasible for individuals. Personally, I went for RAID (local redundancy) + 1 off-site copy (cold storage). That seemed like a good compromise in terms of cost *vs* security *vs* data redundancy.

### About RAID

As mentioned above, I recommend reading [this](https://en.wikipedia.org/wiki/RAID) or [that](https://web.archive.org/web/20250504115209/https://eshop.macsales.com/blog/56056-a-beginners-guide-to-understanding-raid/) to get familiar with RAID. In short, for storage, I would recommend:
  - RAID 1 (mirror) if you only have two drives, but you'll lose half your storage capacity.
  - RAID 5 (stripe) if you have three or more drives, you'll only lose about 1/N of the capacity, N being your number of drives.

For your storage, you can use simple RAID if your enclosure's controller or motherboard supports it. I preferred to use ZFS, which has some advantages over RAID.

### About ZFS

ZFS, or Zettabyte File System, is a file system with some interesting properties for a storage server:

  - You can mount any number of hard drives into *pools* (or [*tanks*](https://serverfault.com/questions/562564/why-are-all-the-zpools-named-tank)), with [different RAID levels](https://web.archive.org/web/20250323043434/https://www.raidz-calculator.com/raidz-types-reference.aspx), similar (roughly) to RAID 1 and 5.
  - ZFS has the concept of snapshots ([*snapshots*](https://web.archive.org/web/20250216204631/https://docs.oracle.com/cd/E19253-01/819-5461/gbcya/index.html)), which let you capture the system state at a given time and roll back to it *a posteriori*.
  - ZFS is purely software. The drives can be plugged into different ports on the server. You could even imagine a ZFS pool using, in the same RAIDZ, an internal HDD and an external SSD connected via USB. More interestingly, you can *export* your pool from one machine, plug the drives into another, and *import* it there. Thus, you're not tied to a particular hardware RAID controller (and especially not to the failure of one!).
  - ZFS offers performance comparable to hardware RAID ([source](https://web.archive.org/web/20250508152314/https://www.krenger.ch/blog/raid-z-vs-hardware-raid-5/)).

**Fun fact:** a zettabyte is 10^21 bytes, or about 2^70 bytes. ZFS can actually allocate volumes of 2^128 bytes, or over 10^26 terabytes! ([source](https://en.wikipedia.org/wiki/ZFS))

### Final setup

Personally, I went for:

  - a QNAP TR-004 bay for server storage, with four 12 TB drives (SST12000VN0008, Seagate Ironwolf) in RAIDZ, for about 31 TB of usable storage.
  - a dual-bay dock for two 3.5" drives, with two 12 TB drives in ZFS mirror, for 12 TB usable as off-site cold storage that I periodically connect and sync with my server for important data.

### Creating a ZFS pool

Once your drives are connected, you can create your pool like this:

`zpool create [pool name] [raid type] [device id1, device id2, etc.]`

`[raid type]`: desired RAID type, typically `raidz` for a ZFS equivalent of RAID 5 (stripe), `mirror` for an equivalent of RAID 1 (mirror).

`[device idx]`: unique identifier for the drives you want to include in the pool. On my drives, it looks like `ata-ST12000VN0008-2YS101_XXXXXXXX` (where `XXXXXXXX` is a serial number). To list all installed drives and get these IDs, run `ls -lh /dev/disk/by-id/`.

**Notes:**

  - Some tutorials use `/dev/sdx` as drive IDs. [This is not recommended](https://web.archive.org/web/20250508160418/https://unix.stackexchange.com/questions/474371/how-do-i-create-a-zpool-using-uuid-or-truly-unique-identifier) because the `x` can change, while the above ID is unique to a particular piece of hardware.
  - Once the pool is created, it's mounted automatically under `/` by default, but you can change this with the `-m [mountpoint]` option.
  - If you want to disconnect your drives for storage (for cold storage) or connect them to another machine, do a `zpool export pool_name` first (and on reconnection, do a `zpool import pool_name`).
  - For documentation, besides the `man` pages, see [Oracle's doc](https://web.archive.org/web/20250508155543/https://docs.oracle.com/cd/E19253-01/820-2315/index.html), which is very complete, as well as [this tutorial](https://web.archive.org/web/20250508155709/https://ubuntu.com/tutorials/setup-zfs-storage-pool#1-overview) (basic) and [this one](https://web.archive.org/web/20250508160141/https://ikrima.dev/dev-notes/homelab/zfs-for-dummies/) (more complete).

If all went well, you now have a ZFS storage pool mounted at your server's root, and you can store data on it as you would on any other partition.

### Fixing non-mounting after reboot

One issue with expansion bays equipped with hard drives is that the drives often spin up one after another and take about 5 seconds each to reach full speed. If you leave your configuration as is and reboot the server, it's likely that your ZFS pool won't mount, because the `mount` command launched by the ZFS service at boot will be executed **before** the drives have reached full speed and are recognized by the system. In that case, you'll have to manually re-import the pool with `zpool import`. To fix this, you need to add a delay to the ZFS pool mounting to give the drives time to spin up (see [this guide](https://web.archive.org/web/20250510133455/https://ounapuu.ee/posts/2021/02/01/how-to-fix-zfs-pool-not-importing-at-boot/)):

  - Check the ZFS journal to see what's wrong: `journalctl -u zfs-import-cache.service -n 50`
  - Edit the config file to add a delay with `sudo systemctl edit zfs-import-cache.service` and add the following lines (list available drives with `lsblk`, which allows us to see if some are missing, and then wait 20 s)

```
[Service]
ExecStartPre=/usr/bin/lsblk
ExecStartPre=/usr/bin/sleep 20
```

20 s is enough for my QNAP TR-004 with four drives, but you may need more depending on your setup and how long your drives take to spin up. While good enough for home use, note that this is quite a dirty workaround, and a better solution would be to probe for disk presence before mounting explicitely instead of just waiting a given amount of time.

### Reading S.M.A.R.T. attributes

[Self-Monitoring, Analysis and Reporting Technology](https://en.wikipedia.org/wiki/Self-Monitoring,_Analysis_and_Reporting_Technology) (SMART) is a tool to monitor your drives' health. Generally, you read SMART attributes with the `smartctl` command. Typically, `smartctl -a /dev/sdx` gives all SMART info for that drive (more info [here](https://web.archive.org/web/20250508162741/https://wiki.evolix.org/HowtoSmart)).

**Notes:**

  - For the QNAP TR-004, you need to specify the disk controller ID with the `--device` option, *i.e.* with `smartctl -a /dev/sdx --device=jmb39x-q,N` (see [here](https://web.archive.org/web/20241208154350/https://github.com/smartmontools/smartmontools/pull/47)), where N is 0-3 and is the disk you want to check among the four in the bay.
  - For a more robust version that survives disk name changes (a disk can be `/dev/sdb` one day and `/dev/sdc` after a reboot or hardware change), use `/dev/disk/by-id/ata-ST12[...]`, which is unique to each disk (see [here](https://superuser.com/questions/1848213/predictable-device-names-in-smartd-conf-on-various-linux-like-systems)).
  - The JMB39x controller in the TR-004 doesn't support reading SMART test results, you only get the main metrics visible with `smartctl -a` (see [here](https://web.archive.org/web/20250510062531/https://github.com/smartmontools/smartmontools/issues/64)).

You can just run `sudo smartctl -a /dev/sdx | grep overall-health` periodically to get a rough idea of a disk's general health. However, the SMART `overall-health` flag [can be misleading](https://web.archive.org/web/20250227141728/https://unix.stackexchange.com/questions/502693/smartctl-reports-overall-health-test-as-passed-but-the-tests-failed) (see also [here](https://web.archive.org/web/20250430041248/https://www.smartmontools.org/wiki/FAQ#ATAdriveisfailingself-testsbutSMARThealthstatusisPASSED.Whatsgoingon)). For a better idea and to automate tests, you can use `smartd`, a system daemon that can run tests automatically and notify the sysadmin by email. For that, you need to:

  - Set up a way for your server to send emails.
  - Configure `smartd` to run a test periodically and send a warning email if a disk starts to fail.

#### Setting up a mail client

You can use `msmtp` to let your server send emails. Start by installing the `msmtp` package (typically with `apt-get install msmtp`). Then edit the config file at `/etc/msmtprc` to add your email account settings, typically something like:

```
# Default values for all accounts.
defaults
auth           on
tls            on
tls_starttls   on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /var/log/msmtp

# Example for a Gmail account
account        gmail
auth           plain
host           smtp.gmail.com
port           587
from           your.account@gmail.com
user           your.account
password       your password (see notes below for gmail!)

account default: gmail
```

To test if your config is good, the following command:

```
printf "Subject:Whatever\n\nTheMessageBody" | sudo msmtp your.account@gmail.com
```

should send you an email with the subject and body defined by the `printf`.

**Notes:**

  - When installing `msmtp`, it's recommended to disable AppArmor due to [errors it can cause](https://web.archive.org/web/20250406031854/https://askubuntu.com/questions/878288/msmtp-cannot-write-to-var-log-msmtp-msmtp-log).
  - The `account default` line lets you choose the account to use when nothing is passed to the `-a` argument of `msmtp`. You can have several email accounts configured in `msmtprc`, and choose which to use with `msmtp -a gmail` or `msmtp -a outlook` when sending.
  - For a Gmail account, and probably for other providers, you can't use your main password for security reasons. Google lets you get an ["app password"](https://web.archive.org/web/20250123175809/https://support.google.com/accounts/answer/185833?hl=fr) instead, which replaces your password in `msmtprc`.
  - Remember to do `sudo chmod 400` on `msmtprc`.
  - Don't forget the `sudo` before `msmtp`, so it looks for its config in `/etc/msmtprc` (the root config file) and can write to its log file (`/var/log/msmtp`. If you want to run the command without `sudo`, put your config file in `~/.msmtprc` as explained [here](https://web.archive.org/web/20250406031854/https://wiki.debian.org/msmtp) or do `sudo chmod 666` on `/var/log/msmtp`.

Once `msmtp` is installed and working, you also need to install `mailutils` to get the `mail` command used by `smartd` (which will use your `msmtp` config). If all goes well, the command

```
printf "TheMessageBody" | mail -s "Whatever" your.account@gmail.com
```

should also send you an email with the body defined by `printf` and the subject defined with the `-s` option (see [the doc](https://web.archive.org/web/20250510115259/https://manpages.ubuntu.com/manpages/noble/man1/bsd-mailx.1.html) for `mail`).

#### Configuring `smartd`

Now, configure `smartd` to send an email if it detects a problem on a disk. Do this by editing `/etc/smartd.conf`. At first, you can leave all lines commented and just add the line `DEVICESCAN -a -n standby -m your.email@gmail.com -M test`. If all is well, restarting `smartd` with `sudo systemctl restart smartd` should send you an info email. If not, check the `smartd` journal with `journalctl -xeu smartmontools.service` or the `msmtp` logs with `tail /var/log/msmtp`.

For an explanation of `smartd` commands, see [the doc](https://web.archive.org/web/20250510120341/https://man.freebsd.org/cgi/man.cgi?smartd.conf%285%29), which is very well written. To trigger alerts if a disk has a problem, I personally added a line like this for each disk:

`/dev/disk/by-id/ata-[...] -d jmb39x-q,N -a -m your.account@gmail.com -n standby`

where N is 0 to 3. The `-a` option will alert on errors or SMART attribute degradation, while `-n standby` avoids spinning up disks if they're stopped (in hibernation).

### Using `rsync` to sync your *cold storage*

As mentioned above, it's a good idea to have an off-site backup of your data, which you periodically sync with your server's pool. To do this, you can use the `rsync` command ([man](https://web.archive.org/web/20250509091025/https://linux.die.net/man/1/rsync)):

`rsync -a --delete --no-i-r --info=progress2 SOURCE/ DESTINATION/`

Options:

  - `-a`: archive mode. Enables recursion and preserves permissions, creation and modification dates, *etc.*
  - `--delete`: deletes from the destination any files not present in the source.
  - `--no-i-r`: disables incremental recursion, making the progress percentage display a bit more accurate.
  - `--info=progress2`: gives info on transfer progress (see [here](https://web.archive.org/web/20250403092628/https://unix.stackexchange.com/questions/215271/understanding-the-output-of-info-progress2-from-rsync/261139#261139)).
  - You could add the `-z` option to compress data, but it's not useful if you're transferring already-compressed data (*e.g.* `.jpg`, `.mp3`, `mp4`, *etc.*).

**WARNING:** be sure to add the trailing slashes! You might want to run the above command with the `-n` option first for a *dry-run*.

**Note:** for very long transfers, especially if you're using a remote machine to launch the transfer from an SSH terminal, you can use `tmux` so the transfer isn't interrupted if you lose connection. More info [here](https://web.archive.org/web/20250419034253/https://www.redhat.com/en/blog/introduction-tmux-linux).

## Setting up a Samba server

Having a storage server with a few dozen terabytes is great, but being able to access it from another machine is even better! To access your server as a network drive from your other machines, we'll use [Samba](https://fr.wikipedia.org/wiki/Samba_(informatique)), a protocol supported by Windows, MacOS, and Linux for maximum interoperability.

I won't detail all the steps here because there are [many](https://web.archive.org/web/20250510125753/https://documentation.ubuntu.com/server/how-to/samba/file-server/index.html) [guides](https://web.archive.org/web/20250510130158/https://shape.host/resources/installer-samba-sur-ubuntu-22-04-un-guide-complet) that do it very well online. Basically, follow these steps:

- Install Samba with `sudo apt-get install samba`
- Edit the config file `/etc/samba/smb.conf` to specify the share parameters you want, typically:

```
[Share_Name]
   comment = A comment
   path = /path/to/share
   writable = yes
   browsable = yes
```

- Add a user and password to Samba with `sudo smbpasswd -a my_user`. This user can be an existing user on the machine, or a new one created for the occasion with `sudo adduser my_user`
- Restart the Samba service with `sudo systemctl restart smbd`. Also check that the Samba server starts at boot with `sudo systemctl status smbd`, which should be `enable` (otherwise use `sudo systemctl enable smbd`).
- Don't forget to allow Samba in the Linux firewall with `sudo ufw allow samba`

To access your server from Windows, enter `\\ip.ip.ip.ip\Share_Name` using your server's IP in the explorer address bar, and enter the login for the user you added above.

**Notes:**

  - To see which users are added to Samba, run `sudo smbstatus`.
  - On Windows, feel free to mount your Samba server as a network drive, so it gets a drive letter (*e.g.* `Z:`).

## Setting up a VPN with OpenVPN

Setting up a VPN lets you securely access your server from anywhere in the world, which is pretty powerful. I highly recommend following [the doc](https://web.archive.org/web/20250505064141/https://openvpn.net/community-resources/how-to/) from OpenVPN, which details the steps. Start by installing the necessary tools with:

`sudo apt-get install openvpn openssl easy-rsa`

**Note:** the setup I describe below lets you securely connect to a virtual network including the server and other machines connected to it. This means that, from the client's point of view, the server will have an IP like `10.50.50.1`, the client will have `10.50.50.2`, and another machine connected to the server will have `10.50.50.3`. These three machines can then communicate as if they were on the same network. That's all this config does. In particular, **it does NOT mean** that all your internet traffic will go *via* a secure tunnel and exit through the server's internet connection (unlike the layman meaning of the term "VPN", and what services like NordVPN offer). You can configure OpenVPN to do that though, see [here](https://web.archive.org/web/20250505064141/https://openvpn.net/community-resources/how-to/#routing-all-client-traffic-including-web-traffic-through-the-vpn).

### Generating security keys

To connect to our VPN, we'll use security keys. The general idea is that both the client AND the server have their own set of keys, protected by password, and the keys are transmitted to clients via a secure channel (USB stick, or [encrypted with GPG](https://web.archive.org/web/20250511071055/https://stackoverflow.com/questions/55383876/encrypt-and-decrypt-files-with-password/55384047#55384047) if sent over the internet). Note that with this authentication scheme, you need to generate a new key for each new client you want to allow to connect to your VPN.

Start by finding where `easy-rsa` is installed (typically in `/usr/share/easy-rsa/`) and copy that directory to your home with something like:
`cp -r easy-rsa ~/easy-rsa`. Then do `cp vars.example vars` and edit `vars` to fill in the necessary fields, for example change the certificate expiration date and enter your personal info if you want. Otherwise, you can leave the fields blank like this:

```
set_var EASYRSA_REQ_COUNTRY	"."
set_var EASYRSA_REQ_PROVINCE	"."
set_var EASYRSA_REQ_CITY	"."
set_var EASYRSA_REQ_ORG	        "John Doe"
set_var EASYRSA_REQ_EMAIL	"thisemailadress@doesnot.exist"
set_var EASYRSA_REQ_OU		"."
set_var EASYRSA_CA_EXPIRE	365000
set_var EASYRSA_CERT_EXPIRE	36500
```

Then (from the folder with the `easy-rsa` script and the `vars` file you just edited):

```
./easyrsa clean-all
./easyrsa build-ca
./easyrsa build-server-full server-name
./easyrsa build-client-full client-name
./easyrsa gen-dh
```

Each `build` step will ask you to create a password:

  - The `ca` password, which you'll need if you want to use the authentication chain again (*e.g.* to add new clients).
  - The `server` password, used by the OpenVPN server to start its service.
  - The `client` password, used by the client to connect to the server.

Normally, all generated keys are in `easy-rsa/pki`. Then put:

```
ca.crt
server-name.crt
server-name.key
dh.pem
```

in `/var/local/openvpn` and do `chmod 400` on them. Finally, put

```
ca.crt
client-name.crt
client-name.key
```

somewhere (like `home/openvpn`) on your client machine. Be sure to securely transfer these certificates, even if the risk is limited thanks to the additional layer of password protection.

### A clean config file for OpenVPN

You can start from the sample config file [here](https://web.archive.org/web/20250425085619/https://github.com/OpenVPN/openvpn/blob/master/sample/sample-config-files/server.conf), modify it as you like, and put it in `/etc/openvpn/server`. Mine looks like this, keeping only the uncommented lines:

```
askpass pass
port 443
proto tcp
dev tun
ca /var/local/openvpn/ca.crt
cert /var/local/openvpn/server-name.crt
key /var/local/openvpn/server-name.key  # This file should be kept secret
dh /var/local/openvpn/dh.pem
topology subnet
server 10.50.50.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt
keepalive 10 120
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```

The important lines I changed are as follows:

  - `askpass pass`, so you don't have to enter the server password every time the service starts. `pass` is a 400-chmoded file next to the config file in `/etc/openvpn/server` and contains the `server` password defined above.
  - `port 443` and `proto tcp` make your VPN look like a [regular HTTPS connection](https://en.wikipedia.org/wiki/HTTPS#Technical). This lets you connect to your VPN from anywhere, while the default OpenVPN port (1194) is often blocked by public or corporate firewalls. **Note:** if you also want to host an HTTPS web server on the same machine, it's possible with `port-share`, but requires extra config, see [here](https://web.archive.org/web/20250318175242/https://memo-linux.com/openvpn-sur-le-port-443-partage-avec-un-serveur-web/).
  - The key files in `local` should also ideally be 400-chmoded.
  - `server 10.50.50.0 255.255.255.0` gives the subnet IP for your VPN. This means that for the client, the server will be at `10.50.50.1`. Typically, [the OpenVPN documentation](https://web.archive.org/web/20250505064141/https://openvpn.net/community-resources/how-to/#numbering-private-subnets) recommends an address somewhere in the middle of the `10.X.X.X` range.

**Note:** you'll find [references](https://web.archive.org/web/20240822022408/https://www.reddit.com/r/OpenVPN/comments/y4jdvy/configuring_the_openvpn_on_tcp_or_udp/?rdt=62515) and [references](https://web.archive.org/web/20250320043004/https://www.reddit.com/r/VPN/comments/41pky3/udp_vs_tcp/) online saying TCP is slower than UDP and that UDP should be preferred for OpenVPN. It seems that's not necessarily true in practice, see [here](https://web.archive.org/web/20250217062459/https://www.fastvue.co/sophos/blog/testing-sophos-ssl-vpn-performance-udp-or-tcp/) and [here](https://web.archive.org/web/20241213215156/https://www.top10vpn.com/fr/guides/udp-vs-tcp/). Personally, from a remote network, I had exactly the same bandwidth (about 100 Mbps) in TCP and UDP. So I chose TCP to be sure that I could access my VPN from any network, especially since I plan to use my VPN as a way to administer my server remotely, and not to transfer large amounts of data.

### Opening the right ports

To make your VPN accessible, you need to open the corresponding port in your firewall. You can see which ports are currently opened with
`sudo ufw status` and add a port and protocol with `sudo ufw allow portnumber/protocol`. In our case, do `sudo ufw allow 443/tcp` if it's not already open.

### Setting up port-forwarding on your internet box

This step is about forwarding incoming connections on your public IP to your server. Typically, this means that connections coming from outside (the internet) to IP `156.124.87.35:443` (where `156.124.87.35` is your public IP) will be forwarded by your internet box to the local IP `192.168.0.1:443`, where `192.168.0.1` is your server's local IP.

The setup depends on your box, but generally involves three steps:

  - If your internet service provider (ISP) allows it, you can ask for a static IP. This isn't really related to port-forwarding *per se*, but will make client connection easier.
  - Assign your server a static local IP.
  - Set up a [NAT rule](https://en.wikipedia.org/wiki/Network_address_translation) to forward external requests to your public IP and port 443 TCP to your server's fixed local IP (on port 443 and TCP).

### Starting the VPN automatically with the system

OpenVPN automatically starts all servers it finds in `/etc/openvpn/server`, i.e., all files in that folder ending with `.conf`. In theory, you just need to run: `sudo systemctl start openvpn-server@server-name` and the same with `start` replaced by `enable` to make it start by default. The name after the @ is the `.conf` file containing the server config. To check if the server started OK, `systemctl status openvpn-server@server-name` should return the statuses `enable` and `active (running)`.

### Connecting from the client side

On the client side, we'll assume a Unix command line (Linux or MacOS; this will also work with [WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) on Windows). Otherwise [GUI clients](https://web.archive.org/web/20250510193829/https://openvpn.net/client/) also exist, but I haven't tested them. After installing OpenVPN, place the following files in the same folder after 400-chmoding them:

```
ca.crt
client.conf
client-name.crt
client-name.key
```

where the `client.conf` file contains the following lines:

```
client
dev tun
proto tcp
remote your.public.ip.address 443
resolv-retry infinite
nobind
persist-key
persist-tun
ca /path/to/the/ca.crt
cert /path/to/the/client-name.crt
key /path/to/the/client-name.key
remote-cert-tls server
verb 3
```

To connect to your VPN, simply run:

```
sudo openvpn /path/to/client.conf
```

This should prompt you, in addition to your admin password, for the `client` password defined when generating the key for this particular client. If all goes well, you should see a log line containing `Initialization Sequence Completed`. You can now access your server at `10.50.50.1`. If this works, congratulations! Your VPN is fully functional.

### Bonus: measuring your VPN bandwidth

If you want to measure the bandwidth between your client and server, you can use the `iperf` tool. On both your client and server, install `iperf` with:

```
sudo apt-get install iperf  # On Linux
brew install iperf          # On MacOS
```

Then, on the server side, run `iperf -s` to start an iperf server. By default, the server listens on port 5001, so remember to open this port with `sudo ufw allow 5001/tcp`. On the client side, after connecting the VPN, run `iperf -c 10.50.50.1 -r`, wherein `-c` specifies the server's IP and the `-r` flag runs a symmetric bandwidth test (client to server, then server to client).

## Serving Jellyfin on the Internet

This section explains how to set up a [Jellyfin](https://web.archive.org/web/20250509030531/https://jellyfin.org/docs/) media server, which offers an interface similar to most existing Video-On-Demand services. We'll then see how to make this server accessible from the internet securely using Cloudflare tunnels.

### Installation

It's quite straightforward, just follow the installation procedure detailed [here](https://web.archive.org/web/20250508172635/https://jellyfin.org/docs/general/installation/linux/):

```
curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

Normally the service starts automatically, and you can check it with `systemctl status jellyfin` (otherwise run `sudo systemctl enable jellyfin` and `sudo systemctl restart jellyfin`). It's served on port 8096 locally, which means you can access the server from a web browser: either at `localhost:8096` on the server itself, or at `the.server.local.ip:8096` from another machine on the same local network (as long as port 8096 has been opened on the server with `sudo ufw allow 8096/tcp`). On first launch, an interface guides you through creating an admin account. You can also create secondary users, add folders to scan for organizing your collections, *etc.* See the [documentation](https://web.archive.org/web/20250509030531/https://jellyfin.org/docs/).

### Backup

Typically done at the same time as your *cold storage* backup, you can back up your Jellyfin configuration at any time to keep your settings, users, collections, *etc.*, backed up and safe. To do this, follow the [documentation](https://web.archive.org/web/20250426045259/https://jellyfin.org/docs/general/administration/backup-and-restore/).

### Getting a domain name

Now that your Jellyfin server is running on the local network, let's see how to serve it on the internet. There are several solutions for this, the classic approach being to configure everything yourself, with port opening and forwarding, DNS, a reverse proxy, *etc.* (see [here](https://web.archive.org/web/20250511122939/https://lbrito.ca/blog/2020/06/free_https_home_server.html)).

If you use NordVPN, they also offer a turnkey solution with Meshnet, see [here](https://meshnet.nordvpn.com/how-to/remote-files-media-access/access-jellyfin-media-sever-remotely). There are also [several other solutions](https://web.archive.org/web/20250414090813/https://dev.to/jagkush/a-quick-way-to-access-your-local-server-on-the-internet-4kei) like localhost.run, Ngrok, or Cloudflare tunnels. I chose the latter mainly because it's free, lets you link the tunnel to a domain name that you own (which free accounts on localhost or Ngrok don't), and is relatively simple to set up.

First, you'll need a domain name to serve your website. There are [plenty of providers](https://web.archive.org/web/20250504012124/https://tld-list.com/), and prices range from about 1 to 15€/month depending on the extension you choose. I personally chose Infomaniak, which offers a `.fr` domain for among the lowest prices (5€ the first year, 7€/year renewal), is based in Switzerland, and has a [nice ecological stance](https://web.archive.org/web/20250417215405/https://www.infomaniak.com/fr/ecologie). Note that **buying a domain from Infomaniak automatically gives you 10 MB of web hosting for a static site**. For your information, this website uses that hosting! More details [here](about/this_site). You can also enable HTTPS with Let's Encrypt in your site's settings with a single click.

### Routing a Cloudflare tunnel

Once you have your domain, you can add it to your (free) Cloudflare account. To do this, follow [this guide](https://medium.com/@fabrice_/setting-up-a-media-server-jellyfin-and-making-it-securely-accessible-from-anywhere-in-the-world-ca3b4d9dd19e), and the [Cloudflare doc](https://web.archive.org/web/20250511124347/https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/local-management/create-local-tunnel/), which is very well written.

In short, after adding your domain to Cloudflare (following the step-by-step tutorial in the Cloudflare interface), install `cloudflared` on your server:

```
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt-get update && sudo apt-get install cloudflared
```

Configure it with (from `~/`):

```
cloudflared tunnel login
cloudflared tunnel create jellyfin_tunnel
cloudflared tunnel list
touch .cloudflared/config.yml
nano .cloudflared/config.yml
cloudflared tunnel list
```

And put this into the `config.yml` file (`a1****z0` is the UUID of the tunnel you created):

```
url: http://localhost:8096
tunnel: a1****z0
credentials-file: /home/youruser/.cloudflared/a1****z0.json
```

The command `cloudflared tunnel list` should list your tunnel. Then, to route your `jellyfin_tunnel` to a subdomain of your main domain:

```
cloudflared tunnel route dns jellyfin_tunnel jellyfin.yourdomain.com
cloudflared tunnel run jellyfin_tunnel
```

I chose [jellyfin.edervieux.fr](https://jellyfin.edervieux.fr/), but you can use any subdomain. You can also use Cloudflare tunnels to serve other local web services on your main public domain!

### Starting the tunnel automatically

Normally, to avoid having to run `cloudflared tunnel run jellyfin_tunnel` after every reboot, you should be able to add a service to `systemd` as we did earlier for Jellyfin or OpenVPN. The procedure is detailed [here](https://web.archive.org/web/20250511125505/https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/local-management/as-a-service/linux/), but seems to be broken at the time being, see [here](https://github.com/cloudflare/cloudflared/issues/1457).

**Workaround:** In the meantime, you can start the tunnel automatically by creating your own service with [`systemd`](https://web.archive.org/web/20250304090858/https://manpages.ubuntu.com/manpages/focal/man5/systemd.service.5.html). You can't use [`crontab`](https://web.archive.org/web/20250505211623/https://man7.org/linux/man-pages/man5/crontab.5.html) because the `cloudflared` command doesn't exit and keeps the tunnel open. To do this, create the file `/etc/systemd/system/clouded_jelly.service` with the following lines (see [here](https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6)):

```
[Unit]
Description=Cloudflare tunneling for Jellyfin
After=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=5
User=your_username
ExecStart=cloudflared tunnel run jellyfin_tunnel

[Install]
WantedBy=multi-user.target
```

You can test that the service is configured correctly by starting it with `sudo systemctl restart clouded_jelly`, and enable it at boot with `sudo systemctl enable clouded_jelly`.

# Conclusion

*That's all folks!* If you notice any inaccuracies in this guide, please let me know via the [contact form](contacts). Likewise, if you find any paragraphs unclear or lacking in external resources to be truly standalone, I'll try to keep this guide up to date and improve it based on your feedback.

Good luck.
