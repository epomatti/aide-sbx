# AIDE

Advanced Intrusion Detection Environment (AIDE) sandbox.

> [!TIP]
> Commands to install and configure AIDE were referenced from [CIS Benchmarks](https://downloads.cisecurity.org/#/)

## Install

### Start a Linux VM

This example will use VirtualBox. Use the [VagrantFile](./VagrantFile) template to start your box:

```sh
vagrant up
```

### Install AIDE packages

Install AIDE:

```sh
# Upgrade all packages
sudo apt update && sudo apt upgrade -y

# Install AIDE
sudo apt install -y aide aide-common
```

You'll be prompted to configured Postfix. Selecting `No configuration` will do it for now.

<img src=".assets/postfix.png" />


### Initiate AIDE

Initiate AIDE:

```sh
sudo aideinit
```

Check the output:

```
$ sudo ls -l /var/lib/aide/
total 48576
-rw------- 1 root root 24868906 Apr 17 17:14 aide.db
-rw------- 1 root root 24868906 Apr 17 17:14 aide.db.new
```

Copy the new DB over the baseline:

> [!TIP]
> You may choose to `cp` or `mv`

```sh
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

### Running AIDE

Command to run AIDE manually:

```sh
sudo aide --check --config=/etc/aide/aide.conf
```

## Configuration

> [!TIP]
> Detailed configuration can be found in the manuals: [aide.conf (Ubuntu)](https://manpages.ubuntu.com/manpages/jammy/man5/aide.conf.5.html),[ aide.conf (Debian)](https://manpages.debian.org/bookworm/aide/aide.conf.5.en.html), and further details can be obtained in the [manual](https://aide.github.io/doc/)

Edit the configuration file:

```
/etc/aide/aide.conf
```

The default configuration will include several files from `/etc/aide/aide.conf.d`.

As a sample for manual configuration:

> [!TIP]
> Use `$` to indicate a file

```conf
@@x_include_setenv PATH /bin:/usr/bin
#@@x_include /etc/aide/aide.conf.d ^[a-zA-Z0-9_-]+$

/opt R
/etc R
```

Once changes have been made, reapply them:

```
sudo aide --init --config=/etc/aide/aide.conf
```

It's likely that the database must be copied again:

```sh
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```


### Run Daily

A simple way of running AIDE automatically is using `cron.daily`:

```sh
sudo cp /usr/share/doc/aide/examples/aide.cron.daily /etc/cron.daily/aide
sudo chmod +x /etc/cron.daily/aide
```

To disable the service, there are [options](https://serverfault.com/q/150348/560797). There are caveats.

This will show which scripts are run without executing them:

```sh
sudo run-parts --test /etc/cron.daily
```

## Other

### Verify AIDE Installation

You can run the following commands to verify if AIDE is installed:

```sh
dpkg-query -s aide &>/dev/null && echo "aide is installed"
# aide is installed

dpkg-query -s aide-common &>/dev/null && echo "aide-common is installed"
# aide-common is installed
```

### Troubleshooting

Logs are available in varying locations:

```sh
sudo journalctl | grep aide
sudo grep -i aide /var/log/syslog
sudo grep -i cron /var/log/syslog
sudo ls -l /var/log/aide
```
