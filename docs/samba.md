# Enabling SAMBA

PiBuilder can enable SMB services as an option. 

## Installing SAMBA

PiBuilder assumes that you have a working configuration that you want to preserve across rebuilds. If you do not have a working configuration, you need to do that first. You may find the following tutorials helpful:

* KaliTut [Samba on Raspberry Pi Guide – A To Z](https://kalitut.com/samba-on-raspberry-pi/) (April 2021)
* PiMyLifeUp [Raspberry Pi SAMBA](https://pimylifeup.com/raspberry-pi-samba/) (Feb 2021)
* JUANMTECH [SAMBA file sharing](https://www.juanmtech.com/samba-file-sharing-raspberry-pi/) (Oct 2017)

Note:

* Tutorials differ in the packages they tell you to install. You only need:

	```bash
	$ sudo apt install -y samba smbclient
	```

	The `samba` package *includes* `samba-common` and `samba-common-bin` so you do not need to install those separately.

## Enabling remote-mount of home directory

If you want to remote mount your Raspberry Pi's home directory onto another machine, the simplest approach is:

1. Use `sudo` and edit `/etc/samba/smb.conf`
2. Find the following:

	``` ini
	[homes]
	   comment = Home Directories
	   browseable = no
	
	# By default, the home directories are exported read-only. Change the
	# next parameter to 'no' if you want to be able to write to them.
	   read only = yes
	```
	
3. Make it look like this:

	``` ini
	[homes]
	   comment = Home Directories
	   browseable = yes
	
	# By default, the home directories are exported read-only. Change the
	# next parameter to 'no' if you want to be able to write to them.
	   read only = no
	```
	
4. Restart the service:

	``` bash
	$ sudo service smbd restart
	```

## Adding to PiBuilder

***After*** you have SAMBA working on your Raspberry Pi, you need to preserve three files:

1. Your configuration:

	```bash
	$ cp /etc/samba/smb.conf $HOME
	```

2. Any SAMBA credentials you may have set up:

	```bash
	$ touch $HOME/passdb.tdb
	$ sudo cp /var/lib/samba/private/passdb.tdb $HOME/passdb.tdb
	```

3. Host-specific information generated when SAMBA is first installed on any given host:

	```bash
	$ touch $HOME/secrets.tdb@$HOSTNAME
	$ sudo cp /var/lib/samba/private/secrets.tdb $HOME/secrets.tdb@$HOSTNAME
	```

	`@HOSTNAME` syntax is used because `secrets.tdb` contains *host-specific* information. While you may use common `smb.conf` and `passdb.tdb` files on several hosts, you should obtain `secrets.tdb` from the host on which it was created. 

Next, navigate to the top level of your copy of PiBuilder and create two directories:

```bash
$ cd ~/PiBuilder
$ mkdir -p boot/scripts/support/etc/samba boot/scripts/support/var/lib/samba/private
```

Finally:

1. Move `smb.conf` into `~/PiBuilder/boot/scripts/support/etc/samba`; and
2. Move the `.tdb` files into `~/PiBuilderboot/scripts/support/var/lib/samba/private`.

PiBuilder detects the presence of `smb.conf` and uses it as a trigger to:

1. Install SAMBA;
2. Replace the default versions of the three files with your custom versions; and
3. Create `$HOME/share` as a home for your SMB mount points.

## Caution

The `smb.conf` file can change across OS versions (eg Bullseye to Bookworm).

If you are planning to upgrade, is a good idea to install SAMBA and configure it by hand.
