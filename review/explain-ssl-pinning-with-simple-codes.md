# Explain SSL Pinning with simple codes

## HTTPS and SSL Pinning

There are two major factors in an HTTPS connection, a valid certificate that server presents during handshaking, and a cipher suite to be used for data encryption during transmission. The certificate is the essential component and serves as a proof of identity of the server. The client will only trust the server if the server can provide a valid certificate that is signed by one of the trusted Certificate Authorities that come pre-installed in the client, otherwise, the connection will be aborted.

An attacker can abuse this mechanism by either install a malicious root CA certificate to user devices so the client will trust all certificates that are signed by the attacker, or even worse, compromised a CA completely. Therefore relying on the certificates received from servers alone cannot guarantee the authenticity of the server, and it is vulnerable to potential man-in-the-middle attack.

SSL Pinning is a technique that we use in client side to avoid man-in-the-middle attack by validating the server certificates again even after SSL handshaking. The developers embed (or pin) a list of trustful certificates to the client application during development, and use them to compare against with the server certificates during runtime. If there is a mismatch between the server and the local copy of certificates, the connection will simply be disrupted, and no further user data will be even sent to that server. This enforcement ensures that the user devices are communicating only to the dedicated trustful servers.

However, developers must take extra caution in SSL Pinning, When a pinned certificate is expired and the server has updated a new certificate, since the new certificate is definitely different from the pinned one that the client currently has, the clients will not trust the updated certificate and therefore terminate the connection, in which no further communications can be established between clients and servers, so the application is basically ‘bricked’. Therefore, to avoid such situation, it is always advisable to pin the future certificates in the client applications before release.

There are usually two ways we can achieve SSL Pinning in client applications. Pin either the whole certificate or its hashed public key. The hashed public key pinning is the preferred approach because the same private key can be used in signing the updated certificate, therefore we can save the trouble of pinning a new hashed public key for a new certificate, and reduce the risk of app ‘bricking’.

## Codes In Action

I am going to write a simple Android application that communicates with <www.google.com>, and use [Charles proxy](https://www.charlesproxy.com/) as a man-in-the-middle server, and finally demonstrate how an effective SSL Pining can stop man-in-the-middle attack.

Please note that the following codes are for educational purpose and they are not meant for use in production. Please consider using a mature solution from reputable libraries such as [okhttp](https://square.github.io/okhttp/3.x/okhttp/okhttp3/CertificatePinner.html) for your application.

```java
// Read the pinned certificate from local (i.e., assets folder)
val inputStream = context.assets.open("google.crt")
val pinnedCertificate = CertificateFactory.getInstance("X.509")
    .generateCertificate(inputStream)

// Create a request to www.google.com
val url = URL("https://www.google.com")
val httpsUrlConnection = url.openConnection() as HttpsURLConnection

// Establish the connection
httpsUrlConnection.connect()

// Check the certificates and see if one of the server certificates
// matches the pinned certificate
if (httpsUrlConnection.serverCertificates.contains(pinnedCertificate)) {
    // Open stream
    httpsUrlConnection.inputStream
    Log.d("Pinning", "Server certificates validation successful")
} else {
    Log.d("Pinning", "Server certificates validation failed")
    throw SSLException("Server certificates validation failed for google.com")
}
```

The above codes are self-explanatory. The server certificates from `HttpsUrlConnection` are to be checked against with the local pinned certificate, the connection input stream will be open if there is a certificate match, and an `SSLException` will be thrown otherwise.

I use openssl command to download the <www.google.com> certificate and save it as a pinned certificate in the assets folder. Just run this code and you will see message `Server certificates validation successful` in the logcat.

Now fire up Charles, and install Charles root certificate to the test device and configure test device proxy setting to the IP address of Charles service. With Charles root certificate being installed as trusted CA, Charles now acts as a man-in-the-middle, and all the certificate signed by Charles will be trusted by the client system by default. However, since we have pinned the legit <www.google.com> certificate in our demo application, the server certificate validation will fail in this case.

Run the test application with Charles proxy enabled, as expected, you will see `SSLException` with message `Server certificates validation failed for google.com` in the logcat.

## Further reading

The purpose of this post is just to provide an entry-level introduction to SSL Pinning technique, you can read the [original proposal](https://www.paypal-engineering.com/2015/10/14/key-pinning-in-mobile-applications/) from Paypal engineering team, and learn more about the [bug](https://www.synopsys.com/blogs/software-security/ineffective-certificate-pinning-implementations/) that results in ineffective SSL Pinning in Java/Android.

Oct 23, 2018

author: Zhang QiChuan

link: <https://medium.com/@zhangqichuan/explain-ssl-pinning-with-simple-codes-eaee95b70507>