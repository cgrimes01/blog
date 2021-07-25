---
title: Using an Azure static web app with a Google domain
date: "2021-07-25"
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
4. Click Add and then enter your domain name with the subdomain www, so in my case www.cgrimes.dev
5. Click Next and you should be on the Validate + Configure window
6. At this stage we essentially need to prove to Azure that we own the domain we are trying to use
7. Double check the Domain name on this page and make sure the Hostname record type is set to CNAME
8. At this stage it is now showing the CNAME details that you need to add at Google Domains
9. Copy to your clipboard the Value that is shown for the CNAME record - the auto-generated domain for your app

### Google Domains

1. Log into your Google Domains account, go to My domains and click Manage on the domain you want to use
2. Go to DNS in the left hand option
3. Expand the Custom records section and click Manage custom records
4. Click Create new record and then enter the following
- Host name = www
- Type = CNAME
- TTL = Leave at default of 3600
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

## Configure root domain

### Azure Static Web App

1. Go back to the Custom Domain section for your Azure Static Web App
2. Click Add and then enter your domain name, so in my case cgrimes.dev
3. Click Next and you should be on the Validate + Configure window
4. Double check the Domain name on this page and make sure the Hostname record type is set to TXT
5. Under the Value column there should be Generate Code button that you need to click
6. Once it has been generated click the copy icon
7. The status should now show up as Validating...

### Google Domains

1. Go back to the DNS options for your site in Google Domains
2. Go back to the Manage custom records section and click Create new record again
3. Enter the following information
- Host name = [Leave empty]
- Type = TXT
- TTL = Leave at default of 3600
- Data = Paste in the value you just copied from Azure
4. Save this change

### Azure Static Web App

You don't actually need to do anything here. As with the subdomain though we are waiting for the process to complete and this might take a while but eventually the Custom domain you have just entered will change from Validating to Ready.

## Setup domain forwarding

This is the part where we need to deviate from the docs. The docs ask us to setup an ALIAS record in our DNS so that our root domain (cgrimes.dev) is pointed to the auto-generated domain for out static web app. Google Domains does not support ALIAS records and you can't add CNAME records for the root domain - which is the alternative mentioned in their docs.

So what we can do instead is to setup domain forwarding so that cgrimes.dev is always forwarded to www.cgrimes.dev. This is something that people often do anyway when choosing what their default domain is - www vs non-www. In this instance we are going with www as our default and we can set it up so that the paths and SSL are retained. This should lead to a seamless experience and also not negatively impact SEO.

### Google Domains

1. Go back to the DNS options for your site in Google Domains
2. Expand the Domain forward section and select Manage
3. Enter the following information
- Forward to = www.cgrimes.dev
- From = cgrimes.dev
- Redirect type = Permanent redirect (301)
- Path forwarding = forward path
- SSL = On
4. Save these changes
