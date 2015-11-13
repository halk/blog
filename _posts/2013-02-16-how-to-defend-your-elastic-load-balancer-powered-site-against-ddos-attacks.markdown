---
layout: post
title: "How to Defend Your ELB Powered Site Against DDoS Attacks"
date: 2013-02-16 15:32
comments: true
tags:
    - devops
---
<img src="/img/post/ddos_attack.gif" width="250" height="188" class="right" />
**This article is written for users of [Amazon Web Services (AWS)](http://aws.amazon.com/) running their web services behind an [Elastic Load Balancer (ELB)](http://aws.amazon.com/elasticloadbalancing/).** Although not much different to usual DDoS mitigation, there are some small quirks I learned the hard way.

## What is a Distributed Denial-of-Service (DDoS) attack?

A [Denial-of-Service (DDoS)](http://en.wikipedia.org/wiki/Denial-of-service_attack) is basically an attempt to make the victim's web services unavailable for its intended users by flooding its infrastructure with such an intense load that the infrastructure cannot respond to legitimate traffic anymore. In most cases it's a distributed attack executed with the help of [Botnets](http://en.wikipedia.org/wiki/Botnet#Illegal_botnets). Botnets are usual personal computers in people's homes being compromised with malicious software, making them zombies of a controller. The controller can then use these zombies to do any kind of actions even without the PC owner's awareness. A person or organization, who wants to harm the victim, can partner with self-declared hackers and rent a Botnet to perform DDoS attacks. It's not even expensive.
<!-- more -->
## How To Defend A DDoS Attack In Theory?

Where conventional hosted customers have multiple hardware and software choices, those in the cloud are more or less depending on [Software as a Service (SaaS)](http://en.wikipedia.org/wiki/Software_as_a_service) solutions.

The basic idea is to route all traffic through their infrastructure which cleans legitimate from dirty (attack) traffic. This is most of the time on demand, means you enable this routing only when being under attack. Your infrastructure only accepts traffic from theirs. This is called **DDoS mitigation**. There are many mitigation providers. I have worked with [Cloudflare](https://www.cloudflare.com) and [Neustar SiteProtect](http://www.neustar.biz/enterprise/ddos-protection/what-is-ddos-protection) so far.

**Cloudflare** is a ready-to-go and free or relatively low priced solution (depending on your needs). Once signed up you can directly set everything up to start a mitigation, which is good if you are already under an attack. However Cloudflare has a few cons such as that it does more than what you ask for, that you need to transfer the whole DNS zone (so it will be your DNS server), that you need to update the A record via cron and finally that it has no phone support.

**Neustar SiteProtect** is an Enterprise solution and therefor priced accordingly. You usually have some discussions with the Neustar staff for them to understand your services. Normally it can take up to 72 hours until they have provisioned your account. But they also offer an emergency service where you can just call their number and they'll help you from there. For that they will charge you an emergency fee. You pay a monthly fee and for each DDoS mitigation (72 hours window) they provide. SiteProtect has no con actually, the mitigation is very seamless. The only question is if you can afford it or not.

<h2 id="howdoiknow">How Do I Know If I Am Under Attack?</h2>

Generally if your site does not respond anymore and you make sure it is not only your network or ISP. You may use some services such as [Pingdom](https://www.pingdom.com/) or [NewRelic](http://newrelic.com/) telling you that the site is down. Make sure that no recent changes such as DNS, Firewall and Load balancers were made which could cause the downtime. Double check it. Have a look at [AWS Status Page](http://status.aws.amazon.com/) for any outage on their side.

Usually with Elastic Load Balancer, your servers will operate normally (actually with less load) as ELB simply stops working when under attack. ELB is an auto-scaling service, based on the load to your ELB and the overall load on the server where your ELB runs, it may move to another server to handle the load. But if you suddenly get crazy huge requests in no proportion to your clean traffic, then ELB won't have enough time to scale and it will just stop working like a goatish child. There won't be any hints of a DDoS attack on the servers directly. When you log into AWS Console and go into the details of your ELB, you should see "Out of service" for all [Elastic Compute Cloud (EC2)](http://aws.amazon.com/ec2/) instances connected to your node. 

## How Do I Prepare Myself Against DDoS Attacks?

It is best for your business to prepare yourself in advance. This will ensure quick recovery and continuous uptime. You never know when you get attacked, but it can be during peak time. Now imagine how much money you would loose being offline for more than one hour during peak time. These steps are also needed during an attack if not prepared before. I highly recommend to run a test, preferably over night, once everything is prepared to get confident with the whole process.

### 1. Make Sure That EC2 Instances Are Not Open To Public

Having your Elastic Compute Cloud (EC2) servers open to public is bad practice anyway, but you may have forgotten it. Do not open ports such as 80 or 443, actually no port at all, to public. Best practice is to have your EC2 instances in a [Virtual Private Cloud (VPC)](http://aws.amazon.com/vpc/). You may also think about having SSH open only to one (non-critical) of the servers and jump to the other hosts from there. This is not a single point of failure, as you can easily open the SSH port to other servers in case via VPC configuration in the AWS Console. Access to your web services should only be via ELBs.

Make sure you have no 3rd party web services going directly to specific servers, as they would fail after you disable ports to your servers. This may be the case if you have a specific backend server which accepts requests e.g. from payment gateways or ERPs. You may think of IP whitelisting them for that server and port as a temporary solution.

### 2. Create Alternative Elastic Load Balancers

If you have read the chapter "How Do I Know If I Am Under Attack?" above, then you already know that your ELB will simply stop working when under attack. You have no control when it will recover and work again. The best thing is to create alternative ELBs pointing to the same servers but allowing only access from the mitigation service. For this you need to create a new security group with the IP range of the mitigation service being allowed to access ports 80 and 443. If you have more than one load balancer such as for back end, front end, static you need to have alternative ones for them too.

### 3. Get Yourself A DDoS Mitigation Service Provisioned

You need to decide which mitigation service provider you want to work with and set up an account with them.

#### 3a. Cloudflare

1. Sign up for a Cloudflare account.
2. Add your website by entering the domain name. If you have more than one domain, you may add it later in "My websites".
2. Cloudflare will scan your DNS and successfully import it. No record should be active (have orange cloud icon on the right). Confirm it.
3. Have a look at their [plans](http://www.cloudflare.com/plans). If you depend on HTTPS/SSL, then you need to get at least the Business plan. Choose "Essentially Off" as Security Level.
4. ELBs have no IP address, only hostnames which is fine for CNAME. However your root domain requires at least one A record (direct IP address) for correct resolution. An ELB's IP can change anytime without notice. So you need to script that it gets updated at Cloudflare too (Route53 was doing it automatically for you). [Copy this script](https://github.com/bundan/CloudFlare-ELB-Updater/blob/master/cf_elb_updater.py) to one of your servers, configure the variables in the top ("ELB" variable should be set to the original ELB's hostname where your root domain should point to), do a test run and set it up as cron which runs every minute. 
5. If you need HTTPS/SSL support make sure SSL is [set up correctly](https://support.cloudflare.com/forums/21317627-SSL-at-CloudFlare).
6. Change your DNS servers to Cloudflare in your Domain Registrar's configuration panel.
7. [Customize](https://support.cloudflare.com/entries/22055667-Where-can-I-customize-the-challenge-page-shown-to-my-visitors-) challenge pages to have your branding. Challenge pages are shown   to suspicious traffic. If you choose "I'm under attack" Security Level, then a redirect page will be shown to every visitor once a week.
8. Check your Cloudflare settings that it does only what you want it to do. Cloudflare likes to do more than just DDoS protection such as caching, minimizing JS/CSS, e-mail address obfuscation and tons more.
9. Make sure that your alternative load balancers pass through only traffic from Cloudflare.

#### 3b. Neustar SiteProtect

As mentioned above, the DDoS mitigation is supposed to work On Demand, means you only enable it when needed. This enable step is basically pointing your (sub)domains to their infrastructure by changing the DNS. If you have a high TTL, let's say an hour, you are down for an hour until mitigation starts. A good thumb rule is 5 minutes (TTL of 300 seconds).

When having the phone or email conversation with the SiteProtect team, they will ask you which (sub)domains you want to mitigate. Provide all those you want to in combination with their ELB hostnames. While doing so make sure you give the alternative ELB hostnames, not the original ones, as you want SiteProtect to send traffic to the alternative ones only.

**Once provisioned, you will get a combination list of (sub)domain to IP, called Virtual IPs.** When under attack, you will need to point these (sub)domains to the respective IPs. Make sure you have them ready anytime. Ask SiteProtect team for their IP ranges, so you can configure your alternative ELBs security group to only allow traffic from them.

## I'm Under Attack, What Shall I Do?

If you have completed the preparations above, then you are lucky. You are ready to teach the attackers a lesson.

1. [Make sure](#howdoiknow) that it is a DDoS attack.
2. Go to the [AWS Console](https://console.aws.amazon.com) and change security groups of the original load balancers to same as alternative ones. The attack might be done via IP instead of  domain. Although ELB should not react to such an intensive load, we still want to make sure that no traffic comes through them anymore.

### If You Use Cloudflare

1. Change the ELB configuration in the cron, which updates the A record for root domain in Cloudflare, to the hostname of the alternative ELB where the root domain should point to.
2. Go to "DNS settings" for your website in Cloudflare and change all CNAME records to point to the alternative ELBs.
3. Enable "On Cloudflare"/Active option for all CNAME and A records by clicking on the cloud icon on the right so it becomes orange.
4. Go to "Cloudflare settings" for your website in Cloudflare and change "Basic Protection Level" to "Low".
5. Wait for your site to be accessible.
6. Feel free to increase the protection level if you think it's not enough.
7. Monitor clean and dirty traffic in Cloudflare Analytics.

### If You Use Neustar SiteProtect

1. Take an export of your DNS zone in Route53.
2. Remove all CNAME records of (sub)domains you want to protect.
3. Add new A records based on the Virtual IP list you got from SiteProtect team.
4. Wait for the DNS changes to take effect.
5. Your site should be back.
6. Notify the SiteProtect team about the incident. Set a timer so you don't miss to disable before the 72 hours mitigation window ends.
7. Monitor clean and dirty traffic on the [SiteProtect Portal](https://siteprotect.neustar.biz).

## I Think The Attack Is Over, What Shall I Do To Rollback?

Congratulations, you've made it. Please write a comment here about your experience so I can update this article if needed. To disable follow the following instructions, it is far easier than enabling.

1. Change the security group of original ELBs back so they accept traffic from all.

### If You Use Cloudflare

1. Change the ELB configuration in the cron, which updates the A record for root domain in Cloudflare, to the hostname of the original (not alternative) ELB where the root domain should point to.
2. Go to "DNS settings" for your website in Cloudflare and change all CNAME records to point back to the original ELBs.
3. Disable "On Cloudflare"/Active option for all CNAME and A records by clicking on the cloud icon on the right so it becomes gray.
4. Go to "Cloudflare settings" for your website in Cloudflare and change "Basic Protection Level" to "Off".

### If You Use Neustar SiteProtect

1. Import the DNS zone export file you've exported during enabling in Route53. If you have difficulties, replace the new A records with CNAMEs by hand so that they point to the original ELBs.

## Conclusion

Believe me, having a DDoS attack is not usual. When having one it is an exceptional time. But preparation counts here. Make sure you explain the whole thing to your peers and seniors. With every DDoS attack you have you get more confident with it. Cut yourself some slack.

I wish you good luck with it.

*Please note that I don't guarantee correctness and up-to-dateness of this article. It is at your own risk to question, research and execute the tips given in this article.*