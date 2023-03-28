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

| Key                        | Value                                                    |
| -------------------------- | -------------------------------------------------------- |
| Source Control URL         | *Input your Git repo e.g. https://...@dev.azure.com/...* |
| Source Control Credentials | *Choose your Git credential*                             |
| Options                    | :material-checkbox-outline: Update Revision on Launch    |

Click SAVE ![Save](images/at_save.png). Your project should sync automatically, otherwise click **Projects** and then click the sync icon next to your project. Once this is complete, you can create the job template.

![Project Sync](4-project-sync.png)

## Create Job Template

Select **Templates**.  

Click the ![Add](add.png) icon, and select Job Template.

Complete the form using the following values:

| Key                   | Value                                           |
| --------------------- | ----------------------------------------------- |
| Name                  | IIS Basic Job Template *username*               |
| Description           | Template for the iis-basic playbook             |
| JOB TYPE              | Run                                             |
| INVENTORY             | Workshop Inventory *username*                   |
| PROJECT               | Ansible Workshop Project *username*             |
| Execution Environment | BSS EE - Windows                                |
| PLAYBOOK              | `iis_basic/install_iis.yml`                     |
| CREDENTIAL            | Name: **Windows Credential**                    |
| LIMIT                 | windows                                         |
| OPTIONS               | :material-checkbox-outline: ENABLE FACT STORAGE |

![Create Job Template](images/4-create-job-template.png)

Click SAVE ![Save](images/at_save.png).

## Add Survey

On the resulting page, select the **Survey** tab and press the **Add** button
![Create Survey](images/4-create-survey.png)

Complete the survey form with following values

| Key                    | Value                                                      | Note             |
|------------------------|------------------------------------------------------------|------------------|
| PROMPT                 | Please enter a test message for your new website           |                  |
| DESCRIPTION            | Website test message prompt                                |                  |
| ANSWER VARIABLE NAME   | `iis_test_message`                                         |                  |
| ANSWER TYPE            | Text                                                       |                  |
| MINIMUM/MAXIMUM LENGTH |                                                            | Use the defaults |
| DEFAULT ANSWER         | *Be creative, keep it clean, we’re all professionals here* |                  |

After configuring your survey, click **Save**. On the resulting page, turn on the survey you just created.

![Survey creating](images/4-survey-created.png)

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

When the job has successfully completed, you should see a URL to your website printed at the bottom of the job output.

If all went well, you should see something like this, but with your own custom message of course.

![New Website with Personalized Test Message](images/4-website-output.png)

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