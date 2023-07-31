## What is Ansible?
 Ansible® is an open source IT automation tool that automates provisioning, configuration management, application deployment, orchestration, and many other manual IT processes.

## How Ansible works?
 Ansible works by connecting to what you want automated and pushing programs that execute instructions that would have been done manually. These programs utilize Ansible modules that are written based on the specific expectations of the endpoint’s connectivity, interface, and commands. Ansible then executes these modules (over standard SSH by default), and removes them when finished (if applicable).

There are no additional servers, daemons, or databases required. Typically you'll work with your favorite terminal program, a text editor, and a version control system to keep track of changes to your content.

Ansible Glossary: https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html

## Ansible Architecture

Both community Ansible and Ansible Automation Platform have the concept of a control node and a managed node. 
1. `Control Node`: The control node is where Ansible is executed from, for example where a user runs the ansible-playbook command. 
2. `Managed nodes`: are the devices being automated, for example a Microsoft Windows server.

For automating Linux and Windows, Ansible works by connecting to managed nodes and pushing out small programs, called "Ansible modules," to them. These programs are written to be resource models of the desired state of the system. Ansible then executes these modules (over SSH by default), and removes them when finished. These modules are designed to be **idempotent**(`An operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions.`) when possible, so that they only make changes to a system when necessary.

## Basic Concepts

1. [Control node](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id1): The machine from which you run the Ansible CLI tools (`ansible-playbook` , `ansible`, `ansible-vault` and others). You can use any computer that meets the software requirements as a control node - laptops, shared desktops, and servers can all run Ansible. Multiple control nodes are possible, but Ansible itself does not coordinate across them, see `AAP` for such features.
 
2.  [Managed nodes](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id2): Also referred to as ‘hosts’, these are the target devices (servers, network appliances or any computer) you aim to manage with Ansible. Ansible is not normally installed on managed nodes, unless you are using `ansible-pull`, but this is rare and not the recommended setup.

3. [Inventory](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id3): A list of managed nodes provided by one or more ‘inventory sources’. Your inventory can specify information specific to each node, like IP address. It is also used for assigning groups, that both allow for node selection in the Play and bulk variable assignment. To learn more about inventory, see [the Working with Inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#intro-inventory) section. Sometimes an inventory source file is also referred to as a ‘hostfile’.


4. [Playbooks](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id4): They contain Plays (which are the basic unit of Ansible execution). This is both an ‘execution concept’ and how we describe the files on which `ansible-playbook` operates. Playbooks are written in YAML and are easy to read, write, share and understand. To learn more about playbooks, see [Ansible playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html#about-playbooks).

5. [Plays](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id5): The main context for Ansible execution, this playbook object maps managed nodes (hosts) to tasks. The Play contains variables, roles and an ordered lists of tasks and can be run repeatedly. It basically consists of an implicit loop over the mapped hosts and tasks and defines how to iterate over them.

6. [Roles](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id6): A limited distribution of reusable Ansible content (tasks, handlers, variables, plugins, templates and files) for use inside of a Play. To use any Role resource, the Role itself must be imported into the Play.

7. [Tasks](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id7): The definition of an ‘action’ to be applied to the managed host. Tasks must always be contained in a Play, directly or indirectly (Role, or imported/included task list file). You can execute a single task once with an ad hoc command using `ansible` or `ansible-console` (both create a virtual Play).

8. [Handlers](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id8): A special form of a Task, that only executes when notified by a previous task which resulted in a ‘changed’ status.

9. [Modules](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id9): The code or binaries that Ansible copies to and executes on each managed node (when needed) to accomplish the action defined in each Task. Each module has a particular use, from administering users on a specific type of database to managing VLAN interfaces on a specific type of network device. You can invoke a single module with a task, or invoke several different modules in a playbook. Ansible modules are grouped in collections. For an idea of how many collections Ansible includes, see the [Collection Index](https://docs.ansible.com/ansible/latest/collections/index.html#list-of-collections).

10. [Plugins](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id10): Pieces of code that expand Ansible’s core capabilities, they can control how you connect to a managed node (connection plugins), manipulate data (filter plugins) and even control what is displayed in the console (callback plugins). See [Working with plugins](https://docs.ansible.com/ansible/latest/plugins/plugins.html#working-with-plugins) for details.

11. [Collections](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id11): A format in which Ansible content is distributed that can contain playbooks, roles, modules, and plugins. You can install and use collections through [Ansible Galaxy](https://galaxy.ansible.com/). To learn more about collections, see [Using Ansible collections](https://docs.ansible.com/ansible/latest/collections_guide/index.html#collections). Collection resources can be used independently and discretely from each other.

12. [AAP](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#id12): Short for ‘Ansible Automation Platform’. This is a product that includes enterprise level features and integrates many tools of the Ansible ecosystem: ansible-core, awx, galaxyNG, and so on.