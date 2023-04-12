# 3 - Sync Project to AAP

A job template in AAP is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many
times. Before you can create a job template with a new playbook, you must first sync your Project so that Controller knows about it.  

## Create Project

We already have created a project and pushed it to our remote Git host (Azure DevOps). Let's create a *project* which is logical collection of Ansible playbooks, stored in a (remote) single source of truth.

Click **Projects** and then the ![Add](add.png) icon.

Complete the form using the following values, replace *username* with your username abbreviation:

| Key                   | Value                               |
| --------------------- | ----------------------------------- |
| Name                  | Ansible Workshop Project *username* |
| Description           | Workshop Project for *username*     |
| Organization          | *Choose your organization*          |
| Execution Environment | BSS EE - Windows                    |
| Source Control Type   | Git                                 |

Now, the form changes, input the following values:

| Key                        | Value                                                 | Note                                                     |
| -------------------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| Source Control URL         | https://...                                           | *Input your HTTPS Git URL* |
| Source Control Credentials | Azure DevOps *username*                               | *Choose your Git credential*                             |
| Options                    | :material-checkbox-outline: Update Revision on Launch |                                                          |

Click SAVE ![Save](images/at_save.png). Your project should sync automatically, otherwise click **Projects** and then click the sync icon next to your project. Once this is complete, you can create the job template.

![Project Sync](4-project-sync.png)

## Create Job Template

Select **Templates**.  

Click the ![Add](add.png) icon, and select Job Template.

Complete the form using the following values, replace *username* with your username abbreviation:

!!! danger
    Your playbook targets the `windows` group. We have **all** test hosts in the inventory `Workshop Example CC`, you **must** place a limit on your job template!
    If you target the complete *windows* group, you will automate **all** hosts, even those of your colleagues!

| Key                   | Value                                           | Notes                                    |
| --------------------- | ----------------------------------------------- | ---------------------------------------- |
| Name                  | IIS Basic Job Template *username*               |                                          |
| Description           | Template for the iis-basic playbook             |                                          |
| Job Type              | Run                                             |                                          |
| Inventory             | Workshop Inventory                              |                                          |
| Project               | Ansible Workshop Project *username*             |                                          |
| Execution Environment | BSS EE - Windows                                |                                          |
| Playbook              | `iis_basic/install_iis.yml`                     |                                          |
| Credentials           | Windows Test Host *username*                        |                                          |
| Limit                 | *Your single host as in your local inventory!*  | Do **not** target the **windows** group! |
| Options               | :material-checkbox-outline: Enable Fact Storage |                                          |

![Create Job Template](images/4-create-job-template.png)

!!! danger
    Did you set the `limit`?

Click SAVE ![Save](images/at_save.png).

## Add Survey

On the resulting page, select the **Survey** tab and press the **Add** button
![Create Survey](images/4-create-survey.png)

Complete the survey form with following values

| Key                    | Value                                                      | Note             |
|------------------------|------------------------------------------------------------|------------------|
| QUESTION               | Please enter a test message for your new website           |                  |
| DESCRIPTION            | Website test message prompt                                |                  |
| ANSWER VARIABLE NAME   | `iis_test_message`                                         |                  |
| ANSWER TYPE            | Text                                                       |                  |
| MINIMUM/MAXIMUM LENGTH |                                                            | Use the defaults |
| DEFAULT ANSWER         | *Be creative, keep it clean, we’re all professionals here* |                  |

After configuring your survey, click **Save**. On the resulting page, turn on the survey you just created.

![Survey creating](images/4-survey-created.png)

!!! tip
    You need to activate your survey with the slider!

??? note "The playbook failed locally, what now?"
    Remember that your playbook did not run successful on the command-line because of a undefined variable?  
    In the survey above we defined a *default* value for the variable, you could do this locally as well.

## Running Job Template

Now that you’ve successfully created your Job Template, you are ready to launch it. Once you do, you will be redirected to a job screen which is refreshing in real time showing you the status of the job.

Select TEMPLATES and Click the rocketship icon ![Add](images/at_launch_icon.png) for the **IIS Basic Job Template**

When prompted, enter your desired test message

![Survey Prompt](images/4-survey-prompt.png)

Select **NEXT** and preview the inputs.

Select LAUNCH ![SurveyL](images/4-survey-launch.png)

Sit back, watch the magic happen.

Once again you should be presented with a Job log page. Selecting the **Details** tab should show you the variable you passed into the playbook among other details.

![Job Summary](images/4-job-summary-details.png)

Next you will be able to see details on the play and each task in the
playbook.

![Play and Task Output](images/4-job-summary-output.png)

When the job has successfully completed, the last task tells us what to do. Connect with RDP to the node, open a browser and input `http://localhost`.

If all went well, you should see something like this, but with your own custom message of course.

![New Website with Personalized Test Message](images/4-website-output.png)

!!! note
    As these are test hosts only, the firewall does not allow external access to the webserver, therefor we need to check this locally.

## Optional

Now that you have IIS Installed, create a new playbook called *remove\_iis.yml* to stop and remove IIS.

!!! hint
    First stop the `W3Svc` service using the `win_service` module, then delete the `Web-Server` service using the`win_feature` module.  
    Optionally, use the `win_file` module to delete the index page.

??? success "Solution needed?"

    This playbook stops and removes the IIS service.

    ```yaml
    ---
    - name: Remove the IIS web service
      hosts: windows
      tasks:
        - name: Stop IIS service
          ansible.windows.win_service:
            name: W3Svc
            state: stopped

        - name: Uninstall IIS
          ansible.windows.win_feature:
            name: Web-Server
            state: absent

        - name: Remove website index.html
          ansible.windows.win_file:
            path: C:\Inetpub\wwwroot\index.html
            state: absent
    ```

## End Result

At this point in the workshop, you’ve experienced the core functionality of Automation Controller. But wait… there’s more! You’ve just begun to explore the possibilities of Automation Controller. The next few lessons will help you move beyond a basic playbook.
