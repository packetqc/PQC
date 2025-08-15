# PQC
Post Quantum Crypto learning project

# COMPLIANT

|Technology|ML-DSA|ML-KEM|
|--|--|--|
|WolfSSL|yes|yes|
|CMOX|yes|na|

# Prerequisites

1. Download wolfssl, zip and cubemx pack
2. Openssl patched for OQS Provider (required to gen pqc certificate) [link](https://github.com/wolfSSL/osp/blob/master/oqs/README.md)
   Install wolfssl with pqc features [link](https://github.com/wolfSSL/wolfssl/blob/master/INSTALL)
4. Or, for openssl, OpenSSL version 3.5.0 adds native support
5. For unix install of wolfssl, ./configure --enable-kyber --enable-dilithium
6. Liboqs static library 

# Procedure for embedded

1. create project with STM32CubeMX enabling wolfssl software package and PQC feature
2. build and import liboqs to STM32CubeIDE

# Embedded

## LIBOQS build (liboqs is for test and dev only)

[link](https://github.com/open-quantum-safe/liboqs/wiki/Customizing-liboqs/55cfed39e1027dd1d32170e6b91f557571b18d9e) can be reffered for additional details on building the library

1. apt install gcc-arm-none-eabi
2. download git liboqs
3. cd liboqs
4. git checkout 0.10.1 (or latest version compatible)
1. mkdir build
2. cd build
3. cmake .. -DOQS_BUILD_ONLY_LIB=ON
4. cmake .. -DOQS_BUILD_ONLY_LIB=ON;OQS_MINIMAL_BUILD="OQS_ENABLE_KEM_KYBER;OQS_ENABLE_KEM_ML_KEM;OQS_ENABLE_SIG_DILITHIUM;OQS_ENABLE_SIG_ML_DSA"
5. make

Could be optimized with <b>OQS_MINIMAL_BUILD="OQS_ENABLE_KEM_KYBER;OQS_ENABLE_KEM_ML_KEM;OQS_ENABLE_SIG_DILITHIUM;OQS_ENABLE_SIG_ML_DSA"</b> (Not tested)

## STM32 Installation

1. configure in STM32CubeMX and generate code to STM32CubeIDE project
2. copy and rename example settings file to user_settings.h
3. add symbol WOLFSSL_USER_SETTINGS
4. add symbol WOLFSSL_STM32_CUBEMX

# Linux

## Quick examples and tests on unix
For a quick start, you can run the client and server like this:
```
./examples/server/server -v 4 --pqc P521_ML_KEM_1024
./examples/client/client -v 4 --pqc P521_ML_KEM_1024
```

Copy the certificates and keys into the certs directory of wolfssl. Now you
    can run the server and client like this:

## Quick examples with certificate authentication

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

# Generation of certificates (on Linux distribution)

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
