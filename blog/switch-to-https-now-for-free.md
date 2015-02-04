If you want to see a delightful lock <img src="/assets/images/blog/https/ssl-0-lock.png" /> next to your domain in your browser's URL bar, switch your site to use HTTPS. Here's how to pay **£0** for the trouble.

Why you should bother doing this:

* SSL's not perfect, but we need to [make surveillance as expensive as possible](http://www.theguardian.com/world/2013/sep/05/nsa-how-to-remain-secure-surveillance)
* For privacy not to be suspicious, [privacy should be on by default](http://www.tbray.org/ongoing/When/201x/2012/12/02/HTTPS)
* And hey, bonus: [more complete referrer information](http://stackoverflow.com/a/1361720/16075) for people visiting from sites already using HTTPS (like [Hacker News](https://news.ycombinator.com))

This post shows how to do your part in building a surveillance-resistant Internet by switching your site to HTTPS. Though it takes a bunch of steps, each one is very simple, and you should be able to finish this in **under an hour**.

A quick overview: to use HTTPS on the web today, you need to obtain a certificate file that's signed by a company that browsers trust. Once you have it, you tell your web server where it is, where your associated private key is, and open up port 443 for business. You don't necessarily have to be a professional software developer to do this, but you do need to be **okay with the command line**, and comfortable configuring **a web server you control**.

Most certificates cost money, but at Micah Lee's [suggestion](https://twitter.com/micahflee/status/368163493049933824), I used [StartSSL](https://www.startssl.com). They're who the [EFF](https://www.eff.org/) uses, and **their basic certificates for individuals are free**.

There are two things that could cost you money. One is that if your site is commercial in nature, they'll ask you to pay for a higher level certificate.

More importantly, if your certificate needs to be revoked someday, StartCom will [charge you a $30 fee](https://www.startssl.com/?app=25#72). While revocation has generally been rare, the [Heartbleed](http://heartbleed.com/) exploit is an example where a huge portion of the Internet had to revoke their keys. For some people who had issued a large number of free certificates, this turned out to be expensive.

Still, StartCom makes getting started with SSL simple and inexpensive. Their website is difficult to use at first — especially if you're new to the concepts and terminology behind SSL certificates (like I was). Fortunately, it's not actually that hard; it's just a lot of small steps.

Below, we'll go step by step through signing up with StartSSL and creating your certificate. We'll also cover installing it via Symbiosis, but you can use the certificate with whatever web server you want such as nginx.

## Register with StartSSL

To get started, [visit their signup page](https://www.startssl.com/?app=11&action=regform) and enter your information.

<img src="/assets/images/blog/https/ssl-1-signup.png" class="block upper border" alt="SSL signup image 1"/>

They'll email you a verification code. They tell you to **not close the tab** or navigate away from it, so just keep it open until you get the code, and can paste it in.

<img src="/assets/images/blog/https/ssl-2-signup-verify.png" class="block upper border" alt="SSL signup image 2 - verify" />

You'll need to wait for certification, but it should only take a few minutes. Once you're approved, they'll email you a special link and a verification code to type in.

That'll bring you to a screen to generate a private key. They're generating you this private key inside your browser, using the ["keygen" tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/keygen). However, **this isn't the key you use to make your SSL certificate**. They're using it to create a separate "authentication certificate" that you will use to log in to StartSSL's control panel going forward. You'll make a separate certificate for your website later.

<img src="/assets/images/blog/https/ssl-3-auth-key-generate.png" class="block upper border" alt="SSL signup image 3 - auth key generate"/>

Finally, they'll ask you to "Install" the certificate:

<img src="/assets/images/blog/https/ssl-4-auth-cert.png" class="block upper border" alt="SSL sigup image 4 install certificate" />

Which installs your authentication certificate directly into your browser.

<img src="/assets/images/blog/https/ssl-5-auth-cert-installed.png" class="block upper border" />

If you're in Chrome, you should see this at the top of your browser window:

<img src="/assets/images/blog/https/ssl-6-auth-cert-chrome-install.png" class="block upper border" />

Again, this is just the certificate that identifies you by your email address and lets you log in to StartSSL going forward.

Now, we need to persuade StartSSL that we own the domain name we want to generate a new certificate for. From the control panel, click the "Validations Wizard" tab, and select "Domain Name Validation" from the dropdown.

<img src="/assets/images/blog/https/ssl-7-begin-domain.png" class="block upper border" />

Enter your domain name.

<img src="/assets/images/blog/https/ssl-8-choose-domain.png" class="block upper border" />

Next, you'll select an email address that StartSSL will use to verify you own the domain name.

As you can see, StartSSL will believe you own the domain if you control webmaster@, postmaster@, or hostmaster@ with the domain name, **OR** if you own the email address listed as part of the domain's registrant information (in my case, that's currently `konklone@gmail.com`). Choose an email address where you can receive mail.

<img src="/assets/images/blog/https/ssl-9-choose-domain-email.png" class="block upper border" />

They'll email you a validation code, which you can enter into the field to validate the domain.

<img src="/assets/images/blog/https/ssl-10-domain-validate.png" class="block upper border" />

<img src="/assets/images/blog/https/ssl-11-domain-done.png" class="block upper border" />

## Generating the certificate

Now that StartSSL knows who you are, and knows you own a domain, you can generate your certificate using a private key.

While StartSSL _can_ generate a private key for you — and their FAQ assures you they use only the [highest quality random numbers](https://www.startssl.com/?app=25#43) and [don't hold onto the key](https://www.startssl.com/?app=25#44) afterwards — it's better to create your own, as StartSSL never sees your private key.

To create a new 2048-bit RSA key, open up your terminal and run:

```bash
openssl genrsa -aes256 -out my-private-encrypted.key 2048
```

You'll be asked to choose a pass phrase. [Pick a good one](http://xkcd.com/936/), and **remember it**. This will generate an *encrypted* private key. If you ever need to transfer your key, via the network or anything else, use the encrypted version.

The next step is to decrypt it so that you can generate a "certificate signing request" with it. To decrypt your private key:

```bash
openssl rsa -in my-private-encrypted.key -out my-private-decrypted.key
```

Now, generate a certificate signing request:

```bash
openssl req -new -sha256 -key my-private-decrypted.key -out mydomain.com.csr
```

Go back to StartSSL's control panel and click the "Certificates Wizard" tab, and select "Web Server SSL/TLS Certificate" from the dropdown.

<img src="/assets/images/blog/https/ssl-12-cert-begin.png" class="block upper border" />

Since we generated our own private key, you can hit "**Skip**" here.

<img src="/assets/images/blog/https/ssl-13-cert-key-skip.png" class="block upper border" />

Then, paste in the contents of the .csr file we generated earlier.

<img src="/assets/images/blog/https/ssl-14-csr-enter.png" class="block upper border" />

If all goes well, it should say it received your certificate signing request.

<img src="/assets/images/blog/https/ssl-15-csr-received.png" class="block upper border" />

Now, choose the domain you validated earlier which you plan to use with the certificate.

<img src="/assets/images/blog/https/ssl-16-choose-domain.png" class="block upper border" />

It requires you to add a subdomain. I added "www" for mine.

<img src="/assets/images/blog/https/ssl-17-choose-subdomain.png" class="block upper border" />

It will ask you to confirm. If it looks right, hit "Continue".

<img src="/assets/images/blog/https/ssl-18-cert-ready.png" class="block upper border" />

<div class="callout">
<strong>Note</strong>: It's possible you'll get hit with a "Additional Check Required!" step here, where you wait for approval by email. It didn't happen to me the first time, but did the second time, and my approval arrived in ~30 minutes. Upon approval, you'll need to visit the "Tool Box" tab and visit "Retrieve Certificate" to get your cert.
</div>

That should do it — your certificate will appear in a text field for you to copy and paste into a file. Call it whatever you want, but the rest of the guide will refer to it as `ssl.crt`.

## Installing the certificate in Symbiosis

If you use [Bytemark Symbiosis](http://www.bytemark.co.uk/symbiosis), here's how to install your certificate. If you don't, check out [the original blog post this was forked from](https://konklone.com/post/switch-to-https-now-for-free#setup-with-other-common-hosts) which includes instructions for nginx and other common hosts.

* **Bytemark** and other servers using **[Symbiosis](https://www.bytemark.co.uk/hosting/symbiosis/)** for Debian support [simple SSL hosting as standard](http://symbiosis.bytemark.co.uk/docs/ch-ssl-hosting.html). Use the key generation guide above and name it `ssl.key`, following the Symbiosis documentation. Likewise, the certificate when generated should be `ssl.crt`. You'll also need [StartSSL's `ca-bundle.crt`](http://www.startssl.com/certs/) renamed to `ssl.bundle`.

Qualys' SSL Labs offers an excellent <a href="https://www.ssllabs.com/ssltest/analyze.html">SSL testing tool</a> you can use to see how you're doing.

**Important:** the StartSSL free Class 1 certificate is good for just <strong>1 year</strong>. Don't forget to renew it before then! Set a calendar reminder or something!
</div>

## Mixed Content Warnings

If your site is running on HTTPS, it's important to make sure all linked resources — images, stylesheets, JavaScript, etc. — are HTTPS too. If they're not, users' browsers will complain. Newer versions of Firefox will [outright block insecure content](https://blog.mozilla.org/tanvi/2013/04/10/mixed-content-blocking-enabled-in-firefox-23/) on a secure page.

Fortunately, pretty much every major service with an embed code has an HTTPS version, and most (including Google Analytics and Typekit) handle it automatically. For others, you'll need to figure it out on a case by case basis.

Where you need to support both HTTP and HTTPS, use [protocol-relative URLs](http://billpatrianakos.me/blog/2013/04/18/protocol-relative-urls/) (starting URLs with `//domain.com`). They're supported just about everywhere except (of course) for IE6.

## Back up your keys and certificates

Don't forget to **back up your SSL certificate, and its encrypted private key**. I put them in a private git repository, and included a brief text file describing every other file, and the process or command that created it. Make sure to **record when your certificate expires**, and **set a calendar alarm** for that date!

You should also **back up your authentication certificate** that you use to log in to StartSSL. StartSSL's FAQ [has instructions](https://www.startssl.com/?app=25#4) — it's a .p12 file containing a cert + key that you export from your browser.

## More resources

* [Eric Mill's original blog post about switching to SSL, for free](https://konklone.com/post/switch-to-https-now-for-free) from which [this is forked](https://github.com/BytemarkHosting/writing/blob/writing/blog/switch-to-https-now-for-free.md). Everything written is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). 
