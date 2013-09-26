#pam_krb5+ldap

Patch to extend the pam_krb5 compile options and functionality

Assists with UID/GID mapping for users that are not stored
locally, but within an LDAP directory. Supports RFC2307
POSIX account schema attributes for uid, gid, homeDirectory,
and defaultShell.

##Installation:
-   Pull down the latest patch to add ldap uid/gid mapping:

```shell
git clone https://github.com:jas-/pam_krb5-ldap.git
```

-   Pull down the latest pam_krb5 git repo like so:

```shell
git clone https://git.fedorahosted.org/pam_krb5.git
```

-   Apply the patch

```shell
patch -p0 > latest-pam_krb5+ldap.patch
```

-   Build necessary tools for build

```shell
./autogen
```

-   In order to enable this functionality simply run the configure
   command with the --with-ldap argument like so.

```shell
./configure --with-ldap
```

This will enable the linking against the required ldap.h and 
-lldap libraries and provide LDAP UID/GID mapping.

-   Install the authentication module

```shell
make install
```

If you do not see the "WITH_LDAP" flag during compile of the
source files you may need to force the param like so.

```shell
make CFLAGS="-DWITH_LDAP" install
```

##Configuration
You will now need to configure the /etc/krb5.conf & the PAM service files
to enable the pam_krb5.so authentication module.

### /etc/pam_krb5.conf
Add LDAP specific data to /etc/krb5.conf appdefaults section.

####Options
-   schema: Available options are ad or ldap (use ad for Microsoft Active Directory & ldap for OpenLDAP)
-   ldapservs: A space separated list of Active Directory or OpenLDAP servers hostnames (FQDN)
-   ldapport: The Active Directory or OpenLDAP port assignment (default is 389)
-   binddn: The bind DN for the read only account
-   basedn: The OU with which is a base for account lookups
-   ldapuser: The username to bind with (should be a limited privilege account)
-   ldappass: The password to bind with
-   passwd: The passwd file (default is /etc/passwd)
-   shadow: The shadow file (default is /etc/shadow, NO PASSWORDS GET ENTERED HERE)
-   group: The group file (default is /etc/group)
-   groups: A comma separated list of groups the authenticated user should be added to
-   homedir: If defined will overwrite the entry from Active Directory/OpenLDAP
-   defshell: If defined will overwrite the entry from Active Directory/OpenLDAP

####Example
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

### /etc/pam.d/<service-name>
Now you will want to add the pam_krb5 module to your authentication
stack per the service you wish to enable this authentication for.

#### Example
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

##Errors
If you are still unable to authenticate your users you may enable the debug option like so

```
auth	optional	pam_krb5.so debug
```

Please report any bugs to @ https://github.com/jas-/pam_krb5-ldap/issues
