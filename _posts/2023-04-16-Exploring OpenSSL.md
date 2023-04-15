---
title: Exploring OpenSSL
date: 2023-04-16 02:00:00 +0500
categories: [Blogs, Network Security]
tags: [network security, openssl, ssl, encryption, decryption, secure communication, cryptography]
---

## What is OpenSSL?
OpenSSL is an open-source tool for cryptography- mainly SSL and TLS. It is a powerful tool when and supports all kinds of functionalities- from key generation to client/server tests. 

## Use Cases
The wide range of use cases that OpenSSL covers is too wide to list here. But here are some of the top use cases.
****
### Assymetric key-pair generation
Assymetric encryption is an important part of today's IT infrastructure, including websites. OpenSSL supports many public key encryption algorithms. I will be covering RSA and Eliptic Curves. 

#### RSA 
Generating an RSA private key:<br/>
```openssl genpkey -algorithm RSA -out RSApriv.pem```<br/><br/>
Generating an RSA public key:<br/>
```openssl rsa -pubout -in RSApriv.pem -out RSApub.pem```<br/><br/>
Here are what the public and private keys look like:
![image](https://user-images.githubusercontent.com/88616338/223767406-f289075b-4828-4521-bbf2-a566908436cd.png)
![image](https://user-images.githubusercontent.com/88616338/223767420-7b433c3c-437f-4c62-9799-2ce13b2b8dba.png)

#### Eliptic Curve
Listing the support eliptic curves:<br/>
```openssl ecparam -list_curves```<br/><br/>
We will be using the ```secp384r1``` curve.<br/><br/>
Generating the private key:<br/>
```openssl ecparam -genkey -name secp384r1 -out ECpriv.pem```<br/><br/>
Generating the public key:<br/>
```openssl ec -in ECpriv.pem -pubout -out ECpub.pem```<br/><br/>
Here are what the public and private keys look like:
![image](https://user-images.githubusercontent.com/88616338/223768757-f596e1bd-7df4-4fff-8e12-69fac5babb78.png)
![image](https://user-images.githubusercontent.com/88616338/223768780-d78ddba7-5b2e-4bdf-a0bb-198cb24f520b.png)

****

### Certificate Signing Request (CSR) and generation of certificates
Generating a signing request and the signing key:<br/>
While generating this, we will be asked for our region/organization information.<br/>
```openssl req -new -newkey rsa:2048 -nodes -keyout CSRkey.pem -out CSR.pem```<br/><br/>
Generating an X509 certificate:<br/>
```openssl x509 -req -days 365 -in CSR.pem -signkey CSRkey.pem -out CSRCert.pem```<br/><br/>
Here is what the X509 certificate looks like:
![image](https://user-images.githubusercontent.com/88616338/223769978-a223063e-e0fd-4e30-b240-51f953f3b595.png)
****
### Certificate Revocation List (CRL) and revoking existing certificates
For creating and using CRLs, we first need to configure a CA (Certficate Authority). 
#### Configuring a CA
- Create a new directory for your CA and navigate into that directory.
- Generate a symmetric key for the CA (we will be asked to set a key paraphrase in this step)<br/>
```openssl genpkey -algorithm RSA -out ca.key -aes256```<br/>
- Request an X509 certificate for the CA (we will be asked for region/organization information in this step)<br/>
```openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt```<br/>
- Create a configuration file ```ca.cnf``` for your CA and ensure its contents are as follows:<br/>
* Replace ```/home/kali/Documents/Networks2/CA/``` with the path to your CA directory<br/>
![image](https://user-images.githubusercontent.com/88616338/223778632-0a34174d-8034-46b6-a7f2-c1f921951fc7.png)
- Create necessary directories and files for your CA (in the CA directory)<br/>
```bash
mkdir -p {certs,newcerts}
touch index.txt
echo 01>serial
```
#### Certificate Revocation List and revocation
Revoking an existing certificate:<br/>
```openssl ca -config CA/ca.cnf -revoke CSRCert.pem```<br/><br/>
Generating a CRL to add revoked certificates:<br/>
```openssl ca -config CA/ca.cnf -gencrl -keyfile CA/ca.key -cert CA/ca.crt -out CRL.crl```<br/><br/>
Verifying a certificate against an existing CRL (this step should give you the output "Verification failed" for a revoked certificate):<br/>
```openssl verify -crl_check -CAfile CA/ca.crt -CRLfile CRL.crl CSRCert.pem```<br/><br/>
****
### Message Digest (Hash) Calculation
Listing supported message digest algorithms:<br/>
```openssl list -digest-algorithms```<br/><br/>
Getting the message digest of a file:<br/>
```openssl <hash_algorithm> <file_name>```<br/><br/>
Example MD5 calculation:<br/>
![image](https://user-images.githubusercontent.com/88616338/223780880-d1a3b4b6-b011-4aa7-bb14-0b41fb6ef0c0.png)<br/>
Example SHA1 calculation:<br/>
![image](https://user-images.githubusercontent.com/88616338/223780941-5bc8b59c-cc04-41cd-a9ac-e634f15ee973.png)<br/>
Example SHA256 calculation:<br/>
![image](https://user-images.githubusercontent.com/88616338/223781196-58513bf6-f06f-4c30-beec-f2c7c903c493.png)
****
### Symmetric Encryption/Decryption
Listing supported ciphers:<br/>
```openssl list -cipher-algorithms```<br/><br/>
We can use ```AES-256-CBC``` as an example.
- Create a text file:<br/>
```echo "THIS IS A TEST FILE" > plaintext.txt```
- Encrypt the text file:<br/>
```openssl enc -aes-256-cbc -in plaintext.txt -out ciphertext.enc```<br/><br/>
Here is what the encrypted file looks like:<br/>
![image](https://user-images.githubusercontent.com/88616338/223781968-d87ccb5b-7a31-4fa0-a84e-a61e87fef851.png)
- Decrypt the encrypted file:<br/>
```openssl enc -aes-256-cbc -d -in ciphertext.enc -out plaintext2.txt```<br/><br/>
****
### SSL/TLS Client/Server Testing
Generating server key and certificate:<br/>
```openssl req -newkey rsa:2048 -nodes -keyout SERV.key -x509 -days 365 -out SERV.crt```<br/><br/>
Generating client key and certificate:<br/>
```openssl req -newkey rsa:2048 -nodes -keyout CLIENT.key -x509 -days 365 -out CLIENT.crt```<br/><br/>
Starting an SSL server on port 4433:<br/>
```openssl s_server -accept 4433 -key SERV.key -cert SERV.crt```<br/><br/>
Using an SSL client to connect to the SSL server:<br/>
```openssl s_client -connect localhost:4433 -key CLIENT.key -cert CLIENT.crt```<br/><br/>
Here is an example of a basic TLS client/server communication using OpenSSL:
![image](https://user-images.githubusercontent.com/88616338/223783878-956516da-3988-4fb7-9bab-f23551327378.png)
![image](https://user-images.githubusercontent.com/88616338/223783901-cfca4cb3-5efd-445e-ae25-1aac240495f5.png)
****
### Mail Encryption/Decryption using S/MIME
Generating a key and certificate for the recipient:<br/>
```openssl req -newkey rsa:2048 -nodes -keyout KEY.pem -x509 -days 365 -out CERTIFICATE.pem```<br/><br/>
Encrypting the mail with the recipient's public key:<br/>
```openssl smime -encrypt -aes256 -in plaintext.txt -out ciphertext.enc -outform DER CERTIFICATE.pem```<br/><br/>
Decrypting the mail with the recipient's private key:<br/>
```openssl smime -decrypt -in ciphertext.enc -inform DER -out plaintext2.txt -inkey KEY.pem```<br/><br/>
Here is an example output of what "THIS IS A VERY IMPORTANT MAIL" looks like when it is encrypted:<br/>
![image](https://user-images.githubusercontent.com/88616338/223788877-71cab07b-6672-4cec-8087-9d8f29383403.png)

## Conclusion
After exploring just about 30% of the use cases of OpenSSL, we can see how comprehensive and powerful the tool is for cryptography tests and network security. All large organizations use OpenSSL for the testing and their security protocols and you should too. OpenSSL is a lifesaver and automates countless procedures that you would follow manually.
