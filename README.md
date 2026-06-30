# 🗝️ SPPKI

**S**imple **P**ersonal **PKI**.

SPPKI is in fact a [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) script based on [OpenSSL](https://en.wikipedia.org/wiki/OpenSSL) that allows you to quickly and easily set up a small « in-house » [PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure).

`SPPKIinit` :
- sets up the directory structure
- generates a root certificate ([RootCA](https://en.wikipedia.org/wiki/Root_certificate))
- generates a server certificate ([TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security))
- generates an empty certificate revocation list ([CRL](https://en.wikipedia.org/wiki/Certificate_revocation_list))
- places the Bash scripts `create.user` and `revoke.user`, which are used to create and revoke a user ([mTLS](https://en.wikipedia.org/wiki/Mutual_authentication#mTLS)).

The three variables `O` (Organization), `OU` (Organization Unit) and `WWW` (Common Name for server TLS) must be defined before starting the initialization by running `SPPKIinit` :
open the script in an editor, make the necessary changes, and take this opportunity to check its contents.

```console
/tmp/SPPKI$ bash SPPKIinit

♻️  Simple Personal PKI initialization...
1️⃣  Root CA key password...
Type CA key password 🔑 : 
Retype               🔑 : 
⚠️  THIS IS VERY IMPORTANT SECRET 🔑 : REMEMBER IT AND NOT SHARE IT ⚠️  
2️⃣  Root CA certificat generation...
3️⃣  Server certificat generation...
Using configuration from .conf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'www.example.org'
organizationName      :ASN.1 12:'Company'
organizationalUnitName:ASN.1 12:'Customer'
Certificate is to be certified until Jun 29 09:06:59 2036 GMT (3652 days)
Write out database with 1 new entries
Database updated
Using configuration from .conf
4️⃣  Client helpers generation...
🎉 Simple Personal PKI initialized !
Root CA certificate is « pki/root/ca.crt ».
Server TLS certificate and key are « pki/server/tls.{crt,key} ».
Server mTLS certificate revocation list is « server/crl.pem ».
You can now create and/or revoke users certificat.
```

Once initialized, the PKI has the following « visible » structure :

```
pki/
├── create.user
├── revoke.user
├── root/
│   ├── ca.crt
│   └── ca.key
├── server/
│   ├── crl.pem
│   ├── tls.crt
│   └── tls.key
└── users/
```

> The « hidden » part is for OpenSSL and `*.user` scripts :
> ```bash
> pki/
> ├── .certs/
> ├── .conf
> ├── .db/
> ├── .read.pass
> └── .read.user
> ```

```console
/tmp/SPPKI/pki$ ./create.user 

✅  User certificat generation...
User (ASCII alpha num chars only) : alice
Use « alice » for certificat ([Ctrl]-[C] to abort) ? 
Type CA key password 🔑 : 
Using configuration from .conf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'alice'
organizationName      :ASN.1 12:'Company'
organizationalUnitName:ASN.1 12:'Customer'
Certificate is to be certified until Jun 29 09:07:51 2036 GMT (3652 days)
Write out database with 1 new entries
Database updated
📦  User P12 generation...
Enter Export Password:
Verifying - Enter Export Password:
🚀  You can now send « users/alice.p12 » and « Export Password » to user
```

> once in Alice's hands, key and certificate contained in P12 file must be imported into system and/or browser.<br/>
> if using 👮‍♂️ [mTLSauth](https://github.com/patatetom/mTLSauth), command `python -c "print(int('$(openssl x509 -in /tmp/SPPKI/pki/users/alice.crt -noout -serial)'[7:], 16))"` can be used to easily retrieve serial number of client certificate in decimal format.

```console
/tmp/SPPKI/pki$ ./revoke.user 

✖️  User certificat revocation...
alice
User (ASCII alpha num chars only) : alice
Revoke « alice » ([Ctrl]-[C] to abort) ? 
Type CA key password 🔑 : 
Using configuration from .conf
Revoking Certificate D8D733393C76BBD896B7B38D13DD2E2B.
Database updated
Using configuration from .conf
📌  User « alice » is now revoked and CRL is updated
```
