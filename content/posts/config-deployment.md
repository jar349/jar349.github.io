---
title: "Configuration and Deployment"
date: 2022-11-11T11:25:50-05:00
draft: false
---
In this post, I want to think through the next iteration of how I manage my home lab. Because I haven't already
decided what I'm going to do, I suspect that this post will read pretty stream-of-consciousness. Instead of going back
and editing, I'll try to just keep writing. The result may be that what you read at the top ends up not being what I
implement.

## Automated Configuration
The first iteration of my home lab got wiped away by a power outage that caused corruption of the etcd database
backing my kubernetes cluster. It took me a long time to get it all close to back because I did it all by hand. I kind
of deserved it; I knew better. So when I built it all back again, I (ab)used Ansible to do it. I had never used
Ansible before so while I really butchered it, it got the job done and now I have some familiarity with it!
While Ansible helped get my lab setup again in a way that will let me do it again if my cluster ever needs to be
rebuilt, I still have some problems.

First, the way I built my playbooks had the unfortunate side-effect of not being idempotent. For example, I have steps
that generate a self-signed certificate authority and a few certificates for the control plane of my kubernetes
cluster. Every time I run the playbook I get an all-new CA, which is not what I want. Similarly, once my kubenetes
nodes have joined the cluster I don't want them to continually rejoin just because I run the playbook again. Next, my
ansible playbooks are an embarassment. I now understand the purpose of defining roles and then having your playbooks
refer to the roles but I didn't know that when I started. The result is large playbooks that mimic a bash script.
Lastly, the lack of idempotency caused me to stop updating them when I made future changes to the hosts. So now I'm
half back to where I was before: a cluster problem means I have to rebuild certain things by hand. This can't go on.

In my mind, there are four choices for infrastructure configuration management: terraform, puppet, chef, and ansible. I know there's more, I just am not interested for various reasons.

I'm going to straight-up rule out terraform:
- it's way too trendy at the moment
  - trendy technologies tend to evolve more quickly due to the large interest and increasing adoption and I just want
to set and forget my infra management.
- I don't like its DSL
- I'm suspicious of how terraform handles state; I've seen too many people accidentally check it in.
- Hashicorp doesn't mind breaking APIs, documentation is sometimes insufficient, and their engineers are apparently
[struggling to keep up](https://github.com/hashicorp/terraform/commit/6562466c32a8750d7a71a6cc6232e6b5a28fe13a)
 
### Puppet
Usually, I use whatever tech stack we use at work and at GitHub we use [Puppet](https://puppet.com) but I have to admit
that I don't care for it. Here are my reasons:
- I don't like having to learn a new DSL and I usually find them to be restrictive/leaky abstractions
- The proprietary protocol between agents and the server make me nervous about debugging it and securing it
- I find puppet logs to be confusing

### Chef
I was talking with my coworker, [@northrup](https://github.com/northrup), about what I wanted to acheive and he shared
what he does with his own home setup, which is a cheap linode as a control node for [Chef](https://www.chef.io).
Apparently, Chef cookbooks and recipies are written in Ruby instead of a DSL. Also, Chef agents talk to a Chef server
via JSON payloads over HTTP(S). I cannot yet speak about Chef logs, but with the first two items of my list addressed,
I think it warrants consideration!

The downside to Chef is:
- It's still client/server unlike Ansible. This isn't intrinsically bad but you can't deny "just SSH in and run
commands" is simpler. There's good arguments for client/server but none of that matters to me; I'm only managing like
6-8 hosts. I'm aready way more complicated than I need to be because I could handle the workload of maintaining half a
dozen hosts by hand. So from that perspective, simpler is better.
- Since I don't do pxie boots, that means I need to manually install Chef before I can manage a host. I should really
learn how to have my hosts boot from pxie.
- A quick glance at, for example, kubernetes recipies shows that perhaps Chef is becoming abandoned? Something as
popular as kubernetes should almost certainly have very up-to-date recipies for it, right?

### Ansible
I enjoyed learning about ansible. I found the documentation to be thorough enough that I didn't need to go to forums or
google things. There's a module for just about everything. And if you're at all comfortable with BASH, you can totally
make very BASH-like yaml playbooks (ask me how I know :sweat_smile:). Unique to my own situation, Ansible has the biggest thing going for it: it's what I'm already using, just poorly. As I've said before, it's simpler to understand because you run `ansible-playbook` and the command SSH's into your hosts and runs some commands.

There _are_ a few things that are making me feel like I might want to try Chef instead:
- I don't know how to create my own modules and I don't really want to learn how. I'd rather just write ruby because I
know it can do whatever I need and I'm proficient with it.
- If a certain module doesn't do something you specifically want, what are you going to do?
- Refactoring my existing terrible playbooks into roles scares me because I can't be certain that I haven't broken
it unless I test it on my hosts... which is too late.

## Deployment
One of the things that's foremost in my mind is how I want to deploy my configuration to have it applied to my hosts. I
wonder what other people are doing for this? A scheduled GitHub Action that deploys `main` once an hour? As a security
engineer employed by GitHub, I trust that an SSH key I give it is actually secure. But why risk creating an account
that will _have to have_ sudoers and making it accessible to the internet? Instead of going with a push model, I think
I'd rather go with a pull model. I'd rather have the private part of a deploy key in my infra and pull things from
GitHub to a control node that does stuff from there.

I have more to say about deployment, but I've written a lot already and I think this is a good stopping point while I
consider whether I'm going to go with Chef or Ansible. I think I'll at least watch a few YouTube videos about Chef and
read some documentation before I decide.

