# SPPKI
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
> └── .read.user
> ```
