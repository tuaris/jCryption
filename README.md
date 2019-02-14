## What do you have to do

Easy on the client ... some work on the server.
You can find a sample PHP implementation in the repo.

Include jQuery 1.6.1+ and jCryption 3.0.1

```
<script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"></script>
<script type="text/javascript" src="jquery.jcryption.3.0.1.js"></script>
```             

And than call jCryption on your form

```
$(function() {
    $("form").jCryption();
});
```

thats it ... move on to the server ...
Generate public and private key with OpenSSL (also described here)
Keep in mind, you only have to generate the public and private key once

```
:~$ openssl genrsa -out rsa_1024_priv.pem 1024
:~$ openssl rsa -pubout -in rsa_1024_priv.pem -out rsa_1024_pub.pem
```

If you want it more secure ... it also works with a 2048 or 4096 Bit key
Next are the OpenSSL calls you need on the server to work in sync with the client.
But before that ... here is a general description what exactly happends.

## How does this work

    client requests RSA public key from server
    client encrypts a randomly generated key with the RSA public key
    server decrypts key with the RSA private key and stores it in the session
    server encrypts the decrypted key with AES and sends it back to the client
    client decrypts it with AES, if the key matches the client is in sync with the server and is ready to go
    everything else is encrypted using AES

## OpenSSL

You can find the complete documentation of OpenSSL here for AES and here for RSA.

    Decrypting the key the client sent with RSA.
    Input: base64 encoded key
    Output: unencrypted key from client

    :~$ openssl rsautl -decrypt -inkey rsa_1024_priv.pem

    Send the client a challenge to solve. Encrypt the sent key with encrypted key and send it back to the client.
    Input: unencrypted key from client
    Output: encrypted key sent to the client

    :~$ openssl enc -aes-256-cbc -pass pass:'key' -a -e

    The server has to decrypt the data sent from the client with AES to work with the data.
    Input: encrypted data from client
    Output: unencrypted data

    :~$ openssl enc -aes-256-cbc -pass pass:'key' -d

