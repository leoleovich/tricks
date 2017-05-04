**There are some tips and tricks which I use**

# Unix utils tricks
## sed

Replace \n in any system
```bash
sed -e ':a;N;$!ba;s/\n//g;s/0d0a/\n/g'
```

## find
Find all files changed in the interval
```
find ./ -newermt 2015-07-20 ! -newermt 2015-07-28
```

## iptables

Block every outgoing connection
```
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -j REJECT
```

## ssh
### Forward to localhost
```
ssh -v -L 127.0.0.1:1527:127.0.0.1:1527 server
```
### Forward through one more server
```
ssh -L 8080:real_server:8080 control
```

# Packages
## Build

Setup build environment
```bash
python-all-dev fakeroot unzip python-stdeb dh-python devscripts
```

Pack the package:
```bash
dpkg-deb -b ${PACKAGEDIR} ${WORKDIR}
```
Rebuild debian package
```bash
dpkg-source -x <package>.dsc
dpkg-buildpackage -rfakeroot -uc -us -sa -j3
```

Rebuld pip to deb:
```bash
package='Flask-Profile-0.2'
py2dsc -m 'Oleg Obleukhov <leoleovich@something>' ./${package}.tar.gz && cd deb_dist/${package} && DEB_BUILD_OPTIONS=nocheck debuild -i -us -uc -b
```

## Installation

Check the source of package
```bash
apt-cache policy nginx
```

# Web servers
## Htpasswd
Generate crypt password for htpasswd
```bash
openssl passwd -crypt 12345
```

# Network
## Routes

Change default route
```bash
ip r change default via 10.0.0.240 dev eth0
```
## Tshark

Analyse pcap
```bash
tshark -T fields -e ip.src -r web03.out.pcap  | sort | uniq -c
```

# Filesystem
## Permissions

Fully copy permissions from one directory to another
```bash
getfacl -a it | setfacl -d -M- it-ro/
```

# Concurrent Versions Systems
## Git

Config to rebase branch in master
```bash
git config branch.master.rebase true
git config —global branch.autosetuprebase always
```

Go back to the commmit (it will stay)
```bash
git reset --hard b75b34e 
git push -f
```

Update commint message before push
```bash
git ci --amend
```

# Databases
## Mysql

Backup from live DB using maximum compression
```bash
working_master$ innobackupex --slave-info --safe-slave-backup --parallel=8 --compress --compress-threads=8 --stream=xbstream ./ | ssh -o StrictHostKeyChecking=no <broken_slave> "xbstream -xC /var/lib/mysql/"
broken_slave$ find /var/lib/mysql -name '*.qp' | while read f ; do (qpress -dv $f $(dirname $f) && rm $f)&  done
```

## Postgres

Kill old/slow query
```bash
select * from pg_stat_activity order BY xact_start limit 10;
select pg_terminate_backend(16796);
```

Get list of tables 
```
psql sonar -tXAqc "select relname from pg_catalog.pg_class where relkind = 'r' and relnamespace=2200"  | cat
```

## Rabbitmq

Clear policies
```
rabbitmqctl list_vhosts -q | while read q ; do rabbitmqctl list_policies -q -p $q | while read p ; do echo "${p}" | awk '{print $2}' | while read c ; do rabbitmqctl clear_policy -p $q "${c}" ; done ; done ; done
```
# Virtualization
# KVM

```bash
virsh migrate --live --copy-storage-all --persistent --undefinesource --change-protection --auto-converge --domain <VM> --abort-on-error --desturi qemu+ssh://<HV address>/system --timeout 600
```
