# PQC
Post Quantum Crypto learning project

# Prerequisites

1. Download wolfssl, zip and cubemx pack
2. Openssl patched for OQS Provider (required to gen pqc certificate) [link](https://github.com/wolfSSL/osp/blob/master/oqs/README.md)
   Install wolfssl with pqc features [link](https://github.com/wolfSSL/wolfssl/blob/master/INSTALL)
4. Or, for openssl, OpenSSL version 3.5.0 adds native support

# Generation of certificates

```
 openssl genpkey -algorithm mldsa44 -outform pem -out mldsa44_root_key.pem
 openssl genpkey -algorithm mldsa44 -outform pem -out mldsa44_entity_key.pem
 openssl req -x509 -config root.conf -extensions ca_extensions -days 1095 -set_serial 20 -key mldsa44_root_key.pem -out mldsa44_root_cert.pem
 openssl req -new -config entity.conf -key mldsa44_entity_key.pem -out mldsa44_entity_req.pem
 openssl x509 -req -in mldsa44_entity_req.pem -CA mldsa44_root_cert.pem -CAkey mldsa44_root_key.pem -extfile entity.conf -extensions x509v3_extensions -days 1095 -set_serial 21 -out mldsa44_entity_cert.pem
 openssl verify -no-CApath -check_ss_sig -CAfile mldsa44_root_cert.pem mldsa44_entity_cert.pem
```

note: repeat for mldsa65 and mldsa87

Content of root.conf
```
[ req ]
prompt                 = no
distinguished_name     = req_distinguished_name

[ req_distinguished_name ]
C                      = CA
ST                     = ON
L                      = Waterloo
O                      = wolfSSL Inc.
OU                     = Engineering
CN                     = Root Certificate
emailAddress           = root@wolfssl.com

[ ca_extensions ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer:always
keyUsage               = critical, keyCertSign
basicConstraints       = critical, CA:true
```

Content of entity.conf
```
[ req ]
prompt                 = no
distinguished_name     = req_distinguished_name

[ req_distinguished_name ]
C                      = CA
ST                     = ON
L                      = Waterloo
O                      = wolfSSL Inc.
OU                     = Engineering
CN                     = Entity Certificate
emailAddress           = entity@wolfssl.com

[ x509v3_extensions ]
subjectAltName = IP:127.0.0.1
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer:always
keyUsage               = critical, digitalSignature
extendedKeyUsage       = critical, serverAuth,clientAuth
basicConstraints       = critical, CA:false
```
