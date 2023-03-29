# Preparation

## Configure Dev Environment

We will be using [Visual Studio Code (VScode)](https://code.visualstudio.com/){:target="_blank"} as our IDE for Ansible development, we will connect VScode to a (Linux) development host.

??? warning "**I never worked with VSCode** ! What do I need to know?"

    VSCode ist a really powerfull and customizable IDE. Nearly everything you need to work with Textfiles is there or can be added as a Plugin.

    These Plugins are the main point for many people to choose VSCode.
    (wait a moment, isn't that extendability also a reason people use Ansible?)

    The unchanged look and feel of VSCode after the first start

    ![Visual Studio Code unchanged look](vscode-first-look.png)

    In the `Editor` Area there is a `Walktroughs` Section. It is highly advised to take a look at the Guides there even as a long time User you may be surprised by some features or shortcuts.

    In the `Activity Bar` there are alot of important things present that need on a regular basis. From top to bottom:
    ```markdown
      + File Explorer
        * Expands to a file explorer where you can see the folder structure. 
          Maybe you need to select a folder first
      + Search/Replace
        * Search and replace in your working directory accross all files
      + Git / Source Control
        * Use versioncontrol to commit your change, switch branches 
          and push/pull to a remote destination
      + Run / Debug
        * not really needed for ansible
      + Extensions
        * All the Extension in a single place ready to search/download/install/uninstall 
      + User Account
        * Your User Account 
      + VSCode Settings
        * All Settings, preferences, themes commands and a lot more to explore here
    ```

    This is small overview. We will add more functionality in the preparation phase.

Add the *Remote Explorer* extension as your first Plugin.
<figure markdown>
  ![Install VScode Remote SSH Extension](vscode-install-remote-ssh.png)
  <figcaption>Install VScode Remote SSH Extension</figcaption>
</figure>

??? info "**I never worked with VSCode** ! What changed?"
    Please wait a moment for it to finish. Afterwards there will be a new Icon in your `Activity Bar` it appears right under the `Extensions` Icon. The Status Bar also changed a bit in the left corner. There is a file opening in the Editor Pane with detailed Informations and Settings for the plugin you just installed. Sometimes it even shows a small demo.

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

Do a *right-click* on the SSH target *Ansible-Dev-Node* and choose *Connect in current window...*. You will be asked two questions, what kind of platform the target node is (choose `Linux`), afterwards input your password. On the first connection you need to enter your password multiple times. 

<figure markdown>
  ![VScode Remote SSH connection](VScodeRemoteSSHconnection.png)
  <figcaption>VScode Remote SSH connection 1</figcaption>
</figure>

<figure markdown>
  ![VScode Remote SSH connection](vscode-connect-remote-ssh.png)
  <figcaption>VScode Remote SSH connection 2</figcaption>
</figure>

Give it some time for installing the VScode plugin (in `~/.vscode-server`), a successful connection is established once a green *SSH: Ansible-Dev-Node* box is shown in the VScode footer. If you open a Terminal now, you will be on the remote host and have a remote shell and linux functionality.

<figure markdown>
  ![VScode Remote SSH connection](vscode-connected-remote-ssh.png)
  <figcaption>VScode Remote SSH connected</figcaption>
</figure>


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

### Useful VScode Extension

!!! tip
    This is an optional step, it may help with the first steps and is useful for further development with Ansible.

Visual Studio Code has a huge *Marketplace* with loads of useful Plugins or feature extension, one of which is especially useful for Ansible Development and is directly from RedHat and itself is developed with all code open source.

<figure markdown>
  ![VScode Marketplace Ansible Extension](VScodeMarketPlaceAnsibleExtension.png)
  <figcaption>VScode Marketplace Ansible Extension</figcaption>
</figure>

Go to *Extensions* and search for *redhat.ansible*. Use the extension from *Red Hat*, this one is activly maintained. In the extension description, click the *Install* button.

??? info "What do I need this extension for and what changed?"
    This extension adds some cool features to VSCode that are purely ansible specific and need `ansible` and `ansible-lint` on your remote host to work.

    If VSCode identifies your file as an "Ansible" file it will change it behaviour. You can check this on the right side of your `Status Bar` at the bottom. `.yml` and `.yaml` files should be identified as YAML Language. Which is correct but we want to change that behaviour to be even "more" correct.
    
    ![Language Settings on Default](vscode-status-bar-language-settings.png)

    Click on `YAML`. VSCode opens a dialog at the top. Select `Configure File Association for '.yml'`. Then select Ansible.

    ![Language Settings on Default](vscode-language-settings-change.png)
    ![Language Settings on Default](vscode-language-settings-change-2.png)

    The language in the `Status Bar` should look like this:

    ![Language Settings on Default](vscode-language-settings-ansible.png)


    !!! info "For more information read the official instructions, please" 
    
        [Ansible Extension Information](https://marketplace.visualstudio.com/items?itemName=redhat.ansible){:target="_blank"} for this extension if you want to leverage it's features


## Configure Azure DevOps

In Azure DevOps code is stored in *repositories*. These repositories on the other hand are kept in a *project*. Every project can have multiple repositories.

Login to Azure DevOps and choose the *Project* in which you want to create your *Repository*. Every project, by default, already has one repository, we will create an additional one for our personal workshop content.

In the top, open the drop-down menu and choose *New repository*.

<figure markdown>
  ![Azure DevOps - Create new repository](AzureDevOpsNewRepo.png){ loading=lazy }
  <figcaption>Azure DevOps - Create new repository</figcaption>
</figure>

On the right of your browser window, enter a name `Ansible-Workshop-username`for your repository, replace *username* with your username abbreviation. Leave all other configuration as-is.

<figure markdown>
  ![Azure DevOps - Create new repository](AzureDevOpsNewRepo2.png){ loading=lazy width="500" }
  <figcaption>Azure DevOps - New repository configuration</figcaption>
</figure>

Once your repository (the place where your *code* (read: Ansible content) will be stored) is created, click the `Clone` button on the right.  
Now, click the button `Generate Git credentials`:

<figure markdown>
  ![Azure DevOps - Generate Git credentials](AzureDevOpsGitCredentials.png){ loading=lazy width="500" }
  <figcaption>Azure DevOps - Generate Git credentials</figcaption>
</figure>

!!! tip
    Store the password (access token) now, you won't be able to retrieve it again!
    Also, make note of your username, we need it a bit later on.

All right, almost done!  
Now, let's clone the repository in your home directory on the Ansible Development node.

Click on the `Clone` button again and copy (by clicking *Copy clone URL to clipboard*) the *HTTPS* URL.  
In the terminal of the dev node, paste the content with a right-click after typing `git clone `:

``` { .bash .no-copy }
git clone https://******@dev.azure.com/******/******/_git/Ansible-Workshop-******
```

Change into the directory (with `cd`) and you are done for now!

## Login to AAP

There are a number of constructs in the Automation Controller UI that enable multi-tenancy, notifications, scheduling, etc. However, we are only going to focus on a few of the key constructs that are required for this workshop today.

* Credentials
* Projects
* Inventory
* Job Template

Your Automation Controller instance url and credentials were supplied to you by the trainer.

Use the SAML login option, you may need to open the *icognito* browser, if you need to access with a different user for single-sign on.

### Create SCM Credential

Credentials are utilized by Controller for authentication when launching jobs against machines, synchronizing with inventory sources, and importing project content from a version control system.

There are many [types of credentials](https://docs.ansible.com/automation-controller/latest/html/userguide/credentials.html#credential-types){:target="_blank"} including machine, network, and various cloud providers. For this workshop, we are using **machine** and **source control** credentials.

Select CREDENTIALS from the left hand panel under resources.

<figure markdown>
  ![AAP Credentials](1-controller-credentials.png){ loading=lazy }
  <figcaption>AAP - Add credentials</figcaption>
</figure>

We need to access our source code repository, where our automation projects will live.  
Click the ![Add](add.png) icon and add a new credential.

Complete the form using the following entries, replace *username* with your username abbreviation:

| Key             | Value                         |                                                              |
| --------------- | ----------------------------- | ------------------------------------------------------------ |
| Name            | Azure DevOps *username*       |                                                              |
| Description     | SCM credential for *username* |                                                              |
| Organization    | *Choose your organization*    |                                                              |
| Credential Type | Source Control                |                                                              |
| Username        | *Your name from Azure DevOps* | Username as shown in the *Clone* button in Azure DevOps      |
| Password        | *Your password*               | The password you retrieved when creating the Git credentials |

Select SAVE ![Save](at_save.png)

![Add SCM Credential](1-controller-add-scm-credential.png)

### Create Machine Credential

We need to access our Windows **Test** machines over WinRM, let's create a *machine* credential for that.

Click the ![Add](add.png) icon and add another new credential.

Complete the form using the following entries, again, replace *username* with your username abbreviation:

| Key          | Value                        | Notes                              |
| ------------ | ---------------------------- | ---------------------------------- |
| Name         | Windows Test Host *username* |                                    |
| Organization | Default                      |                                    |
| Type         | Machine                      |                                    |
| Username     | *Login name*                 | *Ask your trainer for assistance!* |
| Password     | *password*                   |                                    |

![Add Machine Credential](1-controller-add-machine-credential.png)

Select SAVE ![Save](at_save.png)

Perfect, all done for now, you are prepared for all further exercises!
