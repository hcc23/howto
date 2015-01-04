How-To create an email certificate with CAcert
=============================================

Preliminaries
-------------

Check your SSL version and make sure it is current:

```
:~> which openssl
/usr/bin/openssl
:~> openssl version
OpenSSL 1.0.1j-fips 15 Oct 2014
```

    
Certtificate Creation
---------------------

Now on to the actual creation:

- Create a new folder to keep your files organized:

```
:~> mkdir myNewKey
:~> cd myNewKey
:~/myNewKey> 
```

- Generate a 4096 bit long RSA key, encrypt it with AES, and store it in a
PEM formatted file:

```
:~/myNewKey> openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -aes-256-ecb -outform PEM -out private_key.pem
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
:~/myNewKey> 
```

- Generate a Certificate Signature Request (CSR):

```
:~/myNewKey> openssl req -new -inform PEM -key private_key.pem -outform PEM -out csr.pem
Enter pass phrase for private_key.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:. 
State or Province Name (full name) [Some-State]:.
Locality Name (eg, city) []:.
Organization Name (eg, company) [Internet Widgits Pty Ltd]:.
Organizational Unit Name (eg, section) []:.
Common Name (e.g. server FQDN or YOUR name) []: <YOUR NAME>
Email Address []: <YOUR EMAIL ADDRESS>

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
:~/myNewKey> 
```
    
- Open the newly created csr.pem in a texteditor or display it in the console:

```
:~/myNewKey> cat csr.pem
-----BEGIN CERTIFICATE REQUEST-----
8< a lot of gibberish ASCII characters 8<
-----END CERTIFICATE REQUEST-----
:~/myNewKey> 
```

The next couple of steps happen on the CAcert website (https://www.cacert.org/index.php?id=4).
    
- Log on to CAcert and go to 'Client Certificates'->'New' (https://www.cacert.org/account.php?id=3)

- Select the email address you enterd in step 3 above and check that the name
also matches.[1]

- Check the 'Show Advanced Options' box and copy-paste your CSR into the text 
field. (Note: the lines with 'BEGIN CERTIFICATE REQUEST' and 'END CERTIFICATE REQUEST'
are part of the request and need to be copy-pasted as well!)

- Check 'I accept the CAcert Community Agreement (CCA)' box and click 'Next'

After a while the website will show you your certificate (again a lot of ASCII 
gibberish enclosed in some 'BEGIN CERTIFICATE' and 'END CERTIFICATE' lines).

- Click 'Download certificate in PEM format' and store the key in your folder.

In order to keep track of certificates, etc., rename your key folder and the 
certificate to the serial number of the certificate (shown on the website below 
the actual certificate information; for this example, let's use '0123AB')

- Renaming the certificate and the folder:

```
:~/myNewKey> mv your.email@address.com.crt 0123AB.crt
:~/myNewKey> cd ..
:~> mv myNewKey 0123AB
:~>
```

- Download and store the CAcert class 1 and class 3 root certificates in your 
key folder. You can get the CAcert root certificates from 
https://www.cacert.org/index.php?id=3, download the PEM formated versions.[2]

- Combine your certificate, your key, and the CAcert certificate into a single
PKCS#12 container:

```
:~/0123AB> openssl pkcs12 -export -inkey private_key.pem -in 0123AB.crt -certfile cacert_class1.crt -certfile cacert_class3.crt -out 0123AB.p12
Enter pass phrase for private_key.pem:
Enter Export Password:
Verifying - Enter Export Password:
```

ToDo's
------

- A write up for including several email addresses into the CSR



Footnotes
---------

1: You can only enter a name if you have enough points.
2: So how can you check that CAcert is actually CAcert? (After all, cacert.org 
uses a self-signed certificate?) Check against a third party: The Allianz 
insurance group lists the certificates they trust at https://rootca.allianz.com/de/secumail_calist.htm.
The CAcert certificates are listed there, so check that site and compare the SHA1
fingerprints. (ALL digits!)
