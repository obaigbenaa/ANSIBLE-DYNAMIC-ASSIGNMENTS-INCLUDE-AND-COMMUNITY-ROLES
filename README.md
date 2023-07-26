# Project-13

## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES 

The last 2 projects have already equipped us with some knowledge and skills in Ansible, so we can perform configurations using _ **playbooks** _, _ **roles** _, and _ **imports** _. Now we will continue configuring our UAT servers, learning and practicing new Ansible concepts and modules.

In this project we will introduce_ ** ** _[_ **dynamic assignments** _](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html#includes-dynamic-re-use) by using _ **include** _ module.

Now you may be wondering, what is the difference between  **static**  and  **dynamic**  assignments?

Well, from [_ **Project 12** _](https://professional-pbl.darey.io/en/latest/project12.html), you can already tell that static assignments use _ **import** _ Ansible module. The module that enables dynamic assignments is _ **include** _.

Hence,

import = Static

include = Dynamic

When the  **import**  module is used, all statements are pre-processed at the time playbooks are **[_parsed_](https://en.wikipedia.org/wiki/Parsing)**. Meaning, when you execute the _ **site.yml** __ ** ** _playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when  the **include**  module is used, all statements are processed only during the execution of the playbook. Meaning, after the statements are  **parsed**, any changes to the statements encountered during execution will be used.

**INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE**

In your  **https://github.com/\<your-name\>/ansible-config-mgt****   **GitHub repository start a new branch and call it ** dynamic-assignments**.

Create a new folder, name it  **dynamic-assignments**. Then inside this folder, create a new file and name it _ **env-vars.yml** _

![](RackMultipart20230726-1-rkwj4_html_e1c10f069e4e91a3.png)

**Note:**  Depending on what method you used in the previous project you may have or not have a roles folder in your GitHub repository – if you used ansible-galaxy, then the roles directory was only created on your Jenkins-Ansible server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case – it is up to you.

Our GitHub should have the following structure by now.

├── dynamic-assignments

│ └── env-vars.yml

├── inventory

│ └── dev

└── stage

└── uat

└── prod

└── playbooks

└── site.yml

└── roles (optional folder)

└──defaults

└── handlers

└── meta

└── tasks

└── templates

└── README.md

└── static-assignments

└── common-del.yml

└── common.yml

└── uat-webservers.yml

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as **servername, ip-address** etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment's variables file. Therefore, create a new folder env-vars, then for each environment, create new **YAML** files which we will use to set variables.

**$ cd ansible-config-mgt**

**$ sudo mkdir env-vars**

**$ sudo touch dev.yml stage.yml uat.yml prod.yml**

Our layout should now look like this.

├── dynamic-assignments

│ └── env-vars.yml

├── env-vars

└── dev.yml

└── stage.yml

└── uat.yml

└── prod.yml

├── inventory

└── dev

└── stage

└── uat

└── prod

├── playbooks

└── site.yml

└── static-assignments

└── common.yml

└── webservers.yml

![](RackMultipart20230726-1-rkwj4_html_e826a9ea20c4498a.png)

Now paste the instruction below into the _ **env-vars.yml** _ file.

---

- name: collate variables from env specific file, if it exists

hosts: all

tasks:

- name: looping through list of available files

include\_vars: "{{ item }}"

with\_first\_found:

- files:

- dev.yml

- stage.yml

- prod.yml

- uat.yml

paths:

- "{{ playbook\_dir }}/../env-vars"

tags:

- always

![](RackMultipart20230726-1-rkwj4_html_5c956a314b01a21.png)

Notice 3 things to notice here:

1. We used include\_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version  **2.8**, the include module is deprecated, and variants of include\_\* must be used. These are:

- [include\_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module)
- [include\_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module)
- [include\_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)

In the same version, variants of  **import**  were also introduced, such as:

- [import\_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module)
- [import\_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module)

1. We made use of a [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) { playbook\_dir } and { inventory\_file }. { playbook\_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory\_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.
2. We are including the variables using a loop. with\_first\_found implies that looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment-specific env file does not exist.

**UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS**

We will now update the _ **site.yml** _ file to make use of the dynamic assignment. (_At this point, we cannot test it yet. We are just setting the stage for what is yet to come_)

Update the _ **site.yml** _ to look like below:

---

- hosts: all

- name: Include dynamic variables

tasks:

import\_playbook: ../static-assignments/common.yml

include: ../dynamic-assignments/env-vars.yml

tags:

- always

- hosts: uat-webservers

- name: Webserver assignment

import\_playbook: ../static-assignments/uat-ebservers.yml

![](RackMultipart20230726-1-rkwj4_html_111788cad58644c5.png)

**COMMUNITY ROLES**

Before we proceed, let us commit and push new changes on **dynamic-assignment** branch to git.

![](RackMultipart20230726-1-rkwj4_html_8c57f67205dc04fd.png)

![](RackMultipart20230726-1-rkwj4_html_a62336cc3b96de87.png)

Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open-source engineers out there. These roles are actually production ready, and dynamic to accommodate most of Linux flavors. With Ansible Galaxy again, we can simply download a ready-to-use ansible role, and move on with other businesses.

**Download Mysql Ansible Role**

We can browse available community roles [_ **here** _](https://galaxy.ansible.com/home)

We will be using a [_ **MySQL role developed by ** __ **geerlingguy** _](https://galaxy.ansible.com/geerlingguy/mysql)_ **.** _

**Hint:**  To preserve your GitHub in its actual state after you install a new role – make a commit and push to master your **'ansible-config-mgt'** directory. Of course, you must have  **git**  installed and configured on  **Jenkins-Ansible**  server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory.

We will now create a new branch called **'roles-feature'** and move into it. This is where we will be creating the role for MySQL database.

**$ git branch roles-feature**

**$ git switch roles-feature**

Now we are in the **roles-feature** branch.

Inside _ **roles** _ directory we will now create a new **MySQL** role with  **ansible-galaxy install geerlingguy.mysql**  and rename the folder to _ **mysql** _

**$ ansible-galaxy install geerlingguy.mysql**

**$ mv geerlingguy.mysql/ mysql**

![](RackMultipart20230726-1-rkwj4_html_ef6eac9877e6f8e2.png)

Read and check the README.md file for instructions and edit roles configuration to use the correct credentials for MySQL required for the  **tooling**  website.

**If you scroll to 'mysql\_users: []' tab of the .md file, you will find the default setup of the configuration as below:**

Update it with the correct credentials and privileges to want for username, host list, password, and database access.

![](RackMultipart20230726-1-rkwj4_html_2ec624e4d44e3da4.png)

Save the updated configuration.

Now we will upload all code changes (on branches **roles-feature and dynamic-assignment** ) into your GitHub:

**$ git add .**

**$ git commit -m "Commit new role files into GitHub"**

**$ git push --set-upstream origin roles-feature**

![](RackMultipart20230726-1-rkwj4_html_36918a1405e4f76f.png)

![](RackMultipart20230726-1-rkwj4_html_de76ae9acaa2a3a9.png)

![](RackMultipart20230726-1-rkwj4_html_d7fd8627bc8f34b1.png)

![](RackMultipart20230726-1-rkwj4_html_6345b65f25d25465.png)

![](RackMultipart20230726-1-rkwj4_html_3f2996360ccb94ed.png)

![](RackMultipart20230726-1-rkwj4_html_7f760d5ee2623ad0.png)

![](RackMultipart20230726-1-rkwj4_html_a1254710cd8fa26b.png)

![](RackMultipart20230726-1-rkwj4_html_dbc9de2b693c7b0c.png)

**$ git switch main**

![](RackMultipart20230726-1-rkwj4_html_bd4d36b19e505d59.png)

**LOAD BALANCER ROLES**

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

1. Nginx
2. Apache

With your experience on Ansible so far you can:

- Decide if you want to develop your own roles, or find available ones from the community.

_#I find it faster to develop roles from the community._

**$ ansible-galaxy install geerlingguy.nginx**

**$ ansible-galaxy install geerlingguy.apache**

**$ mv geerlingguy.nginx/ nginx**

**$ mv geerlingguy.apache/ apache**

- Update both static-assignment and site.yml files to refer the roles

![](RackMultipart20230726-1-rkwj4_html_a903a86e87145384.png)

_ **Important Hints:** _

- Since we cannot use both  **Nginx**  and  **Apache**  load balancer, we need to add a condition to enable either one – this is where we can make use of variables.

- Declare a variable in _ **defaults/main.yml** _ file inside the Nginx and Apache roles. Name each variables  **enable\_nginx\_lb**  and  **enable\_apache\_lb**  respectively.
- Set both values to false like this  **enable\_nginx\_lb: false**  and  **enable\_apache\_lb: false**.
- Declare another variable in both roles  **load\_balancer\_is\_required**  and set its value to  **false**  as well.

![](RackMultipart20230726-1-rkwj4_html_3708e211ab59e31e.png)

Note the content of the file that has been commented out.

We can make use of _ **env-vars\uat.yml** _ file to define which loadbalancer to use in UAT environment by setting respective environmental variables to  **true**.

![](RackMultipart20230726-1-rkwj4_html_4f3fa0e3767ed632.png)

Configure nginx loadbalancer _ **defaults/main.yml** _

![](RackMultipart20230726-1-rkwj4_html_c38880840f22378e.png)

Configure apache loadbalancer _ **defaults/main.yml** _

![](RackMultipart20230726-1-rkwj4_html_2abde6f9501e6a54.png)

![](RackMultipart20230726-1-rkwj4_html_8ea724d2ddf9e91a.png)

Enable super user on apache on _ **roles/apache/tasks/setup-RedHat.yml** _

![](RackMultipart20230726-1-rkwj4_html_f0671a583589557d.png)

Enable super user on nginx on _ **roles/nginx/tasks/setup-RedHat.yml** _

![](RackMultipart20230726-1-rkwj4_html_904fca80a29943fe.png)

Run Ansible against the **uat** and load balancer environment.

**$ ansible-playbook -i inventory/uat.yml playbooks/site.yml**

![](RackMultipart20230726-1-rkwj4_html_28fc72e8724d4089.png)

### Congratulations!!! 

We have learned and practiced how to use Ansible configuration management tool to prepare UAT environment for Tooling web solution.

