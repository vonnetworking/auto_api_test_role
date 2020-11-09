Coding Best Practices for Ansible Roles and Plays
===============================

Overview
--------

This doc will give some high-level standards and best practices to use when developing new Ansible roles and playbooks within the Travelport Automation Framework.  Because Ansible is essentially just a collection of YAML files and supporting config data, this will be part coding style guide and part execution philosophy.

This doc is adapted (heck, in some places straight-up copied) from this [Ansible Best Practices: The Essentials](https://www.ansible.com/blog/ansible-best-practices-essentials) article.  We've pared some of it down a bit, and added some other stuff that really bugs us, but we encourage you to read the article in its entirety.

Guiding Principle - Clarity over Cleverness
-------------------------------------------

### Summary
1. **Name Plays and Tasks in a meaningful way** - Use the ``name`` attribute, and be verbose

2. **Use meaningful variable names** - prefix variables with role or play specific strings, and always use lower_case_underscore_separated var names, no CamelCase, No dashes

3. **Use Native YAML Syntax** - use long-form and avoid short-hand, it improves readability

4. **Use Modules Avoid Run Commands** - Avoid just running a bunch of scripts you already have, it causes problems with repeatability and complicates your plays and roles

5. If you MUST run external scripts
  - KISS - Keep It Simple ~~Stupid~~ Sweetheart - the simpler, the better - we know sometimes complexity is necessary, but make sure its necessary

  - FAIL LOUDLY - A script needs to raise errors if something goes wrong, the worst thing we can do is have a silent failure.  Ansible will error on non-zero exits.

  - DOCUMENTATION IS NOT AN OPTION - Ansible is meant to be explicit and self-documenting, your powershell or perl nightmare is probably not, review it before submitting, and beef up the docs.

  - ONLY RUN IF NEEDED - Think about assertions you can use to determine if rerunning an outside script is necessary



### Clarity over Cleverness


> TL;DR Ansible is meant to be explicit, self-documenting, and as human-readable as possible.  When you're doing anything, always choose clarity and ease of support for the next guy over doing something clever or unclear.  Name all of your plays and tasks in a meaningful way, and be verbose with your variable names, its ok, we promise.

Ansible plays and roles, are just YAML.  This means that the code should be very easy to pick up and understand even for someone with limited development or operations background.  Optimize your plays and roles for readability and simplicity.

### “Name” Your Plays and Tasks in a meaningful way

  Adding name with a human meaningful description better communicates the intent to someone executing a play. Consider this example play and its standard Ansible output.

**Bad Play**

        tasks:
        - yum:
          name: httpd
          state: latest

        - service:
          name: httpd
          state: started
          enabled: yes

**Bad Output**

    PLAY [web] ******************************************************

    TASK [setup] ****************************************************
    ok: [web1]

    TASK [yum] ******************************************************
    ok: [web1]

    TASK [service] **************************************************
    ok: [web1]

...so what actually happened in this play?  With no meaningful names on the tasks, its difficult to tell.

Ansible provides a ``name`` declaration for all tasks and plays, let's see how it looks with a more meaningful set of task names in our play

**Good Play**

    name: installs and starts apache
    tasks:
    - name: install apache packages
      yum:
        name: httpd
        state: latest

    - name: starts apache service
      service:
        name: httpd
        state: started
        enabled: yes

**Good Output**

    PLAY [install and starts apache] ***********************************

    TASK [setup] *******************************************************
    ok: [web1]

    TASK [install apache packages] *************************************
    ok: [web1]

    TASK [starts apache service] ***************************************
    ok: [web1]

That...is...satisfying...  So it's clear now what this play was doing yes?  Get it - Always Name your plays and tasks...got it?  good...moving on.

### Use meaningful variable names

Ansible has a powerful variable processing system that collects metadata from various sources and merges them as a play runs on your hosts. A lot of effort goes into making that power as easy and transparent as possible to users.

A limitation to this approach is that you run the risk of variable name collisions if two different pieces of information are stored in the same variable name.

As a best practice, prefix variables with the source or target of the data it represents. Prefixing variables is particularly vital with developing reusable and portable roles.

You can also dramatically improve the readability of your plays by using human-meaningful variable names that communicate their purpose to others or (more importantly) yourself at a later date.

**Example:**

    apache_max_keepalive: 25
    apache_port: 80
    tomcat_port: 8080

*Note: Variable names should be all lower-case, and use underscores to_separate_terms, Please DO NOT use CamelCase, or dash-separated-values or L33k|Sp38k or mixed case...seriously*

### Use Native YAML Syntax

At its core, the Ansible playbook runner is a YAML parser with added logic such as commandline key=value pairs shorthand. While convenient when cranking out a quick playbook or a docs example, that style of formatting reduces readability. Refrain from using that shorthand as a best practice.

Here is an example of some tasks using the key=value shorthand:


    - name: install telegraf
      yum:
        name: telegraf-{{ telegraf_version }} state=present update_cache=yes disable_gpg_check=yes enablerepo=telegraf
      notify: restart telegraf

    - name: configure telegraf
      template: src=telegraf.conf.j2 dest=/etc/telegraf/telegraf.conf
      notify: restart telegraf

    - name: start telegraf
      service: name=telegraf state=started enabled=yes

Now here is the same tasks using native YAML syntax:

    - name: install telegraf
      yum: telegraf-{{ telegraf_version }}
        state: present
        update_cache: yes
        disable_gpg_check: yes
        enablerepo: telegraf
      notify: restart telegraf

    - name: configure telegraf
      template:
        src: telegraf.conf.j2
        dest: /etc/telegraf/telegraf.conf
      notify: restart telegraf

    - name: start telegraf
      service:
        name: telegraf
        state: started
        enabled: yes
Native YAML has more lines; but, those lines are shorter. It lets the eyes scan straight down the play. The task parameters are stacked and easily distinguished from the next.

Native YAML syntax also has the benefit of improved syntax highlighting in virtually any modern text editor out there. Being native YAML, editors such as vim and Atom will highlight YAML keys (module names, directives, parameter names) from their values further aiding the readability of your content.

### Use Modules Avoid Run Commands

Run commands are what we collectively call the ``command``, ``shell``, ``raw`` and ``script`` modules.  These modules enable users to do commandline operations in different ways. They’re a great catch all mechanism for getting things done, but they should be used sparingly and as a last resort. The reasons are many and varied.

The overuse of run commands is often a symptom of TL;DR regarding Ansible docs and it's common amongst those just becoming familiar with Ansible for automating their work. They use ``shell`` to fire off a bash command they already know without stopping to look at the Ansible docs. That works well enough initially, but it undermines the value of automating with Ansible and sets things up for problems down the road.

The most important thing to consider is that these run commands have little logic to them and no concept of desired state like a typical Ansible module.

That shell that succeeded the first time you ran your play may fail the next time when something already exists. That’s unless you ignore_errors on that task. But how do you catch a real error like wrong permissions?

Ansible at its core is a configuration management tool focused on a principle called **idempotency**.  

    idempotent - adjective

          denoting an element of a set that is unchanged in value when
          multiplied or otherwise operated on by itself.

In simpler terms, everything in an Ansible playbook or role is meant to be repeatable.  The whole point is to be able to run the same set of steps OVER and OVER and have things change *only* where needed.  This is the power of Ansible.

All of Ansible's native modules (with a few exceptions) can be run repeatedly and will only change the environment, if a change is needed.  So for instance, if I were to do this in a role or play:

    tasks:
    - name: "install the awesome package"
      yum:
        name: "awesome"
        state: "running"

Ansible does the following:

1. Check if the awesome package is installed and running, if yes and yes, then exit, I'm done no changes needed.
2. If awesome is installed but not running, then start it and note that I changed something, then exit.
3. If awesome is not installed, then I'll use the yum package manager to download and install as well as start the awesome processes and note that I changed something, then exit

This flow is critical when you're thinking about how to write a good playbook or role.  Note what Ansible is doing.  Ansible is asserting a desired state, and if proven wrong, doing only what is necessary to achieve the desired state.  This saves time AND creates a condition where just by executing a role or playbook one can be assured that a piece of infrastructure is configured as expected

The initial tendency when using Ansible is to repeatedly use the **command** and **shell** modules to execute scripts in various languages you have in your pocket to save time...this is not ideal.  

As much as it is nice to reuse what you already have, you will add unnecessary time and unnecessary complexity to a role by continually running non-Ansible scripts

**If you MUST run outside scripts, stick to these guidelines:**

1. KISS - Keep It Simple ~~Stupid~~ Sweetheart - the simpler, the better - we know sometimes complexity is necessary, but make sure its necessary

2. FAIL LOUDLY - A script needs to raise errors if something goes wrong, the worst thing we can do is have a silent failure.  Ansible will error on non-zero exits.

3. DOCUMENTATION IS NOT AN OPTION - Ansible is meant to be explicit and self-documenting, your powershell or perl nightmare is meant to make men weep...please, spare the next person.  Before including an outside script, do code reviews, hand it to a coworker in sales...if they don't know what you're doing, you should really reconsider using the script.

4. ONLY RUN IF NEEDED - Think about assertions you can use to determine if rerunning an outside script is necessary

    i.e. let's say you're copying a bunch of data from a remote location and linking it to various places on the server's filesystems; perhaps before you recopy a high volume of data around, you can run a quick check to see if a directory exists (Using Ansible's stat module), and even better checksum one or more static files in the set to insure everything is as it should be thereby avoiding unnecessary time when configuring new infrastructure.
