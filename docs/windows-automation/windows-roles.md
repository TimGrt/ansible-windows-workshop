# Roles

Although it is possible to write a playbook in one file as we’ve done throughout this workshop, eventually you’ll want to reuse files and start to organize things.

Ansible Roles are the way we do this. When you create a role, you deconstruct your playbook into parts and those parts sit in a directory structure. This is considered best practice and will save you a lot of time in the future.

For this exercise, you are going to take the playbook you just wrote and refactor it into a role.

Let’s begin with seeing how your iis-basic-playbook will break down into a role…

## Step 1 - Create directory structure

In Visual Studio Code, navigate to explorer and your *WORKSHOP_PROJECT* section where you previously made `iis_advanced`.

![iis\_advanced](images/6-vscode-existing-folders.png)

Select the **iis_advanced** folder.

Create a directory called **roles** by right-clicking on **iis_advanced** and selecting *New Folder*.

Now right-click **roles** and create a new folder underneath called `iis_simple`.

Within *iis_simple* create new folders as follows:

* defaults
* vars
* handlers
* tasks
* templates

Within each of these new folders (except templates), right-click and create *New File* Create a file called `main.yml` in each of these folders. You will not do this under templates as we will create individual template files. This is your basic role structure and main.yml will be the default file that the role will use for each section.

The finished structure will look like this:

![Role Structure](images/6-create-role.png)

!!! tip
    You can let Ansible create the directory structure for you!  
    Run the following command for a complete role skeleton:

    ```bash
    ansible-galaxy role init roles/iis_simple
    ```

## Step 2 - Refactor playbook

In this section, we will separate out the major parts of your playbook including `vars:`, `tasks:`, `template:`, and `handlers:`.

Make a backup copy of `site.yml`, then create a new `site.yml`.

Navigate to your `iis_advanced` folder, right click `site.yml`, click `rename`, and call it `site.yml.backup`

Create a blank new file called `site.yml` in the same folder

Update `site.yml` to look like to only call your role. It should look like below:

```yaml
--8<-- "iis-role-playbook.yml"
```

![New site.yml](images/6-new-site.png)

## Step 3 - Move variables to role

Add a default variable to your role. Edit the `roles\iis_simple\defaults\main.yml` as follows:

```yaml
--8<-- "iis-role-defaults-variables.yml"
```

Add some role-specific variables to your role in `roles\iis_simple\vars\main.yml`.

```yaml
--8<-- "iis-role-vars-variables.yml"
```

!!! info "Hey, wait… did we just put variables in two separate places?"

    Yes… yes we did. Variables can live in quite a few places. Just to name a few:

    * vars directory
    * defaults directory
    * group\_vars directory
    * In the playbook under the `vars:` section
    * In any file which can be specified on the command line using the
      `--extra_vars` option
    * On a boat, in a moat, with a goat *(disclaimer: this is a complete lie)*

    Bottom line, you need to read up on [variable precedence](https://docs.ansible.com/ansible/latest/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable){:target="_blank"} to understand both where to define variables and which locations take precedence. In this exercise, we are using role defaults to define a couple of variables and these are the most malleable. After that, we defined some variables in `/vars` which have a higher precedence than defaults and can’t be overridden as a default variable.

## Step 4 - Create handler

Create your role handler in `roles\iis_simple\handlers\main.yml`.

```yaml
--8<-- "iis-role-handlers-tasks.yml"
```

## Step 5 - Add tasks

Add tasks to your role in `roles\iis_simple\tasks\main.yml`.

```yaml
--8<-- "iis-role-tasks.yml"
```

??? example "More separation is also possible"

    It is even better to split up the tasks into separate *task files* and *import* them in the `main.yml` of your `tasks` folder. The following is only an example on how to split, it is up to you to find a meaningful separation. Always remember, shorter is better, but logically connected tasks should be kept alongside.

    ```yaml title="tasks/main.yml"
    ---
    # tasks file for iis_simple

    - import_tasks: installation.yml
    - import_tasks: firewall.yml
    - import_tasks: configuration.yml
    ```

    ```yaml title="tasks/installation.yml"
    ---
    # tasks file for iis_simple

    - name: Install IIS
      ansible.windows.win_feature:
        name: Web-Server
        state: present
    ```

    ```yaml title="tasks/firewall.yml"
    ---
    # tasks file for iis_simple

    - name: Open port for site on the firewall
      ansible.windows.win_firewall_rule:
        name: "iisport{{ item.port }}"
        enable: yes
        state: present
        localport: "{{ item.port }}"
        action: Allow
        direction: In
        protocol: Tcp
      loop: "{{ iis_sites }}"
    ```

    ```yaml title="tasks/configuration.yml"
    ---
    # tasks file for iis_simple

    - name: Create site directory structure
      ansible.windows.win_file:
        path: "{{ item.path }}"
        state: directory
      loop: "{{ iis_sites }}"

    - name: Create IIS site
      ansible.windows.win_iis_website:
        name: "{{ item.name }}"
        state: started
        port: "{{ item.port }}"
        physical_path: "{{ item.path }}"
      loop: "{{ iis_sites }}"
      notify: restart_iis_service_handler

    - name: Template simple web site to iis_site_path as index.html
      ansible.windows.win_template:
        src: 'index.html.j2'
        dest: '{{ item.path }}\index.html'
      loop: "{{ iis_sites }}"

    - name: Show website addresses
      ansible.builtin.debug:
        msg: "{{ item }}"
      loop:
        - http://{{ ansible_host }}:8080
        - http://{{ ansible_host }}:8081
    ```

## Step 6 - Add template

Add your `index.html` template.

Right-click `roles\iis_simple\templates` and create a new file called `index.html.j2` with the following content:

```html
<html>
<body>

  <p align=center><img src='http://docs.ansible.com/images/logo.png' align=center>
  <h1 align=center>{{ ansible_hostname }} --- {{ iis_test_message }}</h1>

</body>
</html>
```

Now, remember we still have a *templates* folder at the base level of this playbook, so we will delete that now. Right click it and Select *Delete*.

## Step 6 - Commit

Click `File` :material-arrow-right: `Save All` to ensure all your files are saved.

Click the Source Code icon as shown below (1).

Type in a commit message like `Adding iis_simple role` (2) and click the
check box above (3).

![Commit iis\_simple\_role](images/6-commit.png)

Click the `synchronize changes` button on the blue bar at the bottom left. This should again return with no problems.

## Step 8 - Run new playbook

Now that you’ve successfully separated your original playbook into a role, let’s run it and see how it works. We don’t need to create a new template, as we are reusing the one from Exercise 5. When we run the template again, it will automatically refresh from git and launch our new role.

Before we can modify our Job Template, you must first go resync your Project again. So do that now.

Select TEMPLATES

!!! tip
    Alternatively, if you haven’t navigated away from the job templates creation page, you can scroll down to see all existing job templates.

Click the rocketship icon ![Add](images/at_launch_icon.png) for the **IIS Advanced** Job Template.

When prompted, enter your desired test message.

If successful, your standard output should look similar to the figure below. Note that most of the tasks return OK because we’ve previously configured the servers and services are already running.

![Job output](images/6-job-output.png)

When the job has successfully completed, you should see two URLs to your websites printed at the bottom of the job output. Verify they are still working.

## Review

You should now have a completed playbook, `site.yml` with a single role called `iis_simple`. The advantage of structuring your playbook into roles is that you can now add reusability to your playbooks as well as simplifying changes to variables, tasks, templates, etc.

[Ansible Galaxy](https://galaxy.ansible.com){:target="_blank"} is a good repository of roles for use or reference.
