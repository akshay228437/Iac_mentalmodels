# Ansible Mental Model — Understand Ansible from Zero to DevOps

Think of Ansible as a remote automation system. You tell it what the desired state of your servers should be, and Ansible connects to those servers and makes it happen.

For example, you want to install Git, configure Nginx, create directories, and start services on 10 servers. Instead of logging in to every server manually, you write the instructions once and Ansible executes them.

## 1. The big picture

1. Control Node

Your terminal or automation server where Ansible runs

2. Ansible Project

Playbook

What to do

Inventory

Where to do it

Variables

Which values

Roles

Reusable logic

3. SSH + Privilege Escalation

Connect as a remote user, use sudo when needed

server1

Install packages

server2

Configure services

The key idea: Inventory tells Ansible where, playbooks tell it what, variables tell it which values, and modules perform the actual work.

## 2. The five building blocks you need to understand

1. Inventory — Where?

Defines the servers Ansible manages.

ini
```
[web]
server1 ansible_host=192.168.1.10
server2 ansible_host=192.168.1.11
```

Mental model: your server address book.

2. Playbook — What?

Defines which hosts to target and which tasks to execute.

yaml
```
- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Install Git
      ansible.builtin.package:
        name: git
        state: present
```

Mental model: your automation recipe.

3. Modules — How?

Modules perform operations on managed hosts.

- `package` — install or remove packages.
- `service` — manage services.
- `file` — manage files and directories.
- `template` — generate configuration files.
- `copy` — copy files.
- `command` — execute commands.

Mental model: the tools in your toolbox.

4. Variables and templates — Which values?

Variables customize automation for different servers or environments.

yaml
```
vars:
  app_port: 8080
```

In a template:

jinja2
```
listen {{ app_port }};
```

Mental model: one recipe, different ingredients.

5. Roles — How do I reuse it?

Group related tasks, variables, handlers, and templates into reusable components.

text
```
roles/
  nginx/
    tasks/main.yml
    handlers/main.yml
    templates/nginx.conf.j2
    defaults/main.yml
```

Mental model: a packaged automation component.

## 3. Understand how an Ansible command executes

Suppose you run:

bash
```
ansible-playbook -i inventory.ini broken-playbook.yml
```

Ansible follows this general sequence:

1

Read inventory

Find the target hosts and connection details.

2

Parse the playbook

Load plays, tasks, variables, and handlers.

3

Connect over SSH

Authenticate to each managed host.

4

Execute modules

Install packages, manage files, and configure services.

5

Report results

Show ok, changed, unreachable, and failed counts.

Important: `--syntax-check` checks the playbook structure; it does not verify that SSH works or that tasks will succeed on the target machine.

## 4. How all the files fit together

Here is a realistic project structure for a DevOps engineer.

text
```
ansible-project/
├── ansible.cfg
├── inventory/
│   ├── dev.ini
│   └── prod.ini
├── playbooks/
│   ├── site.yml
│   └── deploy.yml
├── roles/
│   ├── common/
│   │   └── tasks/main.yml
│   ├── nginx/
│   │   ├── tasks/main.yml
│   │   ├── handlers/main.yml
│   │   ├── templates/nginx.conf.j2
│   │   └── defaults/main.yml
│   └── app/
│       ├── tasks/main.yml
│       ├── templates/application.properties.j2
│       └── defaults/main.yml
└── group_vars/
    ├── dev.yml
    └── prod.yml
```

| File or directory | Responsibility |
| --- | --- |
| `ansible.cfg` | Default Ansible settings |
| `inventory/` | Target hosts and groups |
| `playbooks/` | Orchestrate automation |
| `roles/` | Reusable automation components |
| `tasks/main.yml` | Steps to execute |
| `handlers/main.yml` | Actions triggered by notifications |
| `templates/` | Dynamic configuration files |
| `defaults/main.yml` | Default role variables |
| `group_vars/` | Values for groups or environments |

You do not need all these files for every project. Start with an inventory and one playbook, then introduce roles as your automation grows.

## 5. The most important concept: desired state

Ansible is designed around describing the state you want.

For example:

yaml
```
- name: Ensure Git is installed
  ansible.builtin.package:
    name: git
    state: present
```

You are not instructing Ansible to run `apt install git` every time. You are declaring that Git must be installed.

- If Git is absent, Ansible installs it.
- If Git is already installed, Ansible normally reports `ok` without changing anything.

This behavior is called idempotency. It makes Ansible safer to run repeatedly.

## 6. Commands to remember

| Goal | Command |
| --- | --- |
| Test connectivity | `ansible -i inventory.ini web -m ansible.builtin.ping` |
| Check syntax | `ansible-playbook -i inventory.ini --syntax-check playbook.yml` |
| Preview possible changes | `ansible-playbook -i inventory.ini playbook.yml --check --diff` |
| Run playbook | `ansible-playbook -i inventory.ini playbook.yml` |
| Target one host | `ansible-playbook -i inventory.ini playbook.yml --limit server1` |
| List target hosts | `ansible -i inventory.ini web --list-hosts` |

The `--check` mode is a simulation and is not supported equally by every module.

## 7. A learning path to follow

1. Inventory and SSH: understand hosts, groups, usernames, authentication, and `become`.
2. YAML and playbooks: practice indentation, tasks, modules, and execution order.
3. Variables and conditions: learn `vars`, `group_vars`, `when`, and loops.
4. Files and templates: configure applications using `copy`, `file`, and `template`.
5. Handlers and roles: build reusable Nginx, Java, Docker, and application roles.
6. Real-world workflows: learn environment separation, Ansible Vault, check mode, and CI/CD integration.

## Test your mental model

Question: You need to install Nginx on `server1`. Which combination tells Ansible both where to connect and what to do?

Template + handler

b Inventory + playbook

Variables + `ansible.cfg`

Role defaults + inventory only

 Check my answer

The mental model to keep in your head is simple: Inventory = where, playbook = what, module = how, variables = values, role = reuse. Once this clicks, Ansible's directory structure and commands become much easier to understand.
