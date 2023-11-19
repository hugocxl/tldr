# Security

* `[Doc]` Crypto
* `[Doc]` TLS/SSL
* `[Doc]` HTTPS
* `[Point]` XSS
* `[Point]` CSRF
* `[Point]` Man-in-the-middle Attack
* `[Point]` SQL/NoSQL Injection

## Crypto

The `crypto` module in Node.js encapsulates various encryption functionalities, including OpenSSL's hash, HMAC, encryption, decryption, signature, and verification functions.

There seem to be some issues with Node.js encryption, as algorithms may produce different results compared to other languages (such as Python). The specifics are still being compiled (timeline uncertain), and additional information is welcome.

> How does encryption ensure the security of user passwords?

Client-side encryption increases the cost of decrypting passwords intercepted during transmission by a third party. For games, client-side encryption is used to prevent cheating and cracking. Server-side encryption (e.g., MD5) aims to prevent a database administrator (DBA) or attacker from obtaining plaintext passwords directly after attacking the database, thereby enhancing security.

## TLS/SSL

Early network protocols, initially used only within universities, assumed mutual trust by default. Traditional network communication, therefore, was not designed with network security in mind. In response to this situation, the major browser company Netscape designed SSL (Secure Socket Layer), which serves the following main purposes:

1. Authenticate users and servers to ensure data is sent to the correct client and server.
2. Encrypt data to prevent interception during transmission.
3. Maintain the integrity of data to ensure it is not altered during transmission.

There are three main features:

* Confidentiality: SSL protocol encrypts communication data using keys.
* Reliability: Both the server and client are authenticated, and client authentication is optional.
* Integrity: SSL protocol checks the integrity of transmitted data.

In 1999, due to widespread adoption, SSL became the de facto standard on the Internet. In the same year, the IETF standardized and strengthened SSL, renaming it Transport Layer Security (TLS). Many articles refer to these two interchangeably (TLS/SSL) because they can be seen as different stages of the same thing.

## HTTPS

On the internet, each website resides on its own server. To ensure that you are accessing the correct website and receiving the correct data without interception or tampering, secure authentication is necessary. Authentication cannot be performed by the target website itself; otherwise, malicious/phishing websites could claim to be legitimate. To maintain basic trust between networks on the internet, major companies early on collaborated to promote Public Key Infrastructure (PKI), a framework for authenticating websites using third-party entities.

Public Key Infrastructure (PKI) is a standardized system that provides a secure foundation platform for the development of e-commerce using public key encryption technology. Its basic structure includes a Certification Authority (CA), a Registration Authority (RA), and a Directory Service (DS) server.

RA coordinates and reviews certificate applications from users, sends the applications to CA for processing, issues certificates, and publishes them to DS. In the process of using certificates, in addition to checking the trust relationship and correctness of the certificate itself, confirmation checks are carried out through the generation and publication of Certificate Revocation Lists (CRL) to determine if a certificate has been revoked for any reason. Certificates act like personal IDs, including certificate serial numbers, user names, public keys, and certificate expiration dates.

In TLS/SSL, you can use OpenSSL to generate public/private keys used for authentication during TLS/SSL transmission. However, these self-generated public/private keys can be enhanced by obtaining authoritative third-party certificates (keys) through PKI infrastructure, thereby securing HTTP transmission. Currently, a popular trend in the blogging community is to use [Let's Encrypt to issue free HTTPS certificates](https://imququ.com/post/letsencrypt-certificate.html).

It is important to note that if PKI is compromised, HTTPS is also compromised. Refer to [HTTPS Hijacking - Zhihu Discussion](https://www.zhihu.com/question/22795329) for scenarios where certificates issued by CA organizations are attacked. Generally, browsers will issue warnings when encountering non-authoritative CA organizations (see [12306](https://kyfw.12306.cn/otn/)). However, if you trust a specific unknown organization/certificate under certain circumstances, it may still be susceptible to hijacking.

Additionally, some CA organizations authenticate via email. If a website's email service is compromised, attackers may obtain authoritative and correct certificates from CA organizations.

## XSS

Cross-Site Scripting (XSS) is a code injection method, named XSS to distinguish it from CSS. It was commonly found in online forums in the early days. The cause was websites not strictly limiting user input, allowing attackers to upload scripts to posts for others to view malicious script pages. Injection methods are simple and include, but are not limited to, JavaScript, VBScript, CSS, Flash, etc.

When other users view these web pages, the malicious scripts are executed, leading to attacks such as cookie theft, session hijacking, and phishing. The principle involves using JavaScript scripts to collect information about the current user's environment (such as cookies) and then transmitting the user data to the attacker's server through methods like img.src, Ajax, onclick/onload/onerror events. Phishing deception is common, using scripts for visual deception, creating fake malicious buttons to overlay/replace real scenarios, misleading users.

> Can filtering HTML tags prevent XSS? Please provide situations where it cannot.

Apart from uploading

```html
<script>alert('xss');</script>
```

users can also upload scripts for attacks using image URLs, such as

```html
<table background="javascript:alert(/xss/)"></table>
<img src="javascript:alert('xss')">
```

and evade checks using various methods, such as spaces, line breaks, tabs

```html
<img src="javas cript:
alert('xss')">
```

or encoding conversions (URL encoding, Unicode encoding, HTML encoding, ESCAPE, etc.) to bypass checks

```
<img%20src=%22javascript:alert('xss');%22>
<img src="javascrip&#116&#58alert(/xss/)">
```

### CSP Policy

In the absence of a unified solution, vendors introduced the Content Security Policy (CSP). Taking Node.js as an example, calculate script hashes:

```javascript
const crypto = require('crypto');

function getHashByCode(code, algorithm = 'sha256') {
  return algorithm + '-' + crypto.createHash(algorithm).update(code, 'utf8').digest("base64");
}

getHashByCode('console.log("hello world");'); // 'sha256-wxWy1+9LmiuOeDwtQyZNmWpT0jqCUikqaqVlJdtdh/0='
```

Set CSP header:

```
content-security-policy: script-src 'sha256-wxWy1+9LmiuOeDwtQyZNmWpT0jqCUikqaqVlJdtdh/0='
```

```html
<script>console.log('hello geemo')</script> <!-- Not executed -->
<script>console.log('hello world');</script> <!-- Executed -->
```

Policy directives can be found in [CSP Policy Directives](https://developer.mozilla.org/en-US/docs/Web/Security/CSP/CSP_policy_directives) and [Ruanyifeng
