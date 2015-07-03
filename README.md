## Simple Python tool for creating PKI structures (CA / Intermediates), and using them to sign certificates.

### Dependencies

* Python 2.7, 3.3 (because of pyOpenSSL)
* pyOpenSSL (sudo pip install pyopenssl)


### Examples

##### Create a CA, and a Intermediate CA

    sh$ catool create -cn myca.example.com -lt 1825 > myca.pem # Creates our CA key and certificate (self-signed), pipes both to a file myca.pem. Sets lifetime to 5 years.

    sh$ catool create -cn myintermediateca.example.com -lt 1825 -pc myca.pem -serial 2 -pl 0 > myintermediateca.pem # Creates our intermediate CA using myca.pem as issuer and signer. Piped into the file myintermediateca.pem.



##### Use a CA or Intermediate CA to sign (create) a couple of certificates

    sh$ catool sign-certificate -cn mysite.example.com -ca-c myintermediateca.pem > server1.pem

    sh$ catool sign-certificate -cn mysite2.example.com -ca-c myintermediateca.pem -serial 1001 > server2.pem

    sh$ catool sign-certificate -cn mysite3.example.com -ca-c myintermediateca.pem -serial 1002 -s 2048 > server3.pem



##### Convert a certificate and its key from PEM format into pkcs#12 format

    sh$ catool convert-certificate -ca-c myca.pem -c mycertificateandkey.pem -p "Password" > mycertificateandkey.pfx



### Documentation

##### sh$ catool -h
    usage: catool [-h] {create,sign-certificate} ...

    positional arguments:
      {create,sign-certificate}
        create              Create a new Certificate Authority (CA).
        sign-certificate    Sign a keypair (optionally creating one), using your
                            root or intermediate CA.

    optional arguments:
      -h, --help            show this help message and exit



##### sh$ catool create -h
    usage: catool create [-h] -cn <common name> [-k <key>]
                         [-lt <lifetime in days>] [-pc <certificate>] [-pk <key>]
                         [-pl <path length>] [-s <keysize>] [-serial <serial>]

    optional arguments:
      -h, --help            show this help message and exit
      -cn <common name>     The common name specified in our new CA / Intermediate
                            certificate.
      -k <key>              Private key (in PEM-format) that you wish to use as
                            the CA's key. Optionally, if you don't specify this, a
                            new key will be generated for you.
      -lt <lifetime in days>
                            The Certificate will be valid for the specified number
                            of days. Defaults to 365 days (a year).
      -pc <certificate>     Use to create an Intermediate Certificate Authority.
                            If a parent key is not specified (by -pk), then we
                            will try to read it from this file as well.
      -pk <key>             Use to create an Intermediate Certificate Authority.
      -pl <path length>     How many intermediate CA's can be chained to sign
                            certificates on our CA's behalf.
      -s <keysize>          If no private key is specified (using -k), a new one
                            will be generated using this keysize. Defaults to 4096
                            bit.
      -serial <serial>      Serial number for the certificate. Defaults to 1.



##### sh$ catool sign-certificate -h
    usage: catool sign-certificate [-h] -ca-c <certificate> [-ca-k <key>] -cn
                                   <common name> [-k <key>]
                                   [-lt <lifetime in days>] [-s <keysize>]
                                   [-serial <serial>]

    optional arguments:
      -h, --help            show this help message and exit
      -ca-c <certificate>   The Certificate Authority certificate to use when
                            signing our keypair. If a Certificate Authority key is
                            not specified on its own (using -ca-k), then we will
                            try to read it from this file as well.
      -ca-k <key>           The Certificate Authority key to use when signing our
                            keypair.
      -cn <common name>     The common name specified in our new, signed,
                            certificate. *Hint* This is usually the hostname (URL)
                            of our web-server when using the target keypair for
                            HTTPS.
      -k <key>              Private key (in PEM-format) that you wish to sign.
                            Optionally, if you don't specify this, a new key will
                            be generated for you.
      -lt <lifetime in days>
                            The Certificate will be valid for the specified number
                            of days. Defaults to 365 days (a year).
      -s <keysize>          If no private key is specified (using -k), a new one
                            will be generated using this keysize. Defaults to 4096
                            bit.
      -serial <serial>      Serial number for the certificate. Defaults to 1000.



##### sh$ catool convert-certificate -h # (from PEM to pkcs12)
        usage: catool convert-certificate [-h] -ca-c <certificate> -c <certificate>
                                  [-k <key>] [-p <passphrase>]

        optional arguments:
            -h, --help           show this help message and exit
            -ca-c <certificate>  The Certificate Authority certificate used when signing
                                 the keypair we are converting.
            -c <certificate>     The certificate we are converting. If a key is not
                                 specified on its own (using -k), then we will try to
                                 read it from this file as well.
            -k <key>             The key we are converting.
            -p <passphrase>      Passphrase for encrypting the keypairs.
