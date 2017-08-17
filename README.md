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

## bash
Be sure the line contains at the end
```
oleg="hello.ending"
echo ${^${oleg}%%.ending}".ending"
hello.ending
oleg="hello"
echo ${^${oleg}%%.ending}".ending"
hello.ending
```

## iptables

Block every outgoing connection
```
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -j REJECT
```

Redirect everything from specific host to another port on localhost
```
iptables -t nat -A PREROUTING -s <from>/32 -p tcp --dport 80 -j DNAT --to-destination <to>:42424
```
if you are using localhost ip as destination, you must enable `sysctl -w net.ipv4.conf.eth0.route_localnet=1` or make it proper:
```
iptables -t nat -A PREROUTING -s <from>/32 -p tcp --dport 80 -j REDIRECT --to-port 42424
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

## Apply patch
patch -p1 --dry-run < debian/patches/02_socket_maxconn.patch

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

## Rebuild from source
wget http://http.debian.net/debian/pool/main/d/dbus/dbus_${SHORT_VERSION}.orig.tar.gz -O ${WORKDIR}/dbus.orig.tar.gz
wget http://http.debian.net/debian/pool/main/d/dbus/dbus_${version}.debian.tar.xz -O ${WORKDIR}/dbus.debian.tar.xz
tar -xzf dbus.orig.tar.gz
tar -xf dbus.debian.tar.xz -C dbus-${SHORT_VERSION}
# update changelog
cd ${WORKDIR}/dbus-${SHORT_VERSION}
debuild -b -uc -us

## Installation

Check the source of package
```bash
apt-cache policy nginx
```

# Web servers
## Htpasswd
Generate crypt password for htpasswd
```bash
openssl passwd -apr1 12345 # md5
openssl passwd -crypt 12345 # crypt (only up to 8 chars will be there)
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
## Tcpdump
Top hosts connecting
```bash
tcpdump -p -r ddos.pcap | awk '{print $3}' | sort | uniq -c | sort -n
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
