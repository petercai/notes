# extract private and public key

To extract private and public keys from a Java KeyStore (JKS) and convert them to PEM format, you can follow these steps:

1. **Convert JKS to PKCS#12**: Use the `keytool` command to convert the JKS file to a PKCS#12 file:

```bash
   keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.p12 -srcstoretype JKS -deststoretype PKCS12
```

2. **Extract Private Key**: Use `openssl` to extract the private key from the PKCS#12 file:

```bash
   openssl pkcs12 -in keystore.p12 -nocerts -nodes -out private_key.pem
```

3. **Extract Public Key**: Use `openssl` to extract the public key from the PKCS#12 file:

```bash
   openssl pkcs12 -in keystore.p12 -clcerts -nokeys -out public_cert.pem
```

4. **Convert Public Certificate to PEM**: If you need the public key in PEM format, you can convert the certificate:

```bash
   openssl x509 -inform PEM -in public_cert.pem -pubkey -noout > public_key.pem
```

These commands will help you convert your keys from JKS to PEM format [[IB from chatgpt]](https://www.baeldung.com/java-keystore-convert-to-pem-format) [[2]](https://stackoverflow.com/questions/652916/converting-a-java-keystore-into-pem-format). If you need more detailed guidance, you can check out tutorials on Baeldung [[IB from chatgpt]](https://www.baeldung.com/java-keystore-convert-to-pem-format) or Stack Overflow[[2]](https://stackoverflow.com/questions/652916/converting-a-java-keystore-into-pem-format).

Do you need help with any specific part of this process?

  
References

[1] [Converting a Java Keystore Into PEM Format - Baeldung](https://www.baeldung.com/java-keystore-convert-to-pem-format)

[2] [Converting a Java Keystore into PEM Format - Stack Overflow](https://stackoverflow.com/questions/652916/converting-a-java-keystore-into-pem-format)