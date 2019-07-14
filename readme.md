# Ansible Notes

# Basics

Specify hosts file:

```
[defaults]
inventory = ./dev
```

### Concepts

Playbooks - define tasks to be performed on a set of hosts

Modules - Do things like install packages check services etc

Handlers - Will be invoked if there is a change (notify: restart docker)

Templates - .j2 files where you can substitute variables into the text

Register - Save the output of a module to a variable

```yaml
- name: get active sites
  shell: ls -1 /etc/nginx/sites-enabled
  register: active
```

Fail Conditions fail: when:

Roles: A clean structure for managing an specific type of operations You assign Roles (that are a set of tasks etc) to specific operations to be completed

=== Managing Variables ===

Use variables {{ db_host_ip }}. Define default varibles in the default main.yml file. 

Declare variables:

with_dict 

when clause:

```yaml
- name: de-activate sites
  file: path=/etc/nginx/site-enabled/{{ item }} state=absent
  with_items: active.stdout_lines
  when: item not in sites
  notify: restart nginx
```

Can create a file to declare variables in group_vars folder with the file name all(All groups)

#### Modules

Modules are used in tasks to do things on the remote hosts. 

Here are some of the most used. 

- apt - Used to install packages
- command - Run bash commands
- service - Used to start restart enable services and check their state
- copy - Used to copy information and set the permissions src, dest, mode
- file - Check sym links, delete files
- pip - Install from a requirements file and create a virtual env
- lineinfile - Change one or more lines a file
- mysql - Setup users, create databases.
- wait_for - See whether a port is up with timeout. state drained to check that something has been shutdown properly. 

URI - E2E Tests send http requests. 

#### Vault

Create an encrypted list of variables in folder structure group_vars > all > vault

ansible-vault create 

Can specify password in file that can be configurwd in the ansible.cfg file

#### Galaxy

Repository for roles 

#### Common Playbooks

Restart Playbook

Status Playbook: Service status will return 1 or 0 depending on status

#### Tips

Ping all hosts `ansible all -m ping`

Include other playbooks - include: control.yml

gather_facts: false for faster executiong of playbooks

cache_valid_time to reduce cache updates for apt package updates

Use `ansible-playbook site.yml --limit app01` to limit to a specific host 

Use tags to run specific tasks --tags "packages" or --skip-tags "packages"

BE CAREFUL WITH TAGS THEY CAN GET OUT OF HAND!

Specifiy whether a how a task has been changed: changed_when: false 

Pipelining can speed up but requires changing ssh security settings

#### Debugging

Ordering is important. If you mess up a config file your service won't start and crash the execution. You can ignore_errors: true to teporarily ignore the task failing and fix the problem.

Try not to use traditional methods to fix problems.

Step through the tasks: ansible-playbook site-yml --step 
or start at specific task specified from (--list-tasks)

Ansible records which hosts failed which you can use in the limit flag to specify to only run those hosts:
`ansible-playbook site.yml --limit @/home/ansible/site.retry

Syntax check to see if there are any problems: 
ansible-playbook --syntax-check site.yml

--check will show whether it can do a certain set of tasks without doing 

You can use debug: var=active.stdout_lines to have ansible print out the output before it uses it in the next task
var=vars for all variables in scope