---
title: An Introduction to Ansible Roles
id: ansible-roles-intro
description: Learn what an Ansible Role is and when you should use one. 
updated: 2022-04-20
categories: [code]
tags: [code]
status: Published
authors: Jason Belk
duration: 5
---

<!-- Overview Step is a required step and must be at the beginning of each codelab -->

{{< step label="Overview" duration="1:00" >}}

### What you’ll Learn

- What is in an Ansible role
- How to create an Ansible role
- When you should use an Ansible role
- How to include an Ansible role in a playbook

### What you'll need

- Access to a [Sandbox with Ansible installed](https://devnetsandbox.cisco.com/RM/Diagram/Index/cae403c2-27af-4c7d-b1e1-99b7d42f1504?diagramType=Topology)
- Text editor (such as VS Code) to remotely edit files on the jump host or your local Ansible installation.
- A basic understanding of Ansible

{{< /step >}}

{{< step label="Ansible Role Basics" duration=1:00" >}}

### What is an Ansible Role?

An [Ansible Role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) lets you load in:

- variables
- tasks
- templates
- files
- custom modules
- handlers

to be used within your playbooks.

They can be as simple as loading in a few variables you always use between playbooks or projects, or as complicated as an entire workflow of tasks that execute based on device types from tons of conditional statements.

### The Ansible Role Structure

Ansible roles have a standard structure to them with directories named a specific way. At the bare minimum you only need to include at least one of the directories, but you can leave out any of the ones you are not using. `main.yml` is a special filename in Ansible and it will look at that YAML file first before any other YAML file in each directory.

```bash
└── sample-role
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```


{{< /step >}}

{{< step label="How to Create an Ansible Role" duration="1:00" >}}

Ansible includes a command-line utility to create your own Ansible roles called `ansible-galaxy`. This is very helpful since Ansible expects the roles to have a certain file structure. If you use the `ansible-galaxy` command, there is no question that the directory structure aligns with what Ansible is expecting to see when it loads in the role (spelling errors are a common problem when working with Ansible expecting things a certain way).

If Ansible is installed, execute the following command to create a `sample-role`:

```bash
ansible-galaxy init sample-role
```

Ansible will then create the following directories and files in the current working directory:

```bash
$ tree .
.
└── sample-role
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml

9 directories, 8 files
ansible-roles$
```

Even though you will probably not use all of the directories listed above, they do not hurt anything to exist and not be used. The directories that see the most use within network automation are:

- `defaults`
    - The variable in this directory are baseline default values for whatever variables the role is going to use in case no input is given to the role when imported
- `tasks`
    - **This** is the real value of what most people use roles for. If you are reusing tasks all the time, abstracting them into a role shortens the length of the playbooks you write and allows you to make changes to those tasks in the role and have the changes trickle down to all the playbooks that use that role.
- `templates`
    - If you have tasks in the role that are using Jinja2 templates, they will be here. 
- `vars`
    - If you have tasks or templates that are using variables, they will be here. You can set default values for them in the defaults directory, otherwise it will not function as designed if there are variables used with no inputs given.


{{< /step >}}

{{< step  label="Using Ansible Roles" duration="3:00" >}}

### When to use Ansible Roles

Ansible roles are a great way of bundling up commonly used tasks, variables and even custom modules. A common practice in software engineering is to take code that is reusable across multiple situations and abstract it, such as in the Python `class` feature. Roles are more limited in scope, but follow a similar design thought process.

For example, if you are always executing the same set of pre / post checks in a network change using Ansible, that could be abstracted into an Ansible role you include before or after your change. You could include a series of tasks that send show commands, save the outputs to a file sent through a formatting clean up of a Jinja2 template and put in directories that are organized per device. Regardless of which change you are doing, whether that is BGP or ACLs, you are going to have a series of commands sent to a device beforehand and afterwards that you will want captured in output files. An Ansible role is a great example of that use case, where you have the exact commands set as variables that can be fed into the role at runtime.

### How to use Ansible Roles

There are a [couple of different ways to use roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#using-roles), though the classic way is to include it at in the playbook play definition level:

```yaml
---

- name: "PLAY 1: SHOW COMMANDS FROM ROLE"
  hosts: routers            
  connection: network_cli   
  roles:
    - sample-role
  tasks:      
    - name: "Print sample-role show command output"
      debug:
        msg: "{{ show_command_output }}"
```

Working with the following inventory file:


```ini

[routers]
sandbox-iosxe-recomm-1.cisco.com ansible_network_os=ios ansible_user=developer ansible_password=C1sco12345
```

Assuming we merely update the `sample-role/tasks/main.yml` with a single task:

```yaml
---
# tasks file for sample-role

  - name: run multiple commands on remote nodes
    ios_command:
        commands:
          - show ip interface brief
          - show ip route
    register: show_command_output
```

Note that the tasks file in the role has no Ansible play definition, hosts or anything but the list of tasks without the `tasks` key. Before the tasks in the playbook begin, it will execute the tasks and load in the variables from the role. 

Note in the output the line `[sample-role : run multiple commands on remote nodes]` which includes the task from the role.

Here is some sample output (truncated for length):

```bash
(ansible-demo) ansible-roles$ ansible-playbook -i inventory working-with-roles.yaml

PLAY [PLAY 1: SHOW COMMANDS FROM ROLE] *****************************************

TASK [sample-role : run multiple commands on remote nodes] *********************
ok: [sandbox-iosxe-recomm-1.cisco.com]

TASK [Print sample-role show command output] ***********************************
ok: [sandbox-iosxe-recomm-1.cisco.com] => {
    "msg": [
        [
            "Interface              IP-Address      OK? Method Status                Protocol",
            "GigabitEthernet1       10.10.20.48     YES other  up                    up      ",
            "GigabitEthernet2       10.255.255.1    YES other  down                  down    ",
            "GigabitEthernet3       192.168.30.1    YES other  down                  down    ",
            "Loopback1              2.2.2.2         YES manual up                    up      ",
            "Loopback2              unassigned      YES other  up                    up      ",
            "Loopback5              5.5.5.5         YES manual up                    up      ",
            "Loopback7              192.168.0.254   YES other  up                    up      ",
            "Loopback21             unassigned      YES unset  up                    up      ",
            "Loopback50             1.1.1.1         YES other  up                    up      ",
 ...
        ],
        [
            "Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP",
            "       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area ",
            "       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2",
            "       E1 - OSPF external type 1, E2 - OSPF external type 2",
            "       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2",
            "       ia - IS-IS inter area, * - candidate default, U - per-user static route",
            "       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP",
            "       a - application route",
            "       + - replicated route, % - next hop override, p - overrides from PfR",
            "",
            "Gateway of last resort is 10.10.20.254 to network 0.0.0.0",
            "",
            "S*    0.0.0.0/0 [1/0] via 10.10.20.254, GigabitEthernet1",
            "      2.0.0.0/8 is variably subnetted, 4 subnets, 2 masks",
            "C        2.2.2.0/24 is directly connected, Loopback1",
            "L        2.2.2.2/32 is directly connected, Loopback1",
            "C        2.22.2.0/24 is directly connected, Loopback222",
            "L        2.22.2.22/32 is directly connected, Loopback222",
            "      3.0.0.0/8 is variably subnetted, 2 subnets, 2 masks",
            "C        3.32.3.0/24 is directly connected, Loopback3333",
            "L        3.32.3.32/32 is directly connected, Loopback3333",
            "      5.0.0.0/32 is subnetted, 2 subnets",
            "C        5.5.5.5 is directly connected, Loopback5",
            "C        5.255.255.5 is directly connected, Loopback2022",
            "      9.0.0.0/32 is subnetted, 1 subnets",
 ....
        ]
    ]
}

PLAY RECAP *********************************************************************
sandbox-iosxe-recomm-1.cisco.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

(ansible-demo) ansible-roles$
```

There are [other ways of using roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#using-roles) so that the tasks you import are executed before or after you tasks, but they are outside the scope of this introduction.


{{< /step >}}


<!-- Call to Action Step is a required step and must be at the end of each codelab -->
{{< step  label="Congratulations" duration="1:00" >}}

Congrats on finishing the lab! 

## Learn More

- [Check out the Cisco Learning Network DevNet Cert Community](https://learningnetwork.cisco.com/s/topic/0TO3i0000008jY5GAI/devnet-certifications-community)
- [Check out other Cisco Training on Network Programmability](https://learningnetworkstore.cisco.com/network-programmability)

{{</step >}}
