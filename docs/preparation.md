# Preparation

## Configure Dev Environment

We will be using [Visual Studio Code (VScode)](https://code.visualstudio.com/){:target="_blank"} as our IDE for Ansible development, we will connect VScode to a (Linux) development host.

VScode has a *Remote Explorer* extension on the far left, click on it, we will add a new *SSH* connection. Click on *Open SSH config file*, the :octicons-gear-16: icon. Use the personal `.ssh\config` file, the first entry in the list.

<figure markdown>
  ![VScode Remote SSH configuration](VScodeRemoteSSHconfig.png)
  <figcaption>VScode Remote SSH configuration</figcaption>
</figure>

Input the following configuration and save the file (use ++ctrl+s++ or `File` and `Save`) afterwards.

```ini
Host Ansible-Dev-Node
    HostName hostname
    User username
```

!!! note
    You trainer will provide you with the exact values for `HostName` and `User`!

After saving, if the new SSH target does not show up, click the :fontawesome-solid-arrow-rotate-right: *Refresh* button when hovering above the *Remote* tab.

Do a *right-click* on the SSH target *Ansible-Dev-Node* and choose *Connect in current window...*. You will be asked two questions, what kind of platform the target node is (choose `Linux`), afterwards input your password.

<figure markdown>
  ![VScode Remote SSH connection](VScodeRemoteSSHconnection.png)
  <figcaption>VScode Remote SSH connection</figcaption>
</figure>

Give it some time for installing the VScode plugin (in `~/.vscode-server`), a successful connection is established once a green *SSH: Ansible-Dev-Node* box is shown in the VScode footer.

### Useful VScode Extension

!!! tip
    This is an optional step, it may help with the first steps and is useful for further development with Ansible.

Visual Studio Code has a huge *Marketplace* with loads of useful Plugins or feature extension, one of which is especially useful for Ansible Development and is directly from RedHat and itself is developed with all code open source.

<figure markdown>
  ![VScode Marketplace Ansible Extension](VScodeMarketPlaceAnsibleExtension.png)
  <figcaption>VScode Marketplace Ansible Extension</figcaption>
</figure>

Go to *Extensions* and search for *Ansible*. Use the extension from *Red Hat*, this one is activly maintained. In the extension description, click the *Install* button.

### Configure Git

We will be working with Git, we will need to configure it slightly. Run the following commands:

```bash
git config --global user.name "FirstName LastName"
```

```bash
git config --global user.email firstNameLastName@beiersdorf.com
```

!!! warning
    These are only examples, use your real Name and E-Mail address!  
    This is used to identify, who made the code changes, which is fundamental for collaborative work on your Ansible playbooks.

### Install Ansible & dependencies

You need to install Ansible (and a couple of Python dependencies) yourself. Copy (the *code block* has a copy button on the right) and paste (*right-click* in the terminal window) the command  in your terminal and hit ++enter++.

```bash
pip3 install --user ansible-core ansible-lint pywinrm pywinrm[kerberos]
```

What is installed here?

* `ansible-core` - the Ansible binaries and a subset of available modules
* `ansible-lint` - a Best-Practice checker for Ansible (not used or discussed in detail today, is a dependency for the Ansible VScode extension)
* `pywinrm` and `pywinrm[kerberos]` - Python libraries for Windows Remote Management

The Ansible binary (and other dependencies) were installed in your home directory and is (not yet) usable. We need to adjust the *PATH*. The following command adds a line to the end of your (personal) `.bashrc` file. Copy and paste the command  in your terminal and hit ++enter++.

```bash
echo -e "\nexport PATH=~/.local/bin:\$PATH" >> ~/.bashrc
```

Afterwards, *source* the file for the changes to take effect:

```bash
source ~/.bashrc
```

## Configure Azure DevOps

!!! warning
    ToDo

## Login to AAP

There are a number of constructs in the Automation Controller UI that enable multi-tenancy, notifications, scheduling, etc. However, we are only going to focus on a few of the key constructs that are required for this workshop today.

* Credentials
* Projects
* Inventory
* Job Template

Your Automation Controller instance url and credentials were supplied to you by the trainer.

Use the SAML login option, you may need to open the *icognito* browser if you need to access with a different user for single-sign on.

### Step 1 - Create Machine Credential

Credentials are utilized by Controller for authentication when launching jobs against machines, synchronizing with inventory sources, and importing project content from a version control system.

There are many [types of credentials](https://docs.ansible.com/automation-controller/latest/html/userguide/credentials.html#credential-types){:target="_blank"} including machine, network, and various cloud providers. For this workshop, we are using **machine** and **source control** credentials.

Select CREDENTIALS from the left hand panel under resources

![Cred](1-controller-credentials.png)

Click the ![Add](add.png) icon and add new credential

Complete the form using the following entries:

| Key          | Value              |                                        |
| ------------ | ------------------ | -------------------------------------- |
| Name         | Windows Credential |                                        |
| Organization | Default            |                                        |
| Type         | Machine            |                                        |
| Username     | student#           | **Replace # with your student number** |
| Password     | *password*         | Replace with your student password     |

![Add Machine Credential](1-controller-add-machine-credential.png)

Select SAVE ![Save](at_save.png)

### Step 2 - Create SCM Credential

Our first credential was to access our Windows machines over WinRM. We need another to access our source code repository where our automation projects will live. Repeat the process as above, but with the following details:

| Key             | Value                           |                                    |
| --------------- | ------------------------------- | ---------------------------------- |
| Name            | Git Credential                  |                                    |
| Description     | SCM credential for project sync |                                    |
| Organization    | Default                         |                                    |
| Credential Type | Source Control                  |                                    |
| Username        | student#                        | Replace # with your student number |
| Password        | *password*                      | Replace with your student password |

Select SAVE ![Save](at_save.png)

![Add SCM Credential](1-controller-add-scm-credential.png)

### Step 3 - Create a Project

A **Project** is a logical collection of Ansible content, represented in Controller. You can manage projects by placing your ansible content into a source code management (SCM) system
supported by Controller, including Git and Subversion.

In this environment, playbooks are stored in a git repository available on the workshop Gitea instance. Before a **Project** can be created in Automation Controller, the git URL for the repository is needed. In order to obtain the URL of your project, login to the Gitea instance, select your workshop project and copy the `https` url presented after clicking the "Copy" button.

![Proj](1-gitea-project.png)
![Clone](1-gitea-clone.png)

Click **Projects** on the left hand panel.

![Proj](1-controller-project.png)

Click the ![Add](add.png) icon and add new project

Complete the form using the following entries (**using your student number in SCM URL**)

| Key                           | Value                                                                     |                          |
| ----------------------------- | ------------------------------------------------------------------------- | ------------------------ |
| Name                          | Ansible Workshop Project                                                  |                          |
| Description                   | Windows Workshop Project                                                  |                          |
| Organization                  | Default                                                                   |                          |
| Default Execution Environment | windows workshop execution environment                                    |                          |
| SCM Type                      | Git                                                                       |                          |
| SCM URL                       | https://git.**WORKSHOP**.demoredhat.com/**student#**/workshop_project.git | URL obtained from Step 1 |
| SCM BRANCH                    |                                                                           | Intentionally blank      |
| SCM CREDENTIAL                | Git Credential                                                            |                          |

OPTIONS

* :material-checkbox-blank-outline: Clean
* :material-checkbox-blank-outline: Delete
* :material-checkbox-blank-outline: Track submodules
* :material-checkbox-outline: Update Revision on Launch
* :material-checkbox-blank-outline: Allow Branch Override

![Defining a Project](1-controller-create-project.png)

Select SAVE ![Save](at_save.png)

Scroll down and validate that the project has been successfully synchronized against the source control repository upon saving. You should see a green icon displaying "Successful" next to the project name in the list view. If the status does not show as "Successful", try pressing the "Sync Project" button again re-check the status.

![Succesfull Sync](1-controller-project-success.png)

### Step 4 - Inventories

An inventory is a collection of hosts against which jobs may be launched. Inventories are divided into groups and these groups contain hosts. Inventories may be sourced manually, by entering host names into Controller, or from one of Automation Controller’s supported cloud providers or inventory plugins from Certified Content Collections on Automation Hub.

A static Inventory has already been created for you today. Let's take a look at this inventory and highlight some properties and configuration parameters.

Click **Inventories** from the left hand panel. You will see the preconfigured Inventory listed. Click the Inventories' name **Workshop Inventory** or the Edit button. ![Edit](at_edit.png)

You are now viewing the Inventory. From here, you can add Hosts, Groups, or even Variables specific to this Inventory.

![Edit Inventory](1-controller-edit-inventory.png)

We will be viewing the hosts, so click the **HOSTS** button.

In the Hosts view, we can see every host associated with this inventory. You will also see which groups a host is associated with. Hosts can be associated with multiple groups. These groups can later be used to narrow down the exact hosts we will later run our automation on.

![Hosts View](1-controller-hosts-view.png)

If you click the **GROUPS** button and then select the **Windows** group, you can inspect variables set at the group level that will apply to all hosts in that group.

![Group Edit](1-controller-group-edit.png)

Today, we have already defined a handful of variables to tell Controller how to connect to hosts in this group. You do not have to define these variables as a Group variable here, they could also be Host variables or reside directly in your Template or Playbook. However, because these variables will be the same for **ALL** windows hosts in our environment, we defined them for the entire windows group.

By default, Ansible will attempt to use SSH to connect to any Host, so for Windows we need to tell it utilize a different connection method, in
this case, [WinRM](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html){:target="_blank"}.

**`ansible_connection: winrm`**

We also instruct Ansible to connect to the WinRM SSL port 5986 (the non-SSL port runs on 5985 but is unencrypted).

**`ansible_port: 5986`**

We also tell Ansible to ignore the WinRM cert, since our lab doesn’t have a proper certificate store setup.

**`ansible_winrm_server_cert_validation: ignore`**

Windows also has various authentication methods that we can utilize to connect. Here we tell Ansible to use the **CredSSP** Transport Method to authenticate to our Windows host:

**`ansible_winrm_transport: credssp`**

If you click the **HOSTS** button, you can view the hosts belonging to the windows group. If you click the link for the host on this page, you can view the host specific variables that have been defined.

![Host Edit](1-controller-host-edit.png)

**`ansible_host`**

This is the IP address of this particular server.

**`ansible_password`**

This is the password needed to connect to this server.

**`ansible_user`**

This is the username that Ansible will use along with the password to connect to this server.

These variables are very host specific thus have been defined at the host level instead of at the group level.

You can find more information about these and other settings in the [Ansible Windows Guides](https://docs.ansible.com/ansible/latest/user_guide/windows.html){:target="_blank"}.  
The authentication settings are particularly important and you will need to review them and decide which method is best for your needs.
