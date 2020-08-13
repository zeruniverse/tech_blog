---
title: Apply a Free SSL Certificate and Make it Work
date: 2015-05-22T11:00:00+00:00
author: ZerUniverse
layout: post
categories: Web
---

I Just found that my SSL certificate for this domain expires soon, so I applied a new one from StartSSL today. Currently, StartSSL is the only website offering free SSL certificates which are trusted by most web browsers. It's not a good idea to sign a self-signed certificate because web browsers will tell your visitors the website they try to visit is dangerous (like the following graph), which is extremely unfriendly<!--more-->.

<a href="/postimg/12306_ssl_error.png"><img src="/postimg/12306_ssl_error.png" alt="SSL certificate error" width="100%" /></a>

The first step is going to [https://www.startssl.com/](https://www.startssl.com/) and sign up for a free account. This website use certificate to log you in instead of username and password. After you successfully signing up, it will generate a certificate and install to your web browser. **DO REMEMBER TO EXPORT IT** from your web browser and store it at somewhere you think is safe. I lost my old certificate from StartSSL last year, so I can only sign up a new account today with different email.

Simply verify your email address and your ownership of your domain. Then apply SSL/TLS Certificate for your Web Server.

<a href="/postimg/start_ssl.png"><img src="/postimg/start_ssl.png" alt="Start SSL Certificate" width="100%" /></a>

After you click "Continue", it asks you if you would like to generate a private key. **DO NOT LET IT GENERATE PRIVATE KEY FOR YOU!** I don't know why sometimes it doesn't work (certificate do not match private key) while most time it does. But if you are unlucky (doesn't work in your case), you can't apply for Free SSL certificate for the same domain in this 1 year. So we just click *SKIP button*, which means we will generate CSR file ourselves.

Go to your server, install openssl and generate a CSR file after you generate a private key.

```bash
yum install openssl
openssl genrsa -des3 -out server.key 4096
openssl req -new -key server.key -out server.csr
#input your personal information here
#copy the content from your generated server.csr file
```

Paste the content in your server.csr to the website and then choose a subdomain you would like to include in your certificate (e.g. www) while the root domain is already included. Then create a `server.crt` file in your server and paste the certificate you get in StartSSL to that file.

Now your certificate is accepted by most browser, but sometimes Firefox still gives a certificate error. We should do an extra step to avoid this. The following shell script shows this extra step: merge our server certificate with the StartSSL root certificate.

```bash
wget http://cert.startssl.com/certs/ca.pem
wget http://cert.startssl.com/certs/sub.class1.server.ca.pem
cat ca.pem sub.class1.server.ca.pem >> ca-certs.crt
cat ca-certs.crt >> server.crt
```

Now, we get the certificate and our private key. Next we just need to configure our web server to enable SSL.