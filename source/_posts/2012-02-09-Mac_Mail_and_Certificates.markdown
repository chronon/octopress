---
layout: post
title: Mac Mail.app and S/MIME Certificates 
description: "Quick steps to setting up a free email certificate on a Mac"
date: 2012-02-09
categories: os-x
---

There is lots of information out there on how to get and install a free email certificate, but here it is again condensed and step-by-step. When complete, you'll be able to sign and encrypt email messages with others who are making use of personal certificates.

### Get a free certificate:

1. Find a free certificate provider, such as [InstantSSL/Comodo](http://www.instantssl.com/ssl-certificate-products/free-email-certificate.html). A list of others is [available here](http://kb.mozillazine.org/Getting_an_SMIME_certificate).
2. Complete the form, recieve email confirmation.
3. Open the link in the email with **Firefox**, you should see an alert about certificate installed.

### Export certificate to a file:

1. In Firefox, go to Preferences -> Advanced -> Encryption and push View Certificates.
2. Click Your Certificates, click on the certificate (line with a serial number).
3. Click Backup, select Desktop, and enter a password.

### Import certificate to your keychain:

1. On your desktop, you should have something like comodo.p12, double click it.
2. Enter the password you just set and import the certificate.
3. Save your exported certifcate somewhere for backup, remember your password.

### Using your certificate:

1. Quit Mail.app if it's running, open Mail.app.
2. Compose a new message, notice two new buttons (a lock and a check).
3. When the check box is checked, your public certificate will be sent to the recipient. 
4. Once your recipient has your public certificate, you can sign and encrypt (lock button) messages.
5. Any time you sign a message (check button checked), you are sending your public certificate.

### Using your certificate with iOS devices:

1. Check out this [very good blog post](https://catbrainblog.wordpress.com/2011/12/12/using-smime-on-ios-devices/).
