#pam_krb5+ldap

Patch to extend the pam_krb5 compile options and functionality

Assists with UID/GID mapping for users that are not stored
locally, but within an LDAP directory. Supports RFC2307
POSIX account schema attributes for uid, gid, homeDirectory,
and defaultShell.

##Installation:
1. Pull down the latest pam_krb5 git repo like so:

```shell
git clone git://git.fedorahosted.org/pam_krb5.git
```

2. Pull down the latest patch to add ldap uid/gid mapping:

```shell
git clone git@github.com:jas-/pam_krb5-ldap.git
```

3. Apply the patch

```shell
patch -p0 > latest-pam_krb5+ldap.patch
```

4. Build necessary tools for build

```shell
./autogen
```

5. In order to enable this functionality simply run the configure
   command with the --with-ldap argument like so.

```shell
./configure --with-ldap
```

This will enable the linking against the required ldap.h and 
-lldap libraries and provide LDAP UID/GID mapping.

6. Install the authentication module

```shell
make install
```

If you do not see the "WITH_LDAP" flag during compile of the
source files you may need to force the param like so.

```shell
make CFLAGS="-DWITH_LDAP" install
```

7. Add LDAP specific data to /etc/krb5.conf appdefaults section.
An example is listed here:

```
[appdefaults]
pam = {
        ticket_lifetime = 1d
        renew_lifetime = 1d
        forwardable = true
        proxiable = false
        retain_after_close = false
        minimum_uid = 2
        try_first_pass = true
        ignore_root = true

        schema = ad #ad | ldap to meet your requirements
        ldapservs = ad.domain.com ldap.domain.com
        ldapport = 389
        binddn = uid=read-only-user,ou=Users,dc=ldap,dc=domain,dc=com
        basedn = ou=campus,dc=ad,dc=domain,dc=com
        ldapuser = [readonly-username]
        ldappass = [readonly-password]
        
        passwd = /etc/passwd
        shadow = /etc/shadow

	# If you wish to add the user to groups
	# a comma separted list should be used
	group = /etc/group
	groups = audio,cdrom,cdrw,usb,plugdev,video,games

        # If you define these they will
        # over write anything obtained from
        # ldap/active directory
        #homedir = /home
        #defshell = /bin/bash
}
```

8. Now you will want to add the pam_krb5 module to your authentication
stack per the service you wish to enable this authentication for.

```
auth            required        pam_env.so
auth            sufficient      pam_krb5.so
auth            sufficient      pam_unix.so try_first_pass likeauth nullok
auth            required        pam_deny.so

account         required        pam_unix.so

password        required        pam_cracklib.so difok=2 minlen=8 dcredit=2 ocredit=2 retry=3
password        sufficient      pam_krb5.so
password        sufficient      pam_unix.so try_first_pass use_authtok nullok sha512 shadow
password        required        pam_deny.so

session         required        pam_limits.so
session         required        pam_env.so
session         optional        pam_krb5.so
session         required        pam_unix.so
```

#Errors
If you are still unable to authenticate your users you may enable the debug option like so

```
auth	optional	pam_krb5.so debug
```

Please report any bugs to @ https://github.com/jas-/pam_krb5-ldap/issues
