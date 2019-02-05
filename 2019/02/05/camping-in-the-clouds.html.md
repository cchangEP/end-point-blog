---
author: "Josh Williams"
title: "Camping in the Clouds with Terraform and Ansible"
tags: camps, cloud, development
---

<img src="/blog/2019/02/05/camping-in-the-clouds/image-0.jpg" alt="Base Camp" /><br><a href="https://www.flickr.com/photos/papalars/2773221431/">Photo by Andrew E. Larsen</a> · <a href="https://creativecommons.org/licenses/by-nd/2.0/">CC BY-ND 2.0,</a>

Right, so, show of hands: How many of you work on some bit of web code by doing a `git clone` to your own laptop, developing the feature or bug fix, running through manual testing of the app until you’re happy with it, and off it goes back up to the repo when done? I’m curious, and have a few questions for you.

- Have you ever had a bit of code that worked locally, but didn’t in production because of some difference in systems or dependencies or something else in the stack?
- How do you show off your work to a client or management for approval?
- How do you bring in coworkers to “look over your shoulder“ and help with something, especially ones that are far away?
- On that topic, how do you get a new coworker up to speed if they’re doing development themselves?
- If you’re working on multiple things, do you create multiple clones?
- How’re your backups?

Are you fidgeting nervously thinking about that? Sorry. ☹ But also, check out this little thing: [DevCamps](http://www.devcamps.org/). It’s been an End Point staple for quite a while now, so if you’re reading this you might have heard about it. But long story short, in addition to developing locally this system will spin up your own little environment (a “camp”) on a remote development server which includes a checkout of the code, a separate httpd/nginx and app processes, and a dedicated database with a clone of the data. What’s that all mean?

- Well, figuring that dev server is configured the same, you’ll be working right in a stack identical to production.
- Each camp gets its own port, so you can link your beautiful results to someone and they’ll see exactly what the site will look like when deployed.
- Bring coworkers in to a shared tmux session, just to admire your work.
- And if they just can’t resist working on the same project they can just create their own camp, without worry about installing dependencies and getting the stack operational locally.
- Create multiple camps so you can continue to pretend you can multitask.
- Those server backups your ops team promises are working will automatically preserve your work.
- And unicorns become real.

Recently a client brought their app into “The Cloud.“ Kind of. I can’t use that coveted ‘k8s’ blog post tag yet. It’s still a Java app sitting on a single big VM, but it’s still technically running in “The Cloud.” So we used that opportunity to redo how this “campsite” works and made it cloudy, too. Each “camp” is a dedicated set of VMs that we spin up (and, key point, shut down) as needed.

We do that using [Terraform](https://www.terraform.io/). We’re already using that for this client’s production systems. Terraform has a concept of [workspaces](https://www.terraform.io/docs/state/workspaces.html), which basically clones your single set of systems and lets you apply that multiple times over. You can name the workspace pretty much anything, but we’ve stuck with the traditional camp method of just assigning numbers. Thus, each camp is a numbered workspace, with an identical VM for the app and an identical VM for the database. That gets handed over to Ansible for the same provisioning process.

The configuration split is actually a little more complex than that. We have a ‘prod’ and a ‘dev’ module, which is mostly just different instance sizes. Then, combined with appropriate DNS naming parameters, ‘camp’, ‘staging’, and ‘prod’ configs. Just pull from those as needed. I’ll probably take you into that in more depth later.

But, instead of one big server which is a fixed cost no matter how many camps we’re running, each cloud camp adds to the cost only as long as it’s running. So, like I said above, shutting down the camps is a key point of this. We run them on smaller instances generally, but if you run a number of camps at once that cost adds up fast. Shutting down the camps when they’re not needed saves money. There’s a break-even line somewhere in there, and I’ll leave calculating that as an exercise for the reader. But there’s other benefits, too:

- On traditional camps running on a single system, they all share the same IP address and each gets its own port number. But running on multiple systems with multiple addresses you don’t have to worry about clients with odd egress firewalls.
- Filesystem paths shouldn’t matter inside apps, but sometimes they do, and running camps in their own VMs means the apps can be deployed exactly where they are in production.
- Same with database ports, you get to talk to the system Postgres on its default port, and potentially even on a separate host.
- Also we tend to run our camp servers with SELinux disabled, but you can test things with that enabled in staging environments.

Cloud camps take you the rest of the way there. You can feel like you’re working right on production without *really* feeling all that dirty about it. And, when you’re nearly done, you can control the resources assigned to the VMs; switch the sizing configuration out so that smaller VMs you use for development briefly become production-sized for load testing. Briefly.

I do love making things open source, but in this case I’m not sure there’s really anything to publish; it’s just the same Terraform configuration with just a little bit of tuning to make workspaces work better. But you’ve heard enough from me today, so I’ll write more about all those details later.

There, I hope I’ve planted some ideas in your mind about how you can change that development methodology for the better. On the whole we work better collaboratively, and camp-like systems for development can really assist that. It can feel weird and cumbersome at first, but the clients we have that have adopted it are generally happy campers. It’s still early on, but our cloud-based camp experiment is showing lots of promise, too. Keep it in mind when you’re starting a development project.