---
title: Using an Azure static web app with a Google domain
date: "2021-07-23"
description: "A walkthrough of how to use your Google domain for your Azure static web app"
---

When you first create an [Azure Static Web App](https://azure.microsoft.com/en-gb/services/app-service/static/) you get a randomly generated domain - this is the URL for your site and is the domain you will see in the Overview page for your application when viewed in Azure. This is great for playing around with ideas and, as far as I'm aware, you can't delete this - your application will always have this underlying URL. Unless you are planning on leaving your application as a test site it is quite likely that you will want to add your own, more comprehensible domain(s) at some point. The good news is that even the Free tier for Azure Static Web Apps includes 2 custom domains, and you can get 5 with the Standard pricing tier - with both including SSL certificates for free.

You can purchase a domain through Azure App Service Domains or you can use a domain from another registrar. Azure App Service Domains work really well with Azure Static Web Apps but at this time you are limited to .com, .net, .org, .nl, .in, .biz, .org.uk and .co.in - so you will need to go elsewhere if you want a domain other than these. Luckily it is fairly easy to setup custom domains from any registrar.

I had a [Google Domains](https://domains.google/) domain that I wanted to use with my Azure Static Web App so started setting things up. The documentation for setting up a custom domain in Azure is really good - [Set up a custom domain with free certificate in Azure Static Web Apps](https://docs.microsoft.com/en-us/azure/static-web-apps/custom-domain?tabs=azure-dns), the only problem is that Google domains does not let you create an ALIAS record and so you can't actually follow the instructions exactly.

I would recommend following the Azure docs until you hit the the ALIAS part but am going to document all the steps I went through below as well. This is going to assume that the domain isn't in use and all we want to do is set up the www subdomain and the root. So for this site that would involve setting up www.cgrimes.dev and cgrimes.dev.

## Add www subdomain

The first thing we want to do is to setup the www subdomain. I'm going to use www.cgrimes.dev as the example here.

We need to use a CNAME record here to map www.cgrimes.dev to the Azure Static Web App auto-generated name.

### Azure Static Web App

1. Go to your static web app in Azure
2. Go the Custom domains option under Settings
3. You should at this stage just see the auto-generated Azure given domain
4. Click Add and then enter your domain name, so in my case www.cgrimes.dev
5. Click Next and you should be on the Validate + Configure window
6. At this stage we essentially need to prove to Azure that we own the domain we are trying to use
7. Double check the Domain name on this page and make sure the Hostname record type is set to CNAME
8. At this stage it is now showing you what you need to add at Google Domains
9. Copy to your clipboard the Value that is shown for the CNAME record - the auto-generated domain for your app

### Google Domains

1. Log into your Google Domains account, go to My domains and click Manage on the domain you want to use
2. Go to DNS in the left hand option
3. Expand the Custom records section and click Manage custom records
4. Click Create new record and then enter the following
- Host name = www
- Type = CNAME
- Data = This should be the auto-generated domain name for your app, it should still be on your clipboard
5. Save this change

![Google Domains subdomain](./wwwsubdomainexample.png)

### Back to Azure Static Web App

1. You should still be on the Validation + Configure page so click the Add button
2. It might end up stuck on Adding for some time as it relies on the DNS change you made in your Google Domains account propogating
3. You don't need to keep this window open but can close it and in the Custom domains section you will be able to see the status is Adding...
4. Eventually everything will resolve and the status will change to Ready

![Azure static app subdomain ready](./subdomainready.png)

At this stage you should be able to go to your www subdomain (www.cgrimes.dev) and it will resolve to your Azure Static Web App.
