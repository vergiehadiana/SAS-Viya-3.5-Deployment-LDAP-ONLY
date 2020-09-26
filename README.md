# DISCLAIMER
```
####################################################################
#### DISCLAIMER                                                 ####
####################################################################
#### This Ansible Playbook is provided as-is, without warranty  ####
#### of any kind, either express or implied, including, but not ####
#### limited to, the implied warranties of merchantability,     ####
#### fitness for a particular purpose, or non-infringement.     ####
#### SAS Institute shall not be liable whatsoever for any       ####
#### damages arising out of the use of this documentation and   ####
#### code, including any direct, indirect, or consequential     ####
#### damages. SAS Institute reserves the right to alter or      ####
#### abandon use of this documentation and code at any time.    ####
#### In addition, SAS Institute will provide no support for the ####
#### materials contained herein.                                ####
####################################################################
#  This playbook:
#      was NOT throughly tested
#      is NOT part of Viya
#  The customer should provide the required LDAP and perform the required Authentication work on the Viya server(s)
```


# What this does
If you let it, this playbook will:

- Install OpenLDAP on a server
- Properly configure the MemberOf module
- Seed it with users and groups (of your choosing)
- Configure SSSD on multiple machines so they use the OpenLDAP server for authentication

# Why you may want to use it:
If you deploy Viya, you will need 2 things:

- a running LDAP
- to configure PAM to use that LDAP

This playbook will attempt to quickly do both of these things on a RHEL 7 environment. (It will not work for RHEL 6)

# How to get it:
- Option 1: through git:
  - ```git clone https://gitlab.sas.com/canepg/OpenLDAP.git```
- Option 2: download it as a zip file:
  - Go to the [gitlab page for the project](https://gitlab.sas.com/canepg/OpenLDAP)
  - Click the icon to download a zip file

# How to use it:
Once you you have the content deployed on your Ansible controller, you need to edit 2 files:

## inventory.ini

The inventory allows you to choose which machine is the OpenLDAP server and which one(2) is/are the OpenLDAP Client(s).
If you only have a single machine, it has to be in both host groups.
Replace the "fruit names" with names that are meaningful to you.

## group_vars/all.yml
This contains all the variables.
You don't need to update all of them.

# Execution

**Important**: This playbook is not idempotent. Running it twice in a row will not work.

Once you've updated both files to your liking, you will simply run:

```
ansible-playbook gel.openldapsetup.yml
```

Within about a minute, your OpenLDAP and SSSD configurations should be ready.

# Goodies
The last task of the playbook will give you a command to copy the generated sitedefault.yml to a specific folder in your viya playbook.
If you have not deployed Viya yet, this will auto-populate the right settings in Consul so you can connect right away.

The sitedefault.yml is built to match your newly installed OpenLDAP.

# Troubleshooting
If the playbook fails, or if you need to re-run it with alternative values, make sure to undo everything by running
```
ansible-playbook gel.openldapremove.yml
```
Then, try again.

This has only been tested on RHEL 7.3 so far. Not sure if it will work on anything else.

If you have issues with it, you can create an [issue](https://gitlab.sas.com/canepg/OpenLDAP/issues). I'll see what I can do.



# Manually adding users (contributed by Pravin Bhalerao)

If you want to manually add more users after OpenLDAP was stood up, you can use the below commands to do so.
Note that these would no longer be there if you uninstall OpenLDAP and re-install it.

Run following command as root to add users

``` ldapadd -x -W -D "cn=admin,dc=viya32awsrccm,dc=com" -f sinpro.ldif ```

Where sinpro.ldif contains

```
dn: uid=sinpro,ou=users,dc=viya32,dc=com
cn: sinpro
givenName: Pravin
sn: Bhalerao
objectClass: top
objectclass: inetorgperson
objectClass: organizationalPerson
objectClass: posixaccount
loginshell: /bin/bash
uidNumber: 100010
gidNumber: 100001
homedirectory: /home/sinpro
userPassword: Hello123
mail: sinpro@hostname
l: Cary
o: Demo_org
displayName: Pravin Bhalerao
```

To add multiple users, keep all .ldif in a directory and run –

``` for i in `ls /opt/sas/misc/openldap/users/*.ldif`; do sudo ldapadd -x -D "cn=admin,dc=viya32,dc=com" -w Hello123 -f $i ; done ```

You will need to make sure that these users have a home directory.

```su - <userid>```

will log you in as the user and create it. You would have to do that on each machine where you need the home directory to exist.
