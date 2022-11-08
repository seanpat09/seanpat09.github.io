---
layout: post
title: "Hosting your own Mastodon server"
date: "2022-11-08-T00:00:00.000-08:00"
author: Sean Cuevo
description: My experience setting up my own server

tags:
  - mastodon
  - server
---

## Why Host Your Own Server

With Mastodon becoming more popular, I wanted to see what was invovled in creating my own server. I figured if it didn't work out I could join one of the bigger servers later, either by creating a new profile, or by migrating the profile from my own server. There are a few reasons that I think make hosting your own server a good fit for you.

* You want to have more control over your own content on the internet. While you may be able to move Mastodon servers, only your profile and followers go with you, NOT your posts
* You want to experiment with the technology itself
* You already have a bunch of people you want to follow and that want to follow you. Without this, it may be hard for your content to reach other federated timelines if people aren't able to find you easily.
* You want to host a community and are committed to maintaining and moderating your server.

If you are going to open up registrations, that last piece is important. While people can migrate out of your server should you decide not to maintain the server anymore, I think it's important to be upfront with your intentions so your users aren't suddenly scrambling to find somewhere else to go.

## Still Want to Host a Server?

There are several options for hosting. There are managed products such as [masto.host](https://masto.host) that you can just sign up and they manage the infrastruture for you. Something like [cloudron.io](https://www.cloudron.io/) gives you a little more control in that you own the machine itself (even if just in the cloud), but they manage installing the product on your machine. I don't know much else about these services, but they are available if you are not interested in diving in the command line.

I'm interested in the code itself, but was also intimidated by the idea of installing from source. Luckily Digital Ocean had something to meet my needs.

## My setup experience.

I opted to host on Digital Ocean after I found their 1 click app. If you'd like, you can use [my referal code](https://m.do.co/c/ff49fa0cafe3) to get some Digital Ocean credits for yourself and for me if you don't have an account yet. Once you have an account, go to the [1 click app page](https://marketplace.digitalocean.com/apps/mastodon) and you can create a droplet. When you do, make sure to click a droplet size that has at least 2GB of RAM. I was running into a lot of issues in with a 1GB droplet and am currently running at 80% in the 2GB droplet.

During that setup, make sure to choose to upload an SSH key. Here's [instructions on how to generate an SSH key](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/). I find it's more secure than using a password.

While that is setting up, we'll need a few other things.

First I needed a domain name. I used [name.com](https://name.com) to register mine, but feel free to use whatever provider you want. After getting my domain, I needed to configure the domain to in Digital Ocean. Here's the [link to the official documentation](https://docs.digitalocean.com/products/networking/dns/how-to/add-domains/) from Digital Ocean on how to setup your domain and DNS records.

I also needed an email provider. I tried to set up my your own email server on the droplet with the `sendmail` command, but my email were not being delivered to gmail. Whether it was an issue with my setup or gmail just blocking the emails, I'm not sure, but I eventually gave up. SendGrid has a free tier, which I think should be enough for a single user server. If you're planning to open registrations you might want to shop around to find a provider that fits your budget. If you use SendGrid, follow [the instructions in their documentation to set up an API Key](https://docs.sendgrid.com/ui/account-and-settings/api-keys).

With that all set up, let's ssh into the droplet and install Mastodon. Here are [instructions on connecting with SSH](https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/). Once you are connected, you'll be greeted by the setup wizard and some prompts. Here's what I put to configure my server. When things went wrong I was able to use disconnect from the droplet and reconnect to restart the wizard. The wizard appears to keep running until you succesfully complete it:

```
Welcome to the Mastodon first-time setup!
Domain name: forcedconversation.com
Do you want to store user-uploaded files on the cloud? No
SMTP server: smtp.sendgrid.net
SMTP port: 587
SMTP username: apikey
SMTP password: <my_send_grid_api_key>
SMTP authentication: plain
SMTP OpenSSL verify mode: none
E-mail address to send e-mails "from": Sean <no-reply@forcedconversation.com>
Send a test e-mail with this configuration right now? no
```
I didn't want to configure S3 or some other cloud host, so all user uploaded files will live on the droplet. It's just me so I figured it'd be ok. The only other thing of note is that the SMTP username is literally `apikey`.

After completing those prompts I was able to create my admin account:

```
Great! Saving this configuration...
Booting up Mastodon...
It is time to create an admin account that you'll be able to use from the browser!
Username: squizzleflip
E-mail: sean.cuevo@gmail.com
```

Next I was given my initial password, and prompted to setup Let's Encrypt for SSL

```
You can login with the password: YOUR-PASSWORD-WILL-BE-HERE
The web interface should be momentarily accessible via https://forcedconversation.com/
Launching Let's Encrypt utility to obtain SSL certificate...
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator webroot, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to cancel): sean.cuevo@gmail.com
```

This failed a BUNCH of times - turns out when I set up the DNS recodrs I had a typo in my domain name. A frustrating error, but at least not a complicated one.

After that I had to update certificates:

```
sudo apt install ca-certificates
```

Lastly I needed to add `127.0.0.1 localhost forcedconversation.com` to the file `/etc/hosts`.
And that was it! I was up and running!

## Immediate Upgrades

I started posting on my instance and one of the first messages I got was "How did you manage upgrades?" to which I thought "Upgrades? I just installed it, what would I need to upgrade?" Turns out the Digital Ocean 1-click-app is set to Mastodon version 3.1.3, while the latest release is 3.5.3. COOL.

I connected to my droplet again and in another monitor and opened up the [upgrade documentation](https://docs.joinmastodon.org/admin/upgrading/). Fortunately you can update directly from 3.1.3 to the latest version, but the documentation does warn about making sure to follow any intermediary upgrade steps. For example, if version 3.2.1 has a unique upgrade setup, you'll probably need to do that as well. I opened up all the releases and found I needed to do the following:

* Run `apt install shared-mime-info` as root
* Update ruby to 3.0.3 with `RUBY_CONFIGURE_OPTS=--with-jemalloc rbenv install 3.0.3`
* Run `bundle install`
* Run `yarn install`
* Run `SKIP_POST_DEPLOYMENT_MIGRATIONS=true RAILS_ENV=production bundle exec rails db:migrate`
* Run `RAILS_ENV=production bundle exec rails assets:clobber assets:precompile`
  * The docs don't include the `assets:clobber` part, which caused me to run into [this issue](https://github.com/mastodon/mastodon/issues/15847)
* Run `systemctl restart mastodon-sidekiq`
* Run `systemctl restart mastodon-web`
* Run `RAILS_ENV=production bin/tootctl cache clear`
* Run `RAILS_ENV=production bundle exec rails db:migrate`
* Run `systemctl restart mastodon-sidekiq`
* Run `systemctl restart mastodon-web`

Once that was complete I was finally upgraded the latest version. Hopefully following those steps will make this a little more straightforward if you try this.

Lastly I set the server to single user mode by adding `SINGLE_USER_MODE=true` in the file `/home/mastodon/live/.env.production`. This disable registrations and points the domain directly to my profile.


## What's Next?

Things seem fairly stable right now, but I'm actually not sure what more work will be required. After a few months and some upgrades, I may decide to open up some limited registrations. If you want to try this yourself and have any questions, please feel free to tag in a post on [@squizzleflip@forcedconversation.com](https://forcedconversation.com)! Good luck and happy tooting!
