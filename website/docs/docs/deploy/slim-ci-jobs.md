---
title: "Slim CI jobs in dbt Cloud"
sidebar_label: "Slim CI jobs"
description: "Learn how to create and set up Slim CI checks to test code changes before deploying to production."
---

You can set up Slim [continuous integration](/docs/deploy/continuous-integration) (CI) jobs to run when someone opens a new pull request in your dbt repository. By running and testing only _modified_ models &mdash; which is what _slim_ refers to &mdash; dbt Cloud ensures these jobs are as efficient and resource conscientious as possible on your data platform.

## Prerequisites

- You have a dbt Cloud account that is on the [Team or Enterprise plan](https://www.getdbt.com/pricing/).
- You must be connected using dbt Cloud’s native integration with [GitHub account](/docs/cloud/git/connect-github), [GitLab account](/docs/cloud/git/connect-gitlab), or [Azure DevOps account](/docs/cloud/git/connect-azure-devops).
    - If you’re using GitLab, you must use a paid or self-hosted account which includes support for GitLab webhooks.
    - If you previously configured your dbt project by providing a generic git URL that clones using SSH, you must reconfigure the project to connect through dbt Cloud's native integration.

## Set up Slim CI jobs

dbt Labs recommends that you create your Slim CI job in a dedicated dbt Cloud [deployment environment](/docs/collaborate/environments/dbt-cloud-environments#create-a-deployment-environment) that's connected to a staging database. Having a separate environment dedicated for CI will provide better isolation between your temporary CI schemas builds and your production data builds. Additionally, sometimes teams need their Slim CI jobs to be triggered when a PR is made to a branch other than main. If your team maintains a staging branch in your release process, having a separate environment will allow you to set a [custom branch](/faqs/environments/custom-branch-settings), and accordingly the CI job in that dedicated environment will be triggered only when PRs are made to the specified, custom branch.

1. On your deployment environment page, click **Create One** to create a new CI job.
2. In the **Execution Settings** section: 
    - For the option **Defer to a previous run state**, choose whichever production job that's set to run often. If you don't see any jobs to select from the dropdown, you first need to run a production job successfully. Deferral tells dbt Cloud to compare the manifest of the current CI job against the project representation that was materialized the last time the deferred job was run successfully. By setting this option, dbt Cloud only checks the modified code and compares the changes against what’s running in production, instead of building the full table or the entire DAG.

    <Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/ci-deferral.png" width="70%" title="Example of the dropdown for Defer to a previous run state"/>

    - For the option **Commands**, enter `dbt build --select state:modified+` in the field. This informs dbt Cloud to build only new or changed models and their downstream dependents. Importantly, state comparison can only happen when there is a deferred job selected to compare state to.


3. In the **Triggers** section, choose the **Continuous Integration** (CI) tab. Then, enable the **Run on Pull Requests** option. This configures pull requests and new commits to be a trigger for the Slim CI job.


## Example pull requests

The green checkmark means the dbt build and tests were successful. Clicking on the dbt Cloud section navigates you to the relevant CI run in dbt Cloud.

### GitHub pull request example

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/example-github-pr.png" width="70%" title="GitHub pull request example"/>

### GitLab pull request example

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/GitLab-Pipeline.png" width="70%" title="GitLab pull request"/>

### Azure DevOps pull request example

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Enabling-CI/ADO CI Check.png" width="70%" title="Azure DevOps pull request"/>


## Troubleshooting

If you're experiencing any issues, review some of the common questions and answers below.

<details>
   <summary>Reconnecting your dbt project to use dbt Cloud's native integration with GitHub, GitLab, or Azure DevOps</summary>
   <div>
      <div>If your dbt project relies the generic git clone method that clones using SSH and deploy keys to connect to your dbt repo, you need to disconnect your repo and reconnect it using the native GitHub, GitLab, or Azure DevOps integration in order to enable dbt Cloud Slim CI.<br></br><br></br>
      First, make sure you have the <a href="https://docs.getdbt.com/docs/cloud/git/connect-github">native GitHub authentication</a>, <a href="https://docs.getdbt.com/docs/cloud/git/connect-gitlab">native GitLab authentication</a>, or <a href="https://docs.getdbt.com/docs/cloud/git/connect-azure-devops">native Azure DevOps authentication</a> set up depending on which git provider you use. After you have gone through those steps, go to <strong>Account Settings</strong>, select <strong>Projects</strong> and click on the project you'd like to reconnect through native GitHub, GitLab, or Azure DevOps auth. Then click on the repository link.<br></br><br></br>
      
      Once you're in the repository page, select <strong>Edit</strong> and then <strong>Disconnect Repository</strong> at the bottom.<br></br>
         <Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Enabling-CI/Disconnect-Repository.png" alt="Disconnect repo"/>
         <br></br>
         Confirm that you'd like to disconnect your repository. You should then see a new Configure a repository link in your old repository's place. Click through to the configuration page:<br></br>
         <Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Enabling-CI/repo-config.png" alt="Configure repo"/>
         <br></br>
         Select the <b>GitHub</b>, <b>GitLab</b>, or <b>AzureDevOps</b> tab and reselect your repository. That should complete the setup of the project and enable you to set up a dbt Cloud CI job.</div>
   </div>
</details>
<details>
   <summary>Error messages that refer to schemas from previous PRs</summary>
   <div>
      <div>If you receive a schema-related error message referencing a <i>previous</i> PR, this is usually an indicator that you are not using a production job for your deferral and are instead using <i>self</i>.  If the prior PR has already been merged, the prior PR's schema may have been dropped by the time the Slim CI job for the current PR is kicked off.<br></br> <br></br>
      
      To fix this issue, select a production job run to defer to instead of self.
      </div>
   </div>
</details>
<details>
   <summary>Production job runs failing at the <b>Clone Git Repository</b> step</summary>
   <div>
      <div>dbt Cloud can only checkout commits that belong to the original repository. dbt Cloud _cannot_ checkout commits that belong to a fork of that repository.<br></br><br></br>
      
      If you receive the following error message at the <b>Clone Git Repository</b> step of your job run:<br></br>
         <code>
         Error message:<br></br>
         Cloning into '/tmp/jobs/123456/target'...<br></br>
         Successfully cloned repository.<br></br>
         Checking out to e845be54e6dc72342d5a8f814c8b3316ee220312...<br></br>
         Failed to checkout to specified revision.<br></br>
         git checkout e845be54e6dc72342d5a8f814c8b3316ee220312<br></br>
         fatal: reference is not a tree: e845be54e6dc72342d5a8f814c8b3316ee220312<br></br>
         </code><br></br><br></br>
         
         Double-check that your PR isn't trying to merge using a commit that belongs to a fork of the repository attached to your dbt project.</div>
   </div>
</details>
<details>
   <summary>CI job not triggering for Virtual Private dbt users</summary>
   <div>
      <div>To trigger jobs on dbt Cloud using the <a href="https://docs.getdbt.com/docs/dbt-cloud-apis/admin-cloud-api">API</a>, your Git provider needs to connect to your dbt Cloud account.<br></br><br></br>
      
      If you're on a Virtual Private dbt Enterprise plan using security features like ingress PrivateLink or IP Allowlisting, registering CI hooks may not be available and can cause the job to fail silently.</div>
   </div>
</details>

### Temp PR schema limitations

If your temporary pull request schemas aren't dropping after a merge or close of the PR, it's likely due to the below scenarios. Open and review the toggles below for recommendations on how to resolve this:

<details>
  <summary>You used dbt Cloud environment variables in your connection settings page </summary>
  <div>
    <div>To resolve this, remove environment variables in your <a href="https://docs.getdbt.com/docs/dbt-cloud/using-dbt-cloud/cloud-environment-variables">connections settings</a>.</div>
  </div>
</details>
<details>
  <summary>You have an empty/blank default schema</summary>
  <div>
    <div>To change this, edit and fill in your default schema.</div>
  </div>
</details>
<details>
  <summary>You have overridden the <code>generate_schema_name</code> macro</summary>
  <div>
    <div>To resolve this, change your macro so that the temporary PR schema name contains the default prefix and review the guidance below:
    <br></br>
      • ✅ Temporary PR schema name contains the prefix <code>dbt_cloud_pr_</code> (like <code>dbt_cloud_pr_123_456_marketing</code>) <br></br>
      • ❌ Temporary PR schema name doesn't contain the prefix <code>dbt_cloud_pr_</code> (like <code>marketing</code>). <br></br>
    </div>
  </div>
</details>
<details>
  <summary>You have overridden the <code>generate_database_name</code> macro</summary>
  <div>
    <div>If you assume that the project's default connection is to a database named <code>analytics</code>, review the guidance below to resolve this:
      <br></br>
       • ✅ Database remains the same as the connection default (like <code>analytics</code>) <br></br>
       • ❌ Database has changed from the default connection (like <code>dev</code>). <br></br>
    </div>
  </div>
</details>