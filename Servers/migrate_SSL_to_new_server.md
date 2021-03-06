# How to migrate an SSL Cerficate from one host to another

Follow this if you're migrating from one server to another, and want to use the existing keys. First, you have to find the existing cerficates and keys. The easiest way to find this is to use the Apache/Nginx config to find the current SSL/Cert/Key locations.

Eg, `/usr/local/apache/conf/httpd.conf`, search for 'SSLCertificate'

You should see the locations of existing SSL files in the config
```
SSLCertificateFile /var/cpanel/ssl/installed/certs/1296c8cc4ec68f26f49efa.crt
SSLCertificateKeyFile /var/cpanel/ssl/installed/keys/0b816bba1.key
SSLCACertificateFile /var/cpanel/ssl/installed/cabundles/COMODO_CA_Limited_725c0f_1848799.cabundle
```

Download or copy all 3 files to the new server.

Validate the certificate (.crt) and the private key (.key) with the following commands.

```
openssl rsa -noout -modulus -in /<path>/private.key | openssl md5
openssl x509 -noout -modulus -in /<path>/certificate.crt | openssl md5 
```

The resulting hash must match. If they're not matching, the the key doesn't match the certificate. Either try downloading again, or create new ones.

Then copy all the files to the final location on the target server.
```
/etc/ssl/<new_site>
```

Add these files to the Apache/Nginx config.

Eg on nginx config
```
	ssl_certificate 	/etc/ssl/certificates/certificate.crt;
	ssl_certificate_key /etc/ssl/certificates/private.key;
```

Restart Apache/Nginx and check if the new SSL is working. Go to one of the SSL checkers below to check the validity of the cerficiate. If you get an error on this step, then combine the `.bundle` and `.crt` into one file. Put the content from the .crt file on the top of .bundle file. (Because the root certificate must be the last cerficiate on the chain). Also if you get an error about an incorrect order of certificates, open the combined file in a text editor, and move them to be in the correct order.

```
Example
sudo touch mydomain.merged-crt
sudo bash -c 'cat mydomain_com.crt mydomain_com.ca-bundle >> mydomain.merged-crt'
```

After making changes to the Apache/Nginx config or the certificates, don't forget to restart it.

SSL Checkers
- https://certlogik.com/ssl-checker/ (See the order of certificates. It should be your domain (first) > server > root certificate (last)
- https://casecurity.ssllabs.com/analyze.html
- https://tools.keycdn.com/ssl
- https://www.geocerts.com/ssl-checker


Reference
- https://serversforhackers.com/video/testing-and-debugging-ssl-certificates
- https://www.digitalocean.com/community/questions/ssl-library-error-185073780-key-values-mismatch
- http://stackoverflow.com/questions/4658484/ssl-install-problem-key-value-mismatch-but-they-do-match
