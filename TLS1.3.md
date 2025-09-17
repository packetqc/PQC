# NETXDUO TLS 1.3 for PQC

## X509 Certificate
- Using xca to create ed ca, server and client certificates.
- Add pqc extension with openssl 3.5 or later. [doc](- https://github.com/packetqc/PQC#using-openssl-350-includes-pqc-oqs-not-required-anymore)
- export der format
- convert to c with xdd (or xxd.. tbc name of utility)

## STM32 Compilation
- NX_SECURE_TLS_ENABLE_TLS_1_3 must be defined globally

# References
[github threadx netxduo tls 1.3](https://github.com/eclipse-threadx/rtos-docs/blob/main/rtos-docs/netx-duo/netx-duo-secure-tls/chapter3.md)

