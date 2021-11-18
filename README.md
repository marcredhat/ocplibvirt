


description: Dell PowerEdge R440, Xeon(R) Silver 4114 CPU @ 2.20GHz, 20 cores, 128GB Ram, 4-2TB disks
ssh mchisinevski@vb1237.mydomain.com


[root@vb1237 mchisinevski]# cat /etc/centos-release
CentOS Linux release 8.2.2004 (Core)



[mchisinevski@vb1237 ~]$ cd /
[mchisinevski@vb1237 /]$ exec sudo su
[root@vb1237 /]# fuser -kim  /dev/cl_vb1237/home
[root@vb1237 /]# umount -f /dev/cl_vb1237/home
[root@vb1237 /]# lvremove /dev/cl_vb1237/home
Do you really want to remove active logical volume cl_vb1237/home? [y/n]: y
  Logical volume "home" successfully removed

  [root@vb1237 /]# lvextend -An -l +100%FREE -r /dev/cl_vb1237/root
    Size of logical volume cl_vb1237/root changed from 50.00 GiB (12800 extents) to 1.81 TiB (475651 extents).
    WARNING: This metadata update is NOT backed up.
    Logical volume cl_vb1237/root successfully resized.
  meta-data=/dev/mapper/cl_vb1237-root isize=512    agcount=4, agsize=3276800 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=1, sparse=1, rmapbt=0
           =                       reflink=1
  data     =                       bsize=4096   blocks=13107200, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
  log      =internal log           bsize=4096   blocks=6400, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0
  data blocks changed from 13107200 to 487066624


Append below lines in /etc/sysctl.conf:

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

sysctl -p

Add the AddressFamily line to sshd_config :

# vi /etc/ssh/sshd_config
....
AddressFamily inet
....
Restart sshd for changes to get effect :

# systemctl restart sshd


copy the pull secret from https://cloud.redhat.com/openshift/install/pull-secret to /root/pull-secret

yum update libgcrypt
(because https://bugzilla.redhat.com/show_bug.cgi?id=1925029)

dnf -y module install virt
dnf -y install zip podman buildah skopeo libvirt*  bind-utils wget tar gcc python3-devel python3  xauth virt-install virt-viewer virt-manager libguestfs-tools-c libguestfs-tools tmux httpd-tools git x3270-x11 nc net-tools
systemctl start libvirtd.service
systemctl enable libvirtd



systemctl status libvirtd
[root@vb1237 mchisinevski]# systemctl status libvirtd
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-11-18 10:25:52 PST; 2s ago
     Docs: man:libvirtd(8)
           https://libvirt.org
 Main PID: 33147 (libvirtd)
    Tasks: 19 (limit: 32768)
   Memory: 24.3M
   CGroup: /system.slice/libvirtd.service
           ├─33147 /usr/sbin/libvirtd --timeout 120
           ├─33341 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           └─33342 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper

Nov 18 10:25:54 vb1237.mydomain.com dnsmasq[33341]: started, version 2.79 cachesize 150
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq[33341]: compile time options: IPv6 GNU-getopt DBus no-i18n IDN2 DHCP DHCPv6 no-Lua TFTP no-conntrack ipset auth DNSS>
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq-dhcp[33341]: DHCP, IP range 192.168.122.2 -- 192.168.122.254, lease time 1h
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq-dhcp[33341]: DHCP, sockets bound exclusively to interface virbr0
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq[33341]: reading /etc/resolv.conf
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq[33341]: using nameserver 172.17.64.15#53
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq[33341]: using nameserver 172.21.64.15#53
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq[33341]: read /etc/hosts - 2 addresses
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq[33341]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 addresses
Nov 18 10:25:54 vb1237.mydomain.com dnsmasq-dhcp[33341]: read /var/lib/libvirt/dnsmasq/default.hostsfile


As root, setup a separate dnsmasq server on the host:
fuser -k 53/tcp
dnf  -y install dnsmasq
for x in $(virsh net-list --name); do virsh net-info $x | awk '/Bridge:/{print "except-interface="$2}'; done > /etc/dnsmasq.d/except-interfaces.conf
sed -i '/^nameserver/i nameserver 127.0.0.1' /etc/resolv.conf
systemctl restart dnsmasq
systemctl enable dnsmasq

[root@vb1237 mchisinevski]# systemctl status  dnsmasq
● dnsmasq.service - DNS caching server.
   Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-11-18 10:27:01 PST; 2s ago
 Main PID: 33481 (dnsmasq)
    Tasks: 1 (limit: 820844)
   Memory: 720.0K
   CGroup: /system.slice/dnsmasq.service
           └─33481 /usr/sbin/dnsmasq -k

Nov 18 10:27:01 vb1237.mydomain.com systemd[1]: Started DNS caching server..
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: started, version 2.79 cachesize 150
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: compile time options: IPv6 GNU-getopt DBus no-i18n IDN2 DHCP DHCPv6 no-Lua TFTP no-conntrack ipset auth DNSS>
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: reading /etc/resolv.conf
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: ignoring nameserver 127.0.0.1 - local interface
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: using nameserver 172.17.64.15#53
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: ignoring nameserver 127.0.0.1 - local interface
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: using nameserver 172.21.64.15#53
Nov 18 10:27:01 vb1237.mydomain.com dnsmasq[33481]: read /etc/hosts - 2 addresses


git clone https://github.com/kxr/ocp4_setup_upi_kvm.git
cd ocp4_setup_upi_kvm/


[root@vb1237 ocp4_setup_upi_kvm]# vim .defaults.sh
Set
# -z, --dns-dir DIR
export DNS_DIR="/etc/dnsmasq.d"

Adjust default as needed (no of master, no of workers, CPU, mem)



[root@vb1237 ocp4_setup_upi_kvm]# systemctl stop NetworkManager
[root@vb1237 ocp4_setup_upi_kvm]# systemctl disable NetworkManager

cat /etc/resolv.conf
nameserver 127.0.0.1


./ocp4_setup_upi_kvm.sh --ocp-version 4.7.stable -y


