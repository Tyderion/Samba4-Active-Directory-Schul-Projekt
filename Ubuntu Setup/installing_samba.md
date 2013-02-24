# Installing Samba 4 with Active Directory capabilities

## Ubuntu

### Setup
#### Install needed libraries
Let's install the needed development libraries to build samba 4
  - acl -- Required for a successful AD DC deployment. If this library is not included, samba will build successfully, however you will not be able to change ACL's from the windows frontend. You will receive and error when you provision and if you manually create the smb.conf with +s3fs, you will get Access is denied. from windows on any attempt to change ACL's.
  - xattr
  - blkid
  - gnutls
  - readline
  - openldap -- Required to build the Samba3 components with LDAP support. Lacking this library the build will complete but attempts to provision (via upgrade) an Active Directory domain from an existing Samba3 LDAP backend will fail. Also see samba-tool domain classicupgrade
  - cups -- for printer sharing support
  - bsd or setproctitle - for process title updating support
  - xsltproc and docbook XSL stylesheets -- Required for building man pages and other documentation

```bash
apt-get install build-essential libacl1-dev libattr1-dev \
   libblkid-dev libgnutls-dev libreadline-dev python-dev \
   python-dnspython gdb pkg-config libpopt-dev libldap2-dev \
   dnsutils libbsd-dev attr krb5-user docbook-xsl libcups2-dev
```

I don't know if this is needed later
kerberos:
Realm: SAMBATEST
host + password host: sambahost


#### Enable neede file-system Features
After installing the libraries, we continue:

We need to edit fstab to enable some features of ext3.
- Make  a backup of fstab:
  ```
  sudo cp /etc/fstab{,.bak}
  ```
- Add ```user_xattr,acl,barrier=1``` to the mountoptions of where samba will run to enable xattrs etc.
  - user_xattr: To use the advanced features of Samba4 you need a filesystem that supports both the "user" and "system" xattr namespaces.
  - acl: Support for the ACL on the file system (like ntfs does)
  - barrier=1: ensures that tdb transactions are safe against unexpected power loss. A number of sites have corrupted their AD database in sam.ldb by not having this option enabled.
- The Ubuntu 12.04 Server Kernel already has everything compiled the way we need
- Reboot Ubuntu to active new mount options.


#### Configure Host
I followed this german [guide](http://tridex.net/2012-07-04/active-directory-samba-4-ubuntu-linu/) and copy relevant stuff to this document.

- Static IP
  Folgende Werte müssen durch eigene ersetzt bzw. angepasst werden:

        sambaServer: Name des Servers
        domainName: Name der Domäne
        IP-Adressen

  Netzwerkkonfiguration
    Ubuntu muss zum Betrieb als Domain Controller eine statische IP-Adresse besitzen (/etc/network/interfaces):

        auto eth0
        iface eth0 inet static
        address 192.168.1.10
        netmask 255.255.255.0
        gateway 192.168.1.1
        dns-nameservers 192.168.1.10
        dns-search domainName.local


    Weiteres sollte der Hostname sowie der Domainname über die /etc/hosts aufgelöst werden können:

        127.0.0.1 localhost
        192.168.1.10 sambaServer sambaServer.domainName.local

- Install samba4 packages: ``` sudo apt-get install samba4 samba4-clients```
  Momentan kommt es bei der Installation zu einem Fehler weshalb dpkg das Paket nicht fertig konfigurieren kann. Den Fehler kann man allerdings ignorieren und so reicht es aus, das Paket manuelle als installiert zu markieren. Dazu sucht man sich in der Datei /var/lib/dpkg/status das Paket samba4 und ersetzt den Wert half-configured durch installed.



