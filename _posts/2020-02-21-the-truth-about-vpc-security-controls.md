---
layout: post
title: The Truth about VPC Security Controls
excerpt: GCP's VPC Service Controls protection is often described as a virtual firewall
  for your GCP projects.  That metaphor quickly breaks down if you're trying to actually
  add it to your GCP projects. I learned that the hard way.
tags:
- Technical
- GCP

---

GCP's [VPC Service Controls](https://cloud.google.com/vpc-service-controls) protection is often described as a virtual firewall for your GCP projects.  That's a useful mental model for your company's decision makers to think with, but the analogy quickly breaks down if you're an engineer trying to actually implement VPC-SC protection for your GCP projects.

I learned that the hard way.

Here I'll describe just what VPC-SC is, why it was needed, and a big mistake I made which you reeeally want to make sure you avoid.

# What Problem is VPC-SC Supposed to Solve Anyways?

The core problem VPC-SC is meant to solve is to protect companies against **data exfiltration**.  That's a fancy way to describe preventing adversaries from copying your private data off of your servers and onto their own.

![](/media/2020-02-21-vpc-sc-truth-1.png)

You can setup many types of protections to prevent attackers from getting access to your machines (let's strong with decent passwords, shall we?), but following the principle of [defense in depth](https://www.imperva.com/learn/application-security/defense-in-depth/), enterprises with more critical data protection requirements sometimes need an additional layer of protection which says "Even if someone hacks into our machines, joke's on them. They still can't steal our data!" That's the core problem we're trying to solve here.

This problem has been around for a while. How was it solved in the pre-cloud era?

Back then, companies would have a private corporate network. All employee computers would be inside that network, and that network would have a set of strict firewall rules setup to prevent data from being passed to the outside world.  The firewall would only be opened up for a few, strictly vetted sites that employees had real business need to access.

This way, even if an attacker gained access to a machine they still would not be able to send a copy of the data to themselves.  The firewall would block any such attempts.

![](/media/2020-02-21-vpc-sc-truth-2.png)

Now what changes if you move to the cloud?  Not too much actually!

You can define firewall rules for your cloud VMs as well and you can set those up to match very similar rules to what your local company network had.

![](/media/2020-02-21-vpc-sc-truth-3.png)

But what if you wanted to use any of the other mouth watering array of services GCP offers?  Things like blob storage, pub sub, or juptyer notebooks as a service?

![](/media/2020-02-21-vpc-sc-truth-4.png)

Those services live outside the reach of your firewalls.  They're run on servers which are the entry point for resources owned by all GCP customers.

The usual solution of opening up a firewall hole to allow traffic just to those services doesn't quite work.  Take Google Cloud Storage (GCS) for example, it's specifically designed to store data so it would be trivial for an attacker to take your secret data, push it to their own GCS bucket and then download the data from there at their leisure.

![](/media/2020-02-21-vpc-sc-truth-5.png)

Some companies worked around this problem by self-hosting their own versions of these services within their private corporate networks.  That's an option, but it's expensive. You have to run, debug, maintain, and upgrade both the software and servers, all by yourself.  Your dev ops team is cursed to toil away in the kitchen making boiled chicken, while longingly looking upon the [duck confit](https://images.app.goo.gl/KedFe6Tui7kKR7cc7) delivered to their neighbor's doorsteps.

VPC-SC provides a better way.

# VPC-SC to the Rescue

At a high level, VPC-SC controls are settings you can define across your projects that help you eat from the GCP buffet without worrying about [an alligator eating you](https://youtu.be/YbFnrkeH7IA?t=11).

These settings are called your VPC-SC perimeter.  You create and enable a VPC-SC perimeter (the virtual firewall) by defining three things:

* A list of GCP services that you want to set on lockdown mode
* The specific projects of yours that the lockdown mode should be applied to
* Exceptions you want to make to the above policy (via [Access Context Manager](https://cloud.google.com/access-context-manager/docs)), but let's ignore this category

Let's call the set of projects in the perimeter P. By adding GCP services and projects to the perimeter you're saying you want to see the following behavior implemented by each of those services:

* Data Ingress limits: the service shouldn't allow anything outside the P projects in to read/write data to P. So data is never read from outside the perimeter.
* Data Egress limits: The service shouldn't allow projects P to write data anywhere _except_ in one of the P projects. So data never gets written outside the perimeter.

The above two clauses put together tell GCP Services to make sure your data stays inside your projects.

So you've combined the traditional firewall + DNS rules that were used to protect computers, and added a layer of VPC-SC service level protection on top of that.  These combine to protect your resources from data exfiltration.

How? Now, the only way to access your project's data is from your VM, which is protected by your custom firewall rules.  Even if an adversary manages to get a hold of your GCP credentials they still wouldn't be able to steal your data. They would first have to breach your firewall to enter your VPC-SC perimeter! (Defense in depth!)

![](/media/2020-02-21-vpc-sc-truth-6.png)

And that's how you get the virtual firewall!

_The more astute among you may have noticed that the VPC-SC controls actually have nothing to do with your VPC network.  I never asked, but I assume the name VPC-SC was chosen for marketing purposes to make the 'virtual firewall' analogy easier to accept_

# The Big Misunderstanding

There's one horrible assumption that people tend to make when setting up VPC-SC networks.  I made this mistake too and spent a long time trying to debug it before I realized what I was doing wrong.

Notice how you're adding services into your VPC-SC perimeter to enable the "lockdown mode"?  What happens if you don't add some service to the perimeter?  My naïve assumption was that it would be blocked and inaccessible to my VMs.

That is absolutely, completely, 100% not the case.

In fact, if you don't add a service to your VPC-SC perimeter then you've basically left the route open for your internal VMs to send data to that service but neglected to lock down that service itself. Chickens are in the hen house, but the door is left wide open.

Here comes the fox ready to exfiltrate your data!

![](/media/2020-02-21-vpc-sc-truth-7.png)

In case that wasn't clear: *You must to **opt-in** to **every__ service ***_that you want to have locked-down_. Opting-out means **zero** protection.

There were probably good technical reasons for setting this up as the default behavior. The first guess which comes to mind would be that not all GCP services support the VPC-SC lockdown yet. But boy, you can really get caught with your pants down if you don't see this one coming.

How do you protect yourself against this?  First step is to include all services that you actually plan to use in your VPC-SC perimeter.  Then, block all the services that you do not plan to use.  I'm not sure what the recommended way to do this is (probably something at either the DNS or the IAM levels) but that's what you'll want to do.  Otherwise you'll have left a hole wide open that's big enough for an adversary to drive a Humvee full of chickens through.

Please don't do that

# The Power is Yours

There's a quick overview of what VPC-SC controls offer you and what they don't.  Hope this helped you get a better understanding of how to set it up.

If you have any thoughts, war stories, or corrections, shout out in the comments and let me know!