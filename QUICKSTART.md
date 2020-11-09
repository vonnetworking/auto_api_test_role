# Getting Started with Ansible Automation

## Recommended Reference Material
*(strongly recommended...like if you call or email us we're gonna ask if you watched and read this stuff)*

### Ansible Overview Videos

- [Quick Start Video](https://www.ansible.com/resources/videos/quick-start-video) - *We know...we know, they ask for your email, its redhat...they dont spam you too bad*

- Other Intro videos worth watching - *Note: The videos below take about an hour to go through, but they give critical context to the guidelines below*

Part 1 | Part 2 | Part 3 | Part 4
---|---|---|---
[![Ansible Overview Part 1](http://img.youtube.com/vi/icR-df2Olm8/1.jpg)](https://www.youtube.com/watch?v=icR-df2Olm8) | [![Ansible Overview Part 2](http://img.youtube.com/vi/pRZA9ymZXn0/1.jpg)](https://www.youtube.com/watch?v=pRZA9ymZXn0) | [![Ansible Overview Part 3](http://img.youtube.com/vi/jBiueVhDg1Q/1.jpg)](https://www.youtube.com/watch?v=jBiueVhDg1Q) | [![Ansible Overview Part 4](http://img.youtube.com/vi/aeGDc7rCK_0/1.jpg)](https://www.youtube.com/watch?v=aeGDc7rCK_0)
- [Windows Tutorial video (it isn't great)](https://www.youtube.com/watch?v=1mwwBy4XAzo)
- [YAML Syntax](http://docs.ansible.com/ansible/latest/YAMLSyntax.html)
- [What is an Ansible  playbook](http://docs.ansible.com/ansible/latest/playbooks.html)
- [What is an Ansible  role](http://docs.ansible.com/ansible/latest/playbooks_reuse_roles.html)
- [Full Ansible Docs](https://docs.ansible.com) - You can ignore "inventory", and references to testing, we'll handle that in the Travelport Automation framework
- [Ansible Module Index](http://docs.ansible.com/ansible/latest/modules_by_category.html) *- lists all of the supported integrations and operations that are part of the de facto ansible install.*

## Overview

Ansible is the preferred tool for Infrastructure automation and Configuration Management within Travelport.  The Ansible tool uses the YAML syntax to describe the steps needed to configure a piece of infrastructure from its base, to a functioning application.  This repository is a starting point for the creation of a new Travelport [Ansible Role](http://docs.ansible.com/ansible/latest/playbooks_reuse_roles.html).  The purpose of the role in this repository is to define the steps to configure your specific application on top of an existing server.

***Hint: if you don't know what an ansible role is, you should watch the videos above, and read (at least some of) the linked materials - I know... reading...BLEH! but really...its kinda important!***

In general terms, you can think of a role as a set of repeatable steps performed in the creation of a piece of infrastructure.  Simply put, how would a server go from a basic **"Redhat 7 server"** to a **"Redhat 7  - mysql database server"**.  Well, we'd need to:

- download the appropriate mysql package and install it
- update the config files to
  - change mysql's listening port from the default to Travelport's standard
  - allow connections from all external IPs
  - remove the default root user for security purposes
  - add a default read-only credential (perhaps)
- recycle the service and insure its running

These steps that would need to occur on any base server to make it a "mysql database server" with the appropriate environment variables would make up a "rhel7 mysql role".

So what you're trying to accomplish with this repository is to take a **Base OS install on a server** and turn it into a **|insert your application name here| server**.  

## How do I start developing in here?

Here's an overview of the development process that you are currently undertaking

      | Local Workstation |
      |                   | -- 1. Edits are made here and pushed to the
      |                   |       remote on the master branch
      |   (Git clone)     |\
    --|___________________| \
    |         /|\            \
    |          |              \ -- 2. SSH - credentials sent to you when you onboarded;
    |          | - HTTPS       \       runs your ansible role command against specified
    |          |                \      "dev app machine" - also established at onboarding
    |         \|/                \
    | |  Azure GIT Repo  |        | Ansible Control Machine  |
    | |  (Origin Repo)   |------->| (Actually runs the role) | -- 3. This machine connects
    | |  ______________  |        |__________________________|       to the dev machine and
    |                                         |                      applies the role.
    |                                         |
    |           WINRemoting (If Windows)   -  | - SSH (If Unix / Linux)
    |                                         |
    |                                        \|/
    |                            |  Disposable Dev Machine    | -- 4. This is the server you're
    |--------------------------->| - Same OS as your app -    |    trying to make look right
    |                            |____________________________|    it receives the instructions
    |                                                              and when done, should be a
    |                                                              functioning server running
    |                                                              your app
    |
    |-- 5. Developer then SSH's or RDPs to the disposable dev machine
           to test the new role config, please note, the intention is to throw away
           this machine multiple times during testing, to prove that the Configuration
           works as expected

### So to summarize the picture:

1. If you haven't already, clone this repository to a local directory and start editting the Role
2. When you want to test your work, commit your changes and "git push" back to the Origin repo
3. From putty, or your local SSH client, SSH to the ansible control machine using the credentials both specified in your onboarding email - you can also go back to your ServiceNOW request to get this info
4. follow the menu prompts...You'll need to know:
    - The name of the role you're testing
    - The name of the machine you want to run it against
    - Any variables that are needed to execute your role (these can be specified in the menu, or put in a json file on the server using scp - likely pscp if you're on windows)
5. Once the role has run remote to the development server and see if your configuration looks correct
Wash...Rinse...repeat...as needed

***See the "HOW_TO_TEST.md" file in the root of this repo for more info on testing new ansible content...unfortunately, ansible doesn't run on windows, so it makes things interesting***

## This is still overwhelming...Where should I start editting?

Look, we realize that until you play with ansible a bit, the directory structures and their purposes are a bit nebulous, but don't worry.  There are really only a few files you'll likely need to edit to get things rolling, and the **samples** directory in this repo has some good example code for performing simple tasks like, installing a package, updating a config file using a template, or even running a script on a remote machine (though we really discourage this...we'll explain later).

Worth noting that all of the files mentioned below include comments and simple examples to guide you.

*(Side Note: if you really refused to read / watch the docs at the top of this article, at a BARE minimum, please read about [YAML Syntax](http://docs.ansible.com/ansible/latest/YAMLSyntax.html) as it can be frustrating for people not used to whitespace sensitive languages)*

#### tasks/main.yml
> This is likely where the bulk of your work will get done at the beginning.  When executing a role, Ansible looks to this file to figure start actioning the steps of configuration.  Basically, this is where you add the instructions to be executed.

 > **Note that main.yml is the only file read BY DEFAULT out of this directory, if you choose to split tasks into other files (which is often a good idea), you'll have to pull them in via the tasks/main.yml (see the tasks/main.yml in this repo for an example)**

#### vars/main.yml
 > This is where you put variables to be processed by your role, things like lists of packages to install, or values which will remain consistent MOST of the time but can be overwritten as needed.  Remember the point here is 1 role for all your servers.  That means, Dev, Preprod, Prod; they all need to feed off the same role, so there will be values that need to change to make the role universal, generally, they go in here.

 > **Note that "main.yml" is the only file read BY DEFAULT out of this directory, if you choose to split vars into other files, you'll have to pull them in via the vars/main.yml (see the vars/main.yml in this repo for an example)**

#### files/
> This directory should contain any **STATIC** (non-templated) files that are needed by the role.  Things like external scripts, possibly small code archives.

>**Another hint here** - you should NOT embed large installation packages in this repo, it is trivial to host it on a share accessible via HTTP or NFS and pull it down to the server using "get_url" or the "copy" modules.  Try to keep this repo small-ish, and not embed installation media herein as it will consume MASSIVE space on the automation servers and be very slow to work with.  Example:  I should NOT put MYSQL installation MSI or RPG in this repository, I should instead host it on a remote share, or in a Travelport Yum repo and download and install at runtime

#### templates/
> This directory is similar to the */files* directory **EXCEPT**, it generally contains config, and other files that will have variable substitution done in them.  For instance, lets say you're using a config file to designate a remote resource to which you need to connect at start-up, and that resource is different in Dev, Preprod, and Prod.  Well you can put a templated config file with a variable reference that gets set at run-time when you can reasonably expect to know in what environment the server (or servers) you are configuring belong(s).

> *Note: Ansible uses the [JINJA2](http://docs.ansible.com/ansible/latest/playbooks_templating.html) templating engine*, see link for details.

#### README.md
> This file should be used to document what your role / playbook does, this should be the starting point for documentation on your role.  Follow the format of the existing README.md in the root of this repository.  Be sure to include a list of the variables that your role / playbook expects.

## I still don't get it...Please...Help me!

Again, we know this can be a lot, so as you come across questions, or issues, feel free to raise a ticket to us in [ServiceNOW](https://travelportprod.service-now.com/service_management/get_help.do) under category "Other".  And someone from the automation team will be in touch within 1-2 business days.

Godspeed as you embark upon the holy mission of automation.  May the wind be always at your back, and the sun always upon your face (unless you burn easily, then perhaps a brisk cloudy day is better).
