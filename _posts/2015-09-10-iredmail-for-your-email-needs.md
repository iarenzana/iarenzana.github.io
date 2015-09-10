---
title:  "iRedMail for your email needs"
date:   2015-09-10 14:30:00
description: "How I set up your email server in 30 minutes"
categoy: Tech
tags: email vps server sysadmin iredmail
---

Your Email Server in 30 Minutes
=====
A History Lesson
------
Once upon a time, I would deal with Sendmail a couple of times a year, which is more than enough time to forget the syntax and nuances of the package between configurations. After a few tears and a lot of sweat I would be able to make it work, I guess.

A few months later, I discovered Postfix and forced my company to move to Postfix as the default [MTA](https://en.wikipedia.org/wiki/Message_transfer_agent). That's how much easier I found Postfix to be.
I still think there's some SMTP elements which are more involved than they probably should, such as  handling TLS, authentication, and relaying with your ISP.
If this is just SMTP, how can I even consider setting up an IMAP server?

Some time later I discovered [iRedMail](http://www.iredmail.org/). iRedMail is based on open source software, in fact, one could argue that iRedMail is a super script on steroids, with Red Bull thrown on top, that sets up an entire email solution with minimal configuration.

Let's go over how to install this on a VPS and get your email server running in 30 minutes (or so).

What you will need
----

You will need a few things before you're up and running.

* A domain. Registrars such as [hover.com](hover.com), [godaddy.com](godaddy.com), or [domain.com](domain.com) offer domains for cheap, you will need one. For the purpose of this article, the domain we will use is *example.com*.

* A VPS. I recommend [Digital Ocean](digitalocean.com) as they have simple to set up VPSs with easy to understand pricing. For the purpose of this article, we will use a $5/month droplet running CentOS 7 and we will call it *comms01*.

Prep work
--

We will need to create an 'A' record for comms01 on the example.com domain. To do this, you will need to go to your registrar's DNS panel and create an A record called comms01.example.com with comms01's public IP. 
You will also want to set up an MX record pointing to comms01.example.com in order to receive email.

Install iRedMail
--
SSH into your brand new server:

`ssh root@comms01@example.com`

As a sysadmin I can't recommend you log into your machine as root, but for the sake of time, I will not go over how to create a local, unprivileged user.
As of this writing, the latest version of iRedMail is 0.9.2, so this is what we will use. Let's start by downloading and uncompressing the package:

```bash
wget https://bitbucket.org/zhb/iredmail/downloads/iRedMail-0.9.2.tar.bz2
tar -xvjpf iRedMail-0.9.2.tar.bz2
```

Now we just simply need to run the installer. The installer will take care of the dependencies and the configuration:

```bash
cd iRedMail-0.9.2
bash iRedMail.sh
```

At this point, a graphical interface will come up, this interface will contain most of the configuration you will need to do.  

First, you will be asked for the mail storage directory,  you can just leave it under `/var`.

For web server, feel free to use either one. In my case I will pick Nginx because in my experience it's lighter than Apache and that will be key when using a VPS with 512MB of RAM (as I will point out later, pick a 1GB of RAM VPS ($10/month) if you want to run ClamAV for antivirus protection).

![IMAGE 1](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/09/20150910-01.png)

For the database engine, I'll pick Postgres just because I like it, but MariaDB would be just as good of a pick. I would strongly avoid LDAP unless you're already familiar with it just because it's not as easy to troubleshoot.

![IMAGE 2](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/09/20150910-02.png)


Enter the database password (I'm not telling you what to enter here).

![IMAGE 3](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/09/20150910-03.png)


Enter the domain name *example.com*

![IMAGE 4](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/09/20150910-04.png)


Enter the administrative password, once again, I'm not telling you.

![IMAGE 6](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/09/20150910-06.png)


Feel free to install the default additional components, as we will find them to be useful in the future and, for the most part, they are not very resource intensive. However, I would personally avoid ClamAV as that is actually very heavy application, I would encourage you to get a 1GB of RAM VPS if you want to run the antivirus.

![IMAGE 6](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/09/20150910-06.png)

After this, the installation will begin. You will see dovecot (IMAP), postgres (DB), roundcube (WebUI), postfix (MTA) and other Python and Perl packages being installed. Self-signed SSL certs will be generated, and the configuration will take place. This process, as you can see is pretty much hands free. Just make sure you say 'YES' when asked to open up SSH on the firewall, otherwise you'll get locked out and will need to use the console to get back in.

Once it's done, you will get a summary with the information on how to log in, manage accounts, etc. Reboot the machine.
```bash
shutdown -r now
```

After the reboot, go to `https://comms01.example.com/mail` and you should get a certificate error. This is because your certificates were not signed by a recognized authority. Since you signed the certificate, you should probably trust it, don't you trust yourself? If you get a connection refused, look at the *Caveats* section below for some common problems and how to fix them.

Caveats
--

* I've found that, with CentOS 7, the WebUI will not come up as expected. This is because Nginx will not start by default, just issue the following command to get it up and running:

```bash
systemctl enable nginx
systemctl start nginx
```

* You will need to add firewall rules for Nginx:

```bash
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --zone=public --add-port=443/tcp
```

Summary
--

As you can see, it's easy to run your own email server. I've missed some very important points, such as redundancy, replication, failover, and mail reputation. I will go over some of these topics in the future and how to set up [Mailgun](mailgun.com) as your outgoing SMTP service to improve your email reputation and deliverability.