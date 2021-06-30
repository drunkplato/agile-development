# Agile Development

These are notes from previous companies on the dev process.

## Table of contents

* [Project Managment](#pm)
* [Branching](#branching)
* [Tagging](#tagging)
* [Code Review](#codereview)
* [Continuous Delivery](#ci)
* [Code Style](#codestyle)
* [Atom Plugins](#atom)
* [Testing](#testing)

### <a id="pm"> Project Management </a>
We use our own agile-like approach to development. The meeting cadence is below.

| Meeting | Occurence | Description 
|-|-|-|
| Standup | Daily | A quick update on progress from yesterday, plans for today, and any blockers |
| Sprint Planning | Alternating bi-weekly | Take a look at the features and checkpoints and prioritize; plan the next sprint |
| Backlog Refinement | Alternating bi-weekly | Take a look at our backlog and prioritize |
| Review | Bi-weekly | Take a look at the progress from the previous two weeks. Set feature release deadlines |
| Retro | Bi-weekly | Quick discussion going over what went well, what needs to be improved, and any additional comments |

### User Stories
Our user stories have the following point scale

| 1 | 2 | 4 | 8 | 16 
|-|-|-|-|-|
| X-Small (1/2 day or less) | Small (1-2 days) | Medium (2-4 days) | Large (5-10 days) | Extra-Large (more than 2 weeks) |

#### Structure

Not all parts of this have to be used, but this is the general structure of a JIRA story.
```
Story
<goes here>

Ex: As a user, I want to be able to expand the nav bar in order to see the names of pages and potentially have dropdown navs so I can better navigate the platform.

Links
<goes here>

Design
<attach image if needed>

Details
<additional information if needed>

Done if
<list the requirements of the story>

Ex:
 - Sidebar is collapsed by default.
 - Sidebar last state was saved in localstorage… so if it was collapsed, store collapsed for next page load.
 - Sidebar snaps closed or open depending on how close the cursor is from the breakpoint.
 - Make sure all pages are wrapped in this component.
 - Has storybook
```

### <a id="branching"> Branching </a>

```
production                                  # Most stable build. Deploys to production (tagged with release version)
  └── staging                               # Deploys to staging
      ├── hotfixes-DSP-1128                 # Are created just to fix a bug in production
      └── development
          ├── DSP-546-subdomains            # self-contained story
          ├── feature-public-sharing        # feature branch
          |   └── DSP-547-dashboards        # story
          └── DSP-547-sso
```

### *`production`* branch
Our `production` branch which is supposed to contain the most *stable* and *logically complete* version of the code which is tagged as v1.0, v2.0 etc. Any hotfixes will increment the version number.

__*NOTE*__: this branch is locked by default and require a pull request from the `staging` branch.

As soon as the code is committed to this branch, our CI *(Continous Integration)* setup, if configured, triggers a deployment for the production. 

### `staging` branch
This branch mirrors the staging environment and is used to keep a version of the code isolated from `development`, but also in a place where `hotfix-<x>` branches can be easily merged. 

### `hotfixes` branch
In an agile-*ish* company like ours, it is pretty normal to find ourselves in a situation where we need to release a hotfix for production quickly, regardless of the state of the dev branch. Thus, if and when needed, we create a `hotfixes` branch from `staging`, troubleshoot and fix the problems with the code and *up-merge* the code to `production` through `staging`.

At the same time, the changes from `hotfixes` branch is also *down-merged* to the `development` branch so the next wave of changes coming from dev doesn't miss the hotfix or its ripples. Everytime we push a hotfix to staging, we increment the the version.

After the branch has been merged in both the `production` as well as the `development` branches, the hotfixes branch is deleted to cleanup the repo.

### `development` branch
This is the main branch that we work off of, and represents the latest version of the code. 

__*NOTE*__: these branches are locked by default and require a pull request.

As soon as the code is committed to this branch, our CI *(Continous Integration)* setup triggers a deployment for the staging environment.

### `DSP-<story/task id>` branch
For larger stories, we break stories into smaller tasks and those tasks are merged into the feature-<name>. Also, there are cases when stories are small and do not necessarily fit into a feature. In that case, we would branch off of the release branch and create a pull request back into it when complete.

| Jira story id | example
|--|--|
| DSP-1 | DSP-1-model-cards
| DSP-1123 | DSP-1123-deployment

JIRA will automatically change the status of stories when certain actions are performed in Github. 
- Backlog -> In Progress: when a branch is created starting with the story id, for example DSP-1123-deployment
- In Progress -> Review: when a pull request is created with the story id in the title
- Review -> Done: when the pull request from the last step is merged into development

### <a id="tagging"> [Tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging) </a>
We use tags to keep track of versions using *Git* tags. The way we do this is by incrementing the version number everytime we introduce any changes to the production branch. 

__For example:__
If we have release 1.0 in the `production` branch and we introduce a hotfix, we would increment the version number of the release by 1. 
- `hotfixes-r1-1-DSP-1323` This is release 1 and the first hotfix, therefore the version number would be `v1.1`.
- `hotfixes-r1-2-DSP-1324` This is release 1 and the second hotfix, therefore the version number would be `v1.2`.
- `hotfixes-r2-1-DSP-1325` This is release 2 and the first hotfix, therefore the version number would be `v2.1`.

```sh
git tag -a v1.1 -m "hotfixes-r1-1-DSP-1323"
```

This is mostly done automatically now (though maybe not for hotfixes??)

### <a id="codereview"> Code Review </a>
When you have completed your story and made sure the tests run on your branch, you can submit a pull request into the `development` branch. However, if you are working on a feature branch that the story is branched off of, then you would submit a pull request to the branch `feature-<name>` and assign it to someone for review.

See [Github documentation](https://help.github.com/articles/creating-a-pull-request/) for a guide on how to create pull requests.

### <a id="ci"> Continuous Delivery </a>
We use [CircleCI]() for our CI/CD pipeline. This allows us to deploy builds in an automated way.

Our CICD process works as such:

##### When pushing to the `development` branch.
```
    test ──> build-staging ──> deploy-staging
```
##### When pushing to the `production` branch.
We update the staging environment to the production build for final QA and then after manual approval, we deploy to production.
```
    test ──> build-prod ──> approve (manual) ──> deploy-prod
     │
     └─────> build-staging ──> depoy-staging
```
### <a id="codestyle"> Code Style </a>
**__NOT FINISHED YET__**

See file [Code Style](CodeStyle.md).

### <a id="testing"> Testing </a>
To be honest, this is an area where we are aiming to improve. As of now, we are very good at integration testing, but need more work in unit testing and system testing. Please do your best to write unit tests for any new code that you write.

- __Unit testing__: refers to tests that verify the functionality of a specific section of code, usually at the function level.

- __Integration testing__:  is any type of software testing that seeks to verify the interfaces between components against a software design. This would be testing the flow of the creation, listing, reading, and deleting of a job record via apis.

- __System testing__: is a completely integrated test to verify that the system meets its requirements.  For example, a system test might involve testing a login api, then creating and editing an entry, plus sending or printing results, followed by summary processing or deletion (or archiving) of entries, then logoff.

- __Acceptance testing__: developers and others within the company test the features manually and give feedback and list bugs. 
