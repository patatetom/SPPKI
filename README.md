# SPPKI
**S**imple **P**ersonal **PKI**.

SPPKI is in fact a [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) script based on [OpenSSL](https://en.wikipedia.org/wiki/OpenSSL) that allows you to quickly and easily set up a small « in-house » [PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure).

`SPPKIinit` :
- sets up the directory structure
- generates a root certificate ([RootCA](https://en.wikipedia.org/wiki/Root_certificate))
- generates a server certificate ([TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security))
- places the Bash scripts `create.user` and `revoke.user`, which are used to create and revoke a user ([mTLS](https://en.wikipedia.org/wiki/Mutual_authentication#mTLS)), respectively.

