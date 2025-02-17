# jira2gitlab

`jira2gitlab` is a python script to import Jira Software projects into a Gitlab instance.

At the time of this writing, Gitlab has a nice [Jira integration plugin](https://docs.gitlab.com/ee/integration/jira/). 
While it works well to _connect_ Gitlab to Jira, it is not (yet) suited to completely migrate projects and issues,
and eventually shut Jira down.

This script is based on and takes further previous efforts, mainly https://gist.github.com/Gwerlas/980141404bccfa0b0c1d49f580c2d494

APIs used:
- Jira [API v2](https://docs.atlassian.com/software/jira/docs/api/REST/8.5.0/) (the latest version supported on Jira Server). A password-based login with administrator rights is needed.
- Gitlab [API v4](https://docs.gitlab.com/ee/api/README.html). An access token with administration rights is needed.

Important: this script is meant to work with Jira Server and is NOT compatible with Jira Cloud.

Tested with:
- Jira Server 8.5.1
- Gitlab Self-Managed 15.11.4-ee

## Features:
- Original title, extended with Jira issue key (Optional)
- Original description, extended with link to Jira issue
- Original comments
- Original labels
- (Optional) Original attachments
- (Optional) Original worklogs, as comment + `/spend` quick-action
- (Optional) Issue references in commits from a linked Bitbucket Server are translated to Gitlab issue references
- (Optional) Jira projects may have custom fields configured. At the moment of writing (2023/02/08) there's a [long-running feature request](https://gitlab.com/gitlab-org/gitlab/-/issues/1906) for this functionality on Gitlab, but it hasn't been implemented. You can configure to perform a simple import of this kind of metadata to gitlab issue in a form of a comment with a table that lists all this info. Only simple string data conversion is done in this case
- Jira comment syntax translated to markdown, including tables
- Jira components are translated to labels
- Jira priority is translated to labels
- Jira status and resolution are translated to labels
- Jira last `fix versions` is translated to milestone
- Jira `relates to` link is translated to `relates_to` link
- Jira `blocks` link is translated to `blocks` link (only Gitlab Premium, otherwise `relates_to`)
- Jira `duplicates` link is translated to `/duplicate` quick-action
- Jira sub-task is translated to an issue with a `blocks` link to the parent issue (only Gitlab Premium, otherwise `relates_to`)
- Epics are currently translated to normal issues and loosely coupled via labels with their child issues
  - TODO: traslate them into Gitlab epics (only Gitlab Premium)
- Users are mapped from Jira to Gitlab following an explicit mapping in the configuration
  - (Optional) Users can be created automatically in Gitlab
  - Users that could not be mapped / created on Gitlab are impersonated by Gitlab's Administrator, with comments about the original Jira user
  - Users that could not be mapped / created on Gitlab are reported at the end of the import.
  - Users used / created in Gitlab can be given admin rights (configurable) during the import (needed to import timestamps correctly).
At the end of the import, as well as upon unexpected exit, the assigned admin rights are revoked.
Should this last step fail for any reason, a list of admin rights to be revoked manually is reported.
  - **WARNING**: all users that are created in Gitlab are given the password `changeMe` (configurable). You know what to do ;)
- Multi-project import (projects are created automatically, but not groups)
- Interrupted imports can be continued
- Incremental import: it can be run multiple times, it will update issues that have changed since last import (provided that the `import_status.pickle` file from the previous run is available)


## Usage
- Make sure you can use an admin user on Jira
- Create an access token with full rights on Gitlab
- Customize `jira2gitlab_config.py` (check carefully all the options) and `jira2gitlab_secrets.py`
- In order to create a user mapping it might be useful to have a pariticipant list of each project, i.e. the list of users that created, assigned, and/or commented on an issue. You can use a helper script `jira-user-list.py` for that
- Create all required groups and subgroups in Gitlab, according to your project mapping.
The script creates the projects themselves, but not the groups.
- Install the requirements and run the script:
```
pip install -r requirements.txt
./jira2gitlab.py
```
- If the script was interrupted, or if some issues were updated in Jira, you can run the script again.
Only the differences will be imported (as long as you keep the `import_status.pickle` file)

