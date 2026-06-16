# test localhost.localdomain


## PKI

PKI is the same on `host` and on `container` :

```console
tree pki/
pki/
|-- create.user
|-- revoke.user
|-- root
|   |-- ca.crt
|   `-- ca.key
|-- server
|   |-- crl.pem
|   |-- tls.crt
|   `-- tls.key
`-- users
    |-- user.crt
    |-- user.key
    `-- user.p12
```

PKI is generated on `host` and replicated to `container` :

```console
# @container
cd
nc -l -p 8888 | tar xv
```

```console
# @host
bash SPPKIinit
…
bash create.user
…
tar cv pki/ | nc -4 127.0.0.1 8888
```


## Caddy

```console
{
	admin off
	auto_https off
	debug

	servers {
		strict_sni_host on
	}
}

localhost.localdomain:8888 {
	reverse_proxy localhost:8000

	tls /root/pki/server/tls.crt /root/pki/server/tls.key {
		client_auth {
			mode require_and_verify
			trust_pool file /root/pki/root/ca.crt
			verifier revocation {
				mode crl_only
				crl_config {
					work_dir /tmp/
					storage_type memory
					signature_validation_mode verify
					crl_file /root/pki/server/crl.pem
					trusted_signature_cert_file /root/pki/root/ca.crt
				}
			}
		}
	}
}
```


## Curl

```console
curl https://localhost:8888/
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html
curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the webpage mentioned above.

curl -k https://localhost:8888/
curl: (56) OpenSSL SSL_read: OpenSSL/3.5.6: error:0A00045C:SSL routines::tlsv13 alert certificate required, errno 0
#                                                                                     ********************

curl --cacert pki/server/tls.crt https://localhost:8888/
curl: (56) OpenSSL SSL_read: OpenSSL/3.5.6: error:0A00045C:SSL routines::tlsv13 alert certificate required, errno 0
#                                                                                     ********************

curl --cacert pki/root/ca.crt https://localhost:8888/
curl: (56) OpenSSL SSL_read: OpenSSL/3.5.6: error:0A00045C:SSL routines::tlsv13 alert certificate required, errno 0
#                                                                                     ********************

curl --cacert pki/root/ca.crt --cert pki/users/user.crt --key pki/users/user.key https://localhost:8888/
OK
```


## Httpie

```console
https localhost.localdomain:8888
https: error: SSLError: HTTPSConnectionPool(host='localhost.localdomain', port=8888): Max retries exceeded with url: / (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1032)'))) while doing a GET request to URL: https://localhost.localdomain:8888/

https --verify no localhost.localdomain:8888
https: error: SSLError: HTTPSConnectionPool(host='localhost.localdomain', port=8888): Max retries exceeded with url: / (Caused by SSLError(SSLError(1, '[SSL: TLSV13_ALERT_CERTIFICATE_REQUIRED] tlsv13 alert certificate required (_ssl.c:2657)'))) while doing a GET request to URL: https://localhost.localdomain:8888/
#            ********************

https --verify pki/server/tls.crt localhost.localdomain:8888
https: error: SSLError: HTTPSConnectionPool(host='localhost.localdomain', port=8888): Max retries exceeded with url: / (Caused by SSLError(SSLError(1, '[SSL: TLSV13_ALERT_CERTIFICATE_REQUIRED] tlsv13 alert certificate required (_ssl.c:2657)'))) while doing a GET request to URL: https://localhost.localdomain:8888/
#            ********************

https --verify pki/root/ca.crt localhost.localdomain:8888
https: error: SSLError: HTTPSConnectionPool(host='localhost.localdomain', port=8888): Max retries exceeded with url: / (Caused by SSLError(SSLError(1, '[SSL: TLSV13_ALERT_CERTIFICATE_REQUIRED] tlsv13 alert certificate required (_ssl.c:2657)'))) while doing a GET request to URL: https://localhost.localdomain:8888/
#            ********************

https --verify pki/server/tls.crt --cert pki/users/alice.crt --cert-key pki/users/alice.key localhost.localdomain:8888
HTTP/1.1 200 OK
Alt-Svc: h3=":8888"; ma=2592000
Content-Length: 3
Content-Type: text/html
Date: Mon, 15 Jun 2026 20:19:57 GMT
Last-Modified: Mon, 15 Jun 2026 09:55:40 GMT
Server: SimpleHTTP/0.6 Python/3.13.5
Via: 1.0 Caddy

OK
```

