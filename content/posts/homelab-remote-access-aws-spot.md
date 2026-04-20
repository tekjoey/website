---
title: 'Access Homelab remotely using Tailscale and AWS Spot Instances'
date: '2026-04-20'
tags:
  - aws
  - tailscale
categories:
  - Homelab
draft: true

# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
# description: "This is the description"
# disableHLJS: true # to disable highlightjs
# disableShare: false
# disableHLJS: false
# hideSummary: true

# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page

---

I am working through the material for the  AWS Certified Solutions Architect – Associate certification and I strongly believe that the best way to learn a subject in IT is to play it out. So, since I just finished a few sections on EC2, I decided to have a look around and see what I could do.

I was particularly interested in the Spot instances, primarily since their prices are drastically reduced, I can play around with them and get the "full experience" at a much lower cost. At first I wanted to see if I could create a re-usable instance for practicing with Docker and Ansible.
The idea would be to create a dispasable development environment. I found EC2 templates to fit the bill exactly.
As the name "EC2 Template" implies, you can set whatever various parameters you need to replicate each time, and save it as a template to deploy later. I created a template that has mostly standard stuff (AWSI used the User Data section to create a script to install Docker and then install Ansible. 
Durring all this I learned that the User Data script is run as the root user (a fact I would have known before-hand if I had read the [documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#userdata-linux)), so any instalation commands need to be done for the entire machine, not just for the current user.

In the template I specified that the instances should utilize a one-time spot request. This means that the instances will be billed at the Spot price, not the must higher (though still insanely cheep) On-Demand price. 
I don't need these to persist for much longer than an hour or two, their purpoe is just to give me a stable platform to test Docker and Ansible commands.

After I got that up and running, I though "Why am I doing this, when I could just SSH into my own systems?" The idea was frankly much better; while it would be nice to be able to spin up dispsable VMs for testing purposes, I would rather do that on my own hardware, not pay for it on Amazon's.
And since I use Tailscale for everything, all I need to do is install it via the User Data portion and authenticate with an access key, and Voila! The EC2 instance was on my Tailnet. Then I can use the web-based EC2 Connect, and I can SSH into any device in my homelab!
When I'm done I can terminate the instance which will remove both the VM and the EBS volume, and I'm only charged a penny or two for the short time I used the instance.

Now, as long as I can access the AWS console, I can access my homelab! I hope this has been a helpfull demonstration of both the ease of using the AWS console (I did all of this in about an hour, the most difficult part was just figuring out how to install Ansible!), as well as the myriad of problems than can be solved.
When I first started learning about AWS and EC2 specifically, I thought it was really only usefull for webservers and other production level machines. But now I can see the flexability also lends itself to development, jumpboxes, and many other disposable options as well.
