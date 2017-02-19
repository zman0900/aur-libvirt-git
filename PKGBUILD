# Maintainer: Arthur Borsboom <arthurborsboom@gmail.com>
# Contributor: Jonas Heinrich <onny@project-insanity.org>
# Contributor: Sergej Pupykin <pupykin.s+arch@gmail.com>
# Contributor: Jonathan Wiersma <archaur at jonw dot org>
# Contributor: Dan Ziemba <zman0900@gmail.com>

pkgname=libvirt-git
pkgver=3.0.0.212.gd3ffa0ece
pkgrel=1
pkgdesc="API for controlling virtualization engines (openvz,kvm,qemu,virtualbox,xen,etc)"
arch=('i686' 'x86_64')
url="http://libvirt.org/"
license=('LGPL')
depends=('e2fsprogs' 'gnutls' 'iptables' 'libxml2' 'parted' 'polkit' 'python2'
	 'avahi' 'yajl' 'libpciaccess' 'udev' 'dbus' 'libxau' 'libxdmcp' 'libpcap' 'libcap-ng'
	 'curl' 'libsasl' 'libgcrypt' 'libgpg-error' 'openssl' 'libxcb' 'gcc-libs'
	 'iproute2' 'libnl' 'libx11' 'numactl' 'gettext' 'ceph' 'libssh2' 'netcf')
makedepends=('git' 'pkgconfig' 'lvm2' 'linux-api-headers' 'dnsmasq' 'lxc'
             'libiscsi' 'open-iscsi'
             'perl-xml-xpath' 'libxslt' 'qemu')
optdepends=('ebtables: required for default NAT networking'
	    'dnsmasq: required for default NAT/DHCP for guests'
	    'bridge-utils: for bridged networking'
	    'openbsd-netcat: for remote management over ssh'
	    'qemu'
	    'radvd'
	    'dmidecode')
conflicts=('libvirt')
provides=('libvirt')
options=('emptydirs')
backup=('etc/conf.d/libvirt-guests'
	'etc/conf.d/libvirtd'
	'etc/libvirt/libvirt.conf'
    'etc/libvirt/virtlogd.conf'
	'etc/libvirt/libvirtd.conf'
	'etc/libvirt/lxc.conf'
	'etc/libvirt/nwfilter/allow-arp.xml'
	'etc/libvirt/nwfilter/allow-dhcp-server.xml'
	'etc/libvirt/nwfilter/allow-dhcp.xml'
	'etc/libvirt/nwfilter/allow-incoming-ipv4.xml'
	'etc/libvirt/nwfilter/allow-ipv4.xml'
	'etc/libvirt/nwfilter/clean-traffic.xml'
	'etc/libvirt/nwfilter/no-arp-ip-spoofing.xml'
	'etc/libvirt/nwfilter/no-arp-mac-spoofing.xml'
	'etc/libvirt/nwfilter/no-arp-spoofing.xml'
	'etc/libvirt/nwfilter/no-ip-multicast.xml'
	'etc/libvirt/nwfilter/no-ip-spoofing.xml'
	'etc/libvirt/nwfilter/no-mac-broadcast.xml'
	'etc/libvirt/nwfilter/no-mac-spoofing.xml'
	'etc/libvirt/nwfilter/no-other-l2-traffic.xml'
	'etc/libvirt/nwfilter/no-other-rarp-traffic.xml'
	'etc/libvirt/nwfilter/qemu-announce-self-rarp.xml'
	'etc/libvirt/nwfilter/qemu-announce-self.xml'
	'etc/libvirt/qemu-lockd.conf'
	'etc/libvirt/qemu.conf'
	'etc/libvirt/qemu/networks/autostart/default.xml'
	'etc/libvirt/qemu/networks/default.xml'
	'etc/libvirt/virt-login-shell.conf'
	'etc/libvirt/virtlockd.conf'
	'etc/logrotate.d/libvirtd'
	'etc/logrotate.d/libvirtd.lxc'
	'etc/logrotate.d/libvirtd.qemu'
	'etc/logrotate.d/libvirtd.uml'
	'etc/sasl2/libvirt.conf')
install="libvirt.install"
source=('git+git://libvirt.org/libvirt.git'
	libvirtd.conf.d
	libvirtd-guests.conf.d
	libvirt.tmpfiles.d)
sha256sums=('SKIP'
            '9d0597bbf2bd7892420cebaf0563236fe1483b83ae95ee6263c1ce7f44a44134'
            '0896c30100e9e40aee1eb4a2cf0cac2c0bdd5fd7b077b9d2680d90e77435ea66'
            '5c26353833944db8dc97aa63843734519d6521bd8d88497d94d910ee9d3169d8')

pkgver() {
  cd "$SRCDEST/libvirt"
  git describe --always | sed 's|-|.|g' | sed 's/^.//'
}

prepare() {
  cd "$srcdir/libvirt"

  for file in $(find . -name '*.py' -print); do
    sed -i 's_#!.*/usr/bin/python_#!/usr/bin/python2_' $file
    sed -i 's_#!.*/usr/bin/env.*python_#!/usr/bin/env python2_' $file
  done

  sed -i 's|/sysconfig/|/conf.d/|g' \
    daemon/libvirtd.service.in \
    tools/{libvirt-guests.service,libvirt-guests.sh,virt-pki-validate}.in \
    src/locking/virtlockd.service.in
  sed -i 's|@sbindir@|/usr/bin|g' src/locking/virtlockd.service.in
  # 78 is kvm group: https://wiki.archlinux.org/index.php/DeveloperWiki:UID_/_GID_Database
  sed -i 's|#group =.*|group="78"|' src/qemu/qemu.conf
  sed -i 's|/usr/libexec/qemu-bridge-helper|/usr/lib/qemu/qemu-bridge-helper|g' \
    src/qemu/qemu{.conf,_conf.c} \
    src/qemu/test_libvirtd_qemu.aug.in
  
  sed -i 's/notify/simple/' daemon/libvirtd.service.in
}

build() {
  cd "$srcdir/libvirt"

  export PYTHON=`which python2`
  export LDFLAGS=-lX11
  export RADVD=/usr/bin/radvd
  NOCONFIGURE=1 ./autogen.sh 
  sed -i 's|libsystemd-daemon|libsystemd|g' configure
  ./configure --prefix=/usr --libexec=/usr/lib/"${pkgname/-git/}" --sbindir=/usr/bin \
      --with-storage-lvm --without-xen --with-udev --without-hal --disable-static \
      --with-init-script=systemd \
      --with-qemu-user=nobody --with-qemu-group=nobody \
      --with-netcf --with-interface --with-lxc --with-storage-iscsi
  make
}

package() {
  cd "$srcdir/libvirt"
  make DESTDIR="$pkgdir" install

  install -D -m644 "$srcdir"/libvirtd.conf.d "$pkgdir"/etc/conf.d/libvirtd
  install -D -m644 "$srcdir"/libvirtd-guests.conf.d "$pkgdir"/etc/conf.d/libvirt-guests
  install -D -m644 "$srcdir"/libvirt.tmpfiles.d "$pkgdir"/usr/lib/tmpfiles.d/libvirt.conf

  chown -R 0:78 "$pkgdir"/var/lib/libvirt/qemu
  chmod 0770 "$pkgdir"/var/lib/libvirt/qemu

  chown 0:102 "$pkgdir"/usr/share/polkit-1/rules.d
  chmod 0750 "$pkgdir"/usr/share/polkit-1/rules.d

  rm -rf \
	"$pkgdir"/var/run \
	"$pkgdir"/etc/sysconfig \
	"$pkgdir"/etc/rc.d
}

