# Adding Team Members

> Depending on the Lab Format, this section may have already been done. If you are not the 'devops' admins for the openshift 101 dev/tools projects you can check to see if you have access to the projects with `oc projects`. If you have access to the openshift 101 project you will see something like this.
>
> ```shell
> * ocp101-june-dev - OpenShift 101 (dev)
> ocp101-june-tools - OpenShift 101 (tools)
> ```

## Team Permissions

Once all projects have been created by the Platform Services team, the team admin
must navigate to each project and assign your users the appropriate permissions.

As a team, find each project and add the rest of the team members. Feel free to experiment with
the default roles.

<kbd>![](./images/01_projects.png)</kbd>

- Once in the project, switch to `Developer view` and then navigate to `Project -> Project Access`

- Select `Add Access` at the bottom

<kbd>![](./images/01_add_access.png)</kbd>

- Add each user based on their GitHub id. Please note that we are using SSO with GitHub login at the moment, don't forget the suffix `@github`!

<kbd>![](./images/01_edit.png)</kbd>

- Select `Save`

This can also be done on the CLI with the `oc` utility:

```
oc policy add-role-to-user [role] samwarren
```

## Roles

- Admin: This is the most privileged of the default roles. This role allows everything that **Edit** allow plus the management of **user and service account access**

- Edit: This is the primary role required for developers/devops to do work in a project. It allows the creation/edit/deletion of Openshift Objects including **Secrets**

- View: This is the basic role that provides users with read access to your project. **Secrets are not viewable** with this privilege.

Next page - [OCP4 Web Console](./01b_web_console_overview.md)
