# NETXDUO TLS 1.3 for PQC

## X509 Certificate
- Using xca to create ed ca, server and client certificates.
- Add pqc extension with openssl 3.5 or later. [doc](- https://github.com/packetqc/PQC#using-openssl-350-includes-pqc-oqs-not-required-anymore).
- export der format CA certificate ( and private key PKCS#1 for RSA, RFC 5915 for ECC, if private key is required ). For certificates that identify a device, the associated private key must be loaded along with the certificate.
- export der format client and server certificates ( and private keys if RSA) but ED
- convert to c with 'xxd -i'

  ```
      xxd -i your_certificate.crt > certificate_data.h
  ```

## STM32 Compilation and code
<details>
<summary>click here to see details...</summary>

- NX_SECURE_TLS_ENABLE_TLS_1_3 must be defined globally (use CubeMX and enable crypto prerequisite for x509 validations)
- If no key is supplied, the value NX_SECURE_X509_KEY_TYPE_NONE (0x00). Other values for keys are NX_SECURE_X509_KEY_TYPE_RSA_PKCS1_DER (0x01 RSA, PKCS#1) and NX_SECURE_X509_KEY_TYPE_EC_DER (0x02 ECDSA, RFC 5915).

### server initialization
- nx_secure_x509_certificate_initialize
- nx_secure_tls_local_certificate_add
- nx_secure_tls_local_certificate_remove
- NX_SECURE_X509_CERT is populated

### client initialization
- nx_secure_tls_remote_certificate_allocate ? still required ... nx_secure_tls_session_packet_buffer_set ?
- nx_secure_x509_certificate_initialize
- nx_secure_tls_trusted_certificate_add
- nx_secure_trusted_certificate_remove
- NX_SECURE_X509_CERT is populated

### tls session start
requires to establish tcp first and use it in tls control block

#### server
- nx_tcp_server_socket_listen
- nx_tcp_server_socket_accept

#### client
- nx_tcp_client_socket_connect

#### upon tcp establishment
- nx_secure_tls_session_start

### network packet allocation
- nx_secure_tls_packet_allocate ( not nx_packet_allocate )

### tls session send
- nx_secure_tls_session_send ( identical as nx_tcp_socket_send )

### tls session receive
- nx_secure_tls_session_receive ( identical as nx_tcp_socket_receive )

### tls session close
once tls session is completed, both end must CloseNotify alert to the other side to shutdown the session. Both sides must receive and process the alert to ensure a successful session shutdown.

- nx_secure_tls_session_end ( this will call the session close notify alert )
- NX_SECURE_TLS_SESSION_CLOSED received if trying to transmit to a closed session.
- NX_SECURE_TLS_SESSION_CLOSE_FAIL is the return code on session close failed.

### certificate validation
When using TLS with X.509 certificates for host identification and verification, it is important to understand how those certificates are actually validated. While the TLS specification does not provide detailed instructions on how to validate a certificate, it does refer to the X.509 specification (RFC 5280). In general, it is expected that TLS will perform at least basic validation on incoming certificates (those certificates supplied by the remote host during the TLS handshake), and NetX Duo Secure TLS is no different.

- nx_secure_tls_session_certificate_callback_set : for any additionnal validation: set a callback that will be called automatically, if set, after basic validation.

#### additionnal routines
- nx_secure_x509_common_name_dns_check
- nx_secure_x509_crl_revocation_check
- nx_secure_x509_extended_key_usage_extension_parse
- nx_secure_x509_key_usage_extension_parse
- nx_secure_x509_extension_find
  
NetX Duo Secure's X.509 implementation does provide a service to extract unsupported extensions as well: nx_secure_x509_extension_find. This API is intended for advanced users as it requires knowledge of DER-encoded ASN.1 in order to parse the data returned. It it used internally to extract supported extensions but is supplied for convenience in developing customized support for X.509 extensions.

```
typedef struct NX_SECURE_X509_EXTENSION_STRUCT
{
    /* Identifier (maps to OID) for this extension. */
    USHORT nx_secure_x509_extension_id;

    /* Critical flag - boolean value. */
    USHORT nx_secure_x509_extension_critical;

    /* Pointer to DER-encoded extension data. */
    const UCHAR *nx_secure_x509_extension_data;
    ULONG        nx_secure_x509_extension_data_length;
} NX_SECURE_X509_EXTENSION;
```

To use nx_secure_x509_extension_find, a NX_SECURE_X509_EXTENSION is passed in, along with the certificate and an extension ID, which is an integer representation of the variable-length OID string for a known extension type. A complete list of supported OIDs for X.509 extensions is provided in the API reference for nx_secure_x509_extension_find on page 178.

### client certificate authentication
- nx_secure_tls_local_certificate_add

### server certificate authentication
When Client Certificate Authentication is enabled, the TLS Server will request a certificate from the remote TLS Client during the TLS handshake. In NetX Duo Secure TLS Server, the Client certificate is checked against the store of trusted certificates created with nxsecure_tlstrustedcertificateadd following the X.509 issuer chain.

- nxsecuretlssessionclientverifyenable (to enable client authentication. coded before calling nx_secure_tls_session_start)
- nxsecuretlssessionclientverifydisable 

### cryptographic method
NetX Duo Secure TLS provides the following encryption methods: AES RSA NULL
NetX Duo Secure TLS provides the following authentication methods: HMAC-MD5 HMAC-SHA1 HMAC-SHA256

nx_crypto_algorithm: This field identifies the algorithm described in the variable method Some valid values for NetX Duo Secure TLS are as follows (refer to nx_crypto_const.h for specific values): NX_CRYPTO_NONE NX_CRYPTO_ENCRYPTION_NULL NX_CRYPTO_ENCRYPTION_AES_CBC NX_CRYPTO_AUTHENTICATION_NONE TLS_HASH_SHA_1 TLS_HASH_SHA_256 TLS_HASH_MD5 TLS_CIPHER_RSA TLS_CIPHER_NULL

- NX_CRYPTO_METHOD
- crypto_init_function
- crypto_cleanup_function
- crypto_operation_function : This is the routine that performs the actual encryption, decryption, and authentication services like NX_CRYPTO_ENCRYPT, NX_CRYPTO_DECRYPT, NX_CRYPTO_AUTHENTICATE, NX_CRYPTO_VERIFY


</details>


# References
[github threadx netxduo tls 1.3](https://github.com/eclipse-threadx/rtos-docs/blob/main/rtos-docs/netx-duo/netx-duo-secure-tls/chapter3.md)
[X.509 Section 9.8 extensions to enable hybrid certificates](https://www.bouncycastle.org/resources/preparing-for-the-migration-to-post-quantum-public-key-algorithms-with-hybrid-certificates/)
