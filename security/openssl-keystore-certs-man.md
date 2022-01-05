# Certificates and Java KeyStore manipulation
- [Certificates and Java KeyStore manipulation](#certificates-and-java-keystore-manipulation)
  - [Prerequisites](#prerequisites)
  - [Generating Public/Private key pair](#generating-publicprivate-key-pair)
    - [Generate Private Key](#generate-private-key)
    - [Generate Public Key](#generate-public-key)
    - [Generate Certificate (X.509)](#generate-certificate-x509)
    - [Package Private and Public certificate in PKCS12](#package-private-and-public-certificate-in-pkcs12)
  - [Java Keystore](#java-keystore)
    - [Creating Java Key Store](#creating-java-key-store)
    - [View Java Key Store](#view-java-key-store)
    - [Edit/Change alias name of existing entry](#editchange-alias-name-of-existing-entry)
    - [Clone entry within the key store](#clone-entry-within-the-key-store)
    - [Delete/remove an entry from key store](#deleteremove-an-entry-from-key-store)
    - [Export certificate from the key store](#export-certificate-from-the-key-store)
    - [Import Public key in the key store](#import-public-key-in-the-key-store)
    - [Import Private Key in the key store (PKCS12)](#import-private-key-in-the-key-store-pkcs12)
---
## Prerequisites
  - openssl commandline tool
  - java keytool

---
## Generating Public/Private key pair
The following section contains the explaination and the shell commands in order to generate public private key pair.

### Generate Private Key
Using openssl command, generate private key. Find the command below:
```shell
openssl genrsa -out private.pem 2048
```

### Generate Public Key
Once the private key has been generated, utilise the private key and generate public key. Find the command below:
```shell
openssl rsa -in private.pem -pubout -out public.pem
```

### Generate Certificate (X.509)
X.509 certificate is a standard defining format for storing public key certificates.
```shell
openssl req -new -x509 -sha512 -key private.pem -out certificate.pem -days 3600
```
In above command, the certificate is valid for 10 years. It accepts arguments in days.

### Package Private and Public certificate in PKCS12 
Package private and public X.509 certifcation in PKCS12 format that archives private key and X.509 certificate in a file with password authentication. Find the command below:
```shell
openssl pkcs12 -export -inkey private.pem -in certificate.pem -out certificate.p12
```
Now, PCKB12 can be used to be stored in the Java Key Store.

---
## Java Keystore
### Creating Java Key Store
This section explains how to create a KeyStore using the JKS format as the database format for both the private key, and the associated certificate or certificate chain.
```shell
keytool -genkey -keystore mytruststore -alias sample -keyalg RSA -keysize 2048 -validity 3600
```
Note that, the validity provided in the arguments is in days.
It will prompt following details:
```
Enter keystore password:  
Re-enter new password: 
What is your first and last name?
  [Unknown]:  Nishith Shah
What is the name of your organizational unit?
  [Unknown]:  RnD
What is the name of your organization?
  [Unknown]:  Procrastination Inc
What is the name of your City or Locality?
  [Unknown]:  Ahmedabad
What is the name of your State or Province?
  [Unknown]:  Gujarat
What is the two-letter country code for this unit?
  [Unknown]:  IN
Is CN=Nishith Shah, OU=RnD, O=Procrastination Inc, L=Ahmedabad, ST=Gujarat, C=IN correct?
  [no]:  yes
```
Once all the details supplied, you can view the file created with name `mytruststore`. Now let's see how to view the file.

### View Java Key Store
To list down the certificates in the store, use following command:
```shell
keytool -list -v -keystore mytruststore
```
It will prompt trust store password. This command will list down all the certificates.

To view specific certificate with specific alias, try the command below:
```shell
keytool -list -v -keystore mytruststore -alias sample
```
### Edit/Change alias name of existing entry
There may be necessity to change the name of existing entry of certificate inside the key store. `keytool` provides to change the name of the existing alias. Find the command below:

```shell
keytool -changealias -alias sample -destalias shillyshally -keystore mytruststore
```
To view the changes, [View Java Key Store](###view-java-key-store)

### Clone entry within the key store
To clone any entry within the keystore, use following command:
```shell
keytool -keyclone -alias shillyshally -dest dither -keystore mytruststore
keytool -keyclone -alias shillyshally -dest dillydally -keystore mytruststore
```

### Delete/remove an entry from key store
To delete entry of certificate from the keystore, use following command:
```shell
keytool -delete -alias dither -keystore mytruststore
```
Make sure that the correct alias is passed.

### Export certificate from the key store
To export certificate to external file, use following command:
```shell
keytool -exportcert -keystore mytruststore -alias shillyshally -file shillyshally.pem
keytool -exportcert -keystore mytruststore -alias shillyshally -file shillyshally.crt
```
To export a public key use -importkeystore command. It will export all certificates into the destination file.
```shell
keytool -importkeystore -srckeystore mytruststore -destkeystore dillydally.p12 -deststoretype PKCS12
```

### Import Public key in the key store
Import a public key in the key store using following command:

```shell
keytool -import -alias tiptoe -file certificate.pem -keystore mytruststore
```

### Import Private Key in the key store (PKCS12)
Private key imformation cannot be imported directly into the keystore. Instead you must import the existing private key and the certificate into **`PKCS12 (.p12)`** file. Refer to the section [Package Private and Public certificate in PKCS12](###Package-Private-and-Public-certificate-in-PKCS12) to create **`PKCS12`** store. Find the command below to import **`PKCS12 (.p12)`** file.

```shell
keytool -importkeystore -destkeystore mytruststore -srckeystore certificatestore.p12 -srcstoretype PKCS12
```
---
