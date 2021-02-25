# OpenLDAP + Kerberos lite

## Links

- [OpenLDAP과 Kerberos V 클러스터 구축](https://github.com/rurumimic/openldap-with-kerberos-v)
- [How to OpenLDAP](https://github.com/rurumimic/how-to-openldap)
- [How to OpenLDAP with Kerberos V](https://github.com/rurumimic/how-to-openldap-with-kerberos-v)

---

## Refresh Certs

you will see this error:

```bash
TASK [Add new entries] *********************************************************
fatal: [default]: FAILED! => {"changed": true, "cmd": "ldapadd -x -w password -D \"cn=manager,ou=admins,dc=example,dc=com\" -f directories.ldif -Z", "delta": "0:00:00.031612", "end": "2021-02-25 07:28:21.822569", "msg": "non-zero return code", "rc": 254, "start": "2021-02-25 07:28:21.790957", "stderr": "ldap_start_tls: Connect error (-11)\n\tadditional info: error:14090086:SSL routines:ssl3_get_server_certificate:certificate verify failed (certificate has expired)\nldap_result: Can't contact LDAP server (-1)", "stderr_lines": ["ldap_start_tls: Connect error (-11)", "\tadditional info: error:14090086:SSL routines:ssl3_get_server_certificate:certificate verify failed (certificate has expired)", "ldap_result: Can't contact LDAP server (-1)"], "stdout": "", "stdout_lines": []}
```

refresh certs:

```bash
cd ldap/certs
rm rootca* example*
./gen.certs.sh
```

---

## Start

```bash
vagrant up
```

### Client

```bash
vagrant ssh
```

#### KRB Ticket

`vagrant/lite.example.com@EXAMPLE.COM` / `password`:

```bash
kinit vagrant/lite.example.com

Password for vagrant/lite.example.com@EXAMPLE.COM:
# password
```

Verify:

```bash
klist

Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: vagrant/lite.example.com@EXAMPLE.COM

Valid starting     Expires            Service principal
02/25/21 07:44:52  02/25/21 19:44:52  krbtgt/EXAMPLE.COM@EXAMPLE.COM
	renew until 02/26/21 07:44:52

```

#### LDAP User

```bash
# ldapwhoami -Z
ldapwhoami -h lite.example.com -Z

SASL/GSSAPI authentication started
SASL username: vagrant/lite.example.com@EXAMPLE.COM
SASL SSF: 256
SASL data security layer installed.
dn:cn=vagrant/lite.example.com,ou=people,dc=example,dc=com
```

#### LDAP Search

1. Search 'keanu'
1. Add 'keanu': [Add data to LDAP](#add-data-to-ldap)
1. Search 'keanu'

```bash
ldapsearch -LLL -Y GSSAPI -h lite.example.com -b "dc=example,dc=com" -Z
ldapsearch -LLL -Y GSSAPI -h lite.example.com -b "dc=example,dc=com" "uid=keanu" -Z
```

#### Add data to LDAP

```bash
ldapadd -x -w password -D "cn=manager,ou=admins,dc=example,dc=com" -Z << EOF
dn: cn=Keanu Reeves,ou=people,dc=example,dc=com
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Keanu Reeves
uid: keanu
sn: Reeves
givenName: Keanu
uidNumber: 1001
gidNumber: 500
homeDirectory: /home/users/keanu
loginShell: /bin/bash
EOF
```

```bash
adding new entry "cn=Keanu Reeves,ou=people,dc=example,dc=com"
```

---

## Destroy VM

```bash
vagrant destroy -f
```
