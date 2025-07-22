# PQC
Post Quantum Crypto learning project

# Prerequisites

1. Download wolfssl, zip and cubemx pack
2. Openssl patched for OQS Provider (required to gen pqc certificate) [link](https://github.com/wolfSSL/osp/blob/master/oqs/README.md)
   Install wolfssl with pqc features [link](https://github.com/wolfSSL/wolfssl/blob/master/INSTALL)
4. Or, for openssl, OpenSSL version 3.5.0 adds native support
5. For unix install of wolfssl, ./configure --enable-kyber --enable-dilithium 

# STM32 Installation

1. configure in STM32CubeMX
2. copy and rename example settings file to user_settings.h
3. add symbol WOLFSSL_USER_SETTINGS
4. 

# Quick examples and tests on unix
For a quick start, you can run the client and server like this:
```
./examples/server/server -v 4 --pqc P521_ML_KEM_1024
./examples/client/client -v 4 --pqc P521_ML_KEM_1024
```

Copy the certificates and keys into the certs directory of wolfssl. Now you
    can run the server and client like this:

```
examples/server/server -v 4 -l TLS_AES_256_GCM_SHA384 \
   -A certs/mldsa87_root_cert.pem \
   -c certs/mldsa44_entity_cert.pem \
   -k certs/mldsa44_entity_key.pem \
   --pqc P521_ML_KEM_1024

examples/client/client -v 4 -l TLS_AES_256_GCM_SHA384 \
   -A certs/mldsa44_root_cert.pem \
   -c certs/mldsa87_entity_cert.pem \
   -k certs/mldsa87_entity_key.pem \
   --pqc P521_ML_KEM_1024
```      

# Generation of certificates

## NIST Levels

|Levels|Algorithm at generation|
|--|--|
|Dilithium NIST Level 2|algorithm mldsa44|
|Dilithium NIST Level 3|algorithm mldsa65|
|Dilithium NIST Level 5|algorithm mldsa87|

## Using OpenSSL 3.5.0 (includes pqc, oqs not required anymore)
```
 openssl genpkey -algorithm mldsa44 -outform pem -out mldsa44_root_key.pem
 openssl genpkey -algorithm mldsa44 -outform pem -out mldsa44_entity_key.pem
 openssl req -x509 -config root.conf -extensions ca_extensions -days 1095 -set_serial 20 -key mldsa44_root_key.pem -out mldsa44_root_cert.pem
 openssl req -new -config entity.conf -key mldsa44_entity_key.pem -out mldsa44_entity_req.pem
 openssl x509 -req -in mldsa44_entity_req.pem -CA mldsa44_root_cert.pem -CAkey mldsa44_root_key.pem -extfile entity.conf -extensions x509v3_extensions -days 1095 -set_serial 21 -out mldsa44_entity_cert.pem
 openssl verify -no-CApath -check_ss_sig -CAfile mldsa44_root_cert.pem mldsa44_entity_cert.pem
```

note: repeat for mldsa65 and mldsa87

## Example of certificate configuration files

### Content of root.conf
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

### Content of an entity.conf
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
