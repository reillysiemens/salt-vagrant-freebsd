# Salt Vagrant Demo with FreeBSD 11.0-RELEASE

This is a modification of [UtahDave][UtahDave]'s
[Salt Vagrant demo][Salt Vagrant demo] for FreeBSD 11.0-RELEASE. This will
probably work with FreeBSD 10 and above. Here is a
[full list of official FreeBSD Vagrant boxes][FreeBSD Vagrant boxes].

## Prerequisites
- Git
- Vagrant
- VirtualBox

## Instructions
Run the following commands in a terminal.

```bash
git clone https://github.com/reillysiemens/salt-vagrant-freebsd.git
cd salt-vagrant-freebsd
vagrant up
```

This creates a FreeBSD Salt master and two FreeBSD Salt minions. The master
is pre-seeded with the minion's keys. Be sure to replace these keys in
production.

You can then run these commands.

```bash
vagrant ssh
sudo salt \* test.ping
sudo salt \* state.apply
```

## Troubleshooting
The official FreeBSD Vagrant box uses `UseDNS yes` in its
`/etc/ssh/sshd_config`, which can cause `vagrant ssh` to timeout. If this
happens you may need to manually connect to the VM and set the hostname
```
vagrant ssh minion1
sudo hostname saltminion1.local
```
otherwise the Salt bootstrapping is likely to fail due to the missing `fqdn`
pillar.

[UtahDave]: https://github.com/UtahDave/
[Salt Vagrant demo]: https://github.com/UtahDave/salt-vagrant-demo
[FreeBSD Vagrant boxes]: https://app.vagrantup.com/freebsd
