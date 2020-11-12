---
layout: post
title:  Apexit#01 How to make your site more visible using SSL certificates
date:   2020-11-12 01:00:00
description: Describes what is SSL certificate, the importance of it on your site visibility and how to install it on your site.
permalink: /Apexit-01-How-to-make-your-site-more-visible-using-SSL-certificates/
---
Describes what is SSL certificate, the importance of it on your site visibility and how to install it on your site.

### <strong>Background</strong>
<br/>
Does your website collect the visitor's sensitive information like personal data, credit card information or the passwords? If Yes, this article is for you. The search engine giant, GOOGLE has already made <strong>SSL certificate</strong> Transparency mandatory for its chrome web users.
This simply means that if your website requires the login, you have to get an <strong>SSL certificate</strong> otherwise the GOOGLE will flag it and warn the users before they visit it. Google is making a lot more efforts in making internet traffic secure. If your website is not secured, the search engine giant will flag your website as insecure. It will display a red "X" mark over a padlock in the URL bar.
<br/>

Worried ??

Then don't worry. I will show you how you can configure the <strong>SSL certificate</strong> to your website using very easy steps.
<br/>

<img style="width:100%" src="/img/How to make your site more visible using SSL certificates by mayurkumar.info.png">
<br/>


### What is SSL?
<br/>
<strong>SSL (Secure Sockets Layer)</strong> is the standard security technology for establishing an encrypted link between a web server and a browser. This link ensures that all data passed between the web server and browsers remain private and integral. To be able to create an <strong>SSL</strong> connection a web server requires an <strong>SSL</strong> Certificate.
<br/>

### How do I get the SSL Certificate?
<br/>
<strong>Step 1: Host with a dedicated IP address.</strong>
<br/>

To provide the best security, SSL certificates require your website to have its own dedicated IP address. However, this is an optional step.
<br/>

<strong>Step 2: Buy a Certificate.</strong>
<br/>

There are different levels of validations that "Certificate Authorities" (CAs) offer.
- <strong>Domain Validation (DV):</strong> DV certificates are issued after the CA validates that the requestor owns or controls the domain in question.

- <strong>Organization Validation (OV):</strong> OV certificates can be issued only after the issuing CA validates the legal identity of the requestor.

- <strong>Extended Validation (EV):</strong> EV certificates can be issued only after the issuing CA validates the legal identity, among other things, of the requestor, according to a strict set of guidelines. The purpose of this type of certificate is to provide additional assurance of the legitimacy of your organisation's identity to your site's visitors. EV certificates can be single or multiple domains, but not the wildcard.

You can buy an SSL certificate from CA's as per your requirement. Let's Encrypt and GoDaddy is one of the popular CAs. A certificate is simply a paragraph of letters and numbers that only your site knows, like a really long password. When a user visits your site via HTTPS protocol, the browser will check that password. If it matches, it automatically verifies that your website is valid and controlled by you. The browser will encrypt every request of your website.

<strong>Step 3: Install the certificate.</strong>
<br/>

Copy SSL certificates to your web server. This configuration varies based on a web server. I am showing the configuration for nginx here.

- Copy your certificate to `/etc/nginx/ssl/` folder. Make sure that these files have enough access rights. These files include: OneSSL Certificate, One bundled certificate and one private key to access these certificates.

- Edit your `/etc/nginx/sites-available/defaultfile` as

{% highlight javascript %}
server {
  server_name 'example.com';
  listen 443 ssl;  root /var/www/apps/SR-PLATFORM/shared/public;  ssl_certificate     /etc/nginx/ssl/example.chained.crt;
  ssl_certificate_key /etc/nginx/ssl/example.key;
  ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers         HIGH:!aNULL:!MD5;  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; always";(...)
}
{% endhighlight %}

- Reload Nginx sudo service nginx reload

<strong>Step 4: Redirect all HTTP requests to HTTPS</strong>
<br/>

Add the following code snippet to your `/etc/nginx/sites-available/default` file:
{% highlight javascript %}
server {
  listen 80;
  server_name 'example.com';  # Discourage deep links by using a permanent redirect to home page of HTTPS site
  #return 301 https://$host;  # Alternatively, redirect all HTTP links to the matching HTTPS page
  return 301 https://$server_name$request_uri;
}
{% endhighlight %}
Check out my `/etc/nginx/sites-available/default` file <a href="https://gist.github.com/charusat09/e3043e7bffb0ef285303ba90cbc4b252">here</a>

<br/>

### Troubleshooting
<br/>
If the configurations mentioned above will not work then you can diagnose your problem by using any following formula:

<strong>#1: Use Online services</strong>
<br/>
Go to <a href="https://www.ssllabs.com/ssltest/analyze.html">https://www.ssllabs.com/ssltest/analyze.html</a> and analyse your site's SSL configuration.

<strong>#2: Match Certificate and Private key values:</strong>
<br/>
Log in to your web server. Execute the following command on your web server.
{% highlight javascript %}
openssl rsa -noout -modulus -in example.key | openssl md5
openssl x509 -noout -modulus -in example.chained.crt | openssl md5
{% endhighlight %}
<strong>#3: GoDaddy specific solution</strong>
<br/>
If you are sure about your configurations, values from the above formula #2 are not matching with each other and you have purchased SSL from GoDaddy, then, try with deleting last certificate from gd_bundle-g2-g1.crt file and run the following command on your web server. You need to run this command where your SSL certificates are placed.
{% highlight javascript %}
cat example.com.crt gd_bundle-g2-g1.crt > example.com.chained.crt
{% endhighlight %}
<br/>

### Some useful resources:
<br/>
1. https://www.blog.google/products/chrome/milestone-chrome-security-marking-http-not-secure/
2. https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs
3. https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority

### Conclusion
<br/>
Securing Websites with SSL will help in SEO ranking along with the better user experience. I hope this article will help you. If you have any query then please let me know. I would like to answer.

<strong>Apexit</strong> is technical articles series in which I discuss a real world problem of coding. I develop code for it and share the source code with others.

<hr class="section-divider" />

### Call To Action
<br/>
I am running my two online courses. Join my master 4-week course on "Ignorance of Software Developer and How we can reduce its impact" by clicking <a href="https://forms.gle/ 8RWe119KgGo8mBHF6">HERE</a>.
If you are short on time then join my short course for <strong>"Practical Tips for Ruby on Rails"</strong> by clicking <a href="https://forms.gle/DLKSdhb7xardVreE6">HERE</a>. This course is for all the levels of programming.
You will receive a PDF in your inbox once a week. It's absolutely FREE!! <a href="https://forms.gle/DLKSdhb7xardVreE6">Sign Up.</a>

### About Author
<br/>
Mayurkumar is helping companies to take the stress out of software development and make their business shine. He has more than eight years of experience in designing and building scalable applications using different technologies.

<center><a href="https://mayurkumar.info" target="_blank">mayurkumar</a></center>
