## Branching Model

Branching model are 4 kinds of branch :

* **master** : Master branch contains the latest deployable version. This is the only infinite lifetime branch.
* **feature/xxx** : Feature branches are dedicated branch for one big feature (lot of commits), "xxx" is the feature name.
* **fix/xxx** : Fix branch is dedicated to integrate bugfix on Master branch. If needed the fix is then cherry pick on stable branch.
* **poc/xxx** : Poc branches are dedicated branch to develop a Prove of Concept (PoC).

<img src="https://github.com/Popety/Raneto/tree/master/themes/default/public/images/Git-Workflow.png" style="text-align:left; display:inline"/>

## Master Branch

Master branch contains the latest deployable version. It's the only infinite lifetime branch in our branching model.
This is our backbone where all the different fix and new feature are mixed with each other.

## Feature Branch

Feature branches are dedicated branch to develop a new feature. This is a short/medium lifetime branch deleted as soon feature is merge to master.

A feature branch is always created from Master branch.

### Actions

#### Create a new Feature Branch

**When:** A new feature is specified and planified.
> be careful to use a good name for the feature branch. For instance ```feature/facebook-authentication``` is a good name, ```feature/6th-august``` is not.

**Who:** Any developer

**How:**
```
git checkout master
git pull
git checkout -b feature/xxx
git push
```

#### Rebase Master to Feature Branch

**When:** Frequently.

**Who:** Developer responsible of the branch. Before rebasing a feature branch, be sure that any developer working on this branch push its latest commit to Github.

**How:**
```
git checkout master
git pull
git checkout feature/xxx
git rebase master
git push --force
```

#### Merge Feature Branch to Master

**When:** Feature has been successfully tested on QA. PR has been LGTM by another developer.

**Who:** Developer responsible of the branch.

**How:**
```
git checkout feature/xxx
git pull
git rebase origin/master
git checkout master
git pull
git merge --no-ff feature/xxx
git push
```

#### Remove a Feature Branch

**When:** Just after the merge of the feature branch to Master

**Who:** Developer responsible of the branch.

**How:**
```
git branch -d feature/xxx
git push origin --delete feature/xxx
```

## Fix Branch

Fix Branch are dedicated branch to fix a bug. This is a short lifetime branch deleted as soon fix is merge to master.

A fix branch is always created from Master branch.

### Actions

#### Create a Fix Branch

**When:** A Jira issue has been created, time to resolve it is already estimated.
> be careful to use a good name for the fix branch. The fix branch must contains the Jira ticket, for instance ```fix/POPETY2-386```.

**Who:** Developer responsible to fix the issue.

**How:**
```
git checkout master
git pull
git checkout -b fix/yyy
git push
```

#### Merge a Fix Branch to Master

**When:** Fix has been successfully tested. PR has been LGTM by another developer.

**Who:** Developer responsible to fix the issue.

**How:**
```
git checkout fix/yyy
git pull
git rebase origin/master
git checkout master
git pull
git merge fix/yyy --squash
git commit -a
git push
```

#### Remove a Fix Branch

**When:** After the merge of the fix branch to Master

**Who:** Developer responsible to fix the issue.

**How:**
```
git push origin --delete fix/yyy
git branch -d fix/yyy
```

## PoC Branch

Poc branches are dedicated branch to develop a Prove of Concept (PoC).

### Actions

#### Create a new PoC Branch

**When:** A new PoC is planified.

**Who:** Developer

**How:**
```
git checkout master
git pull
git checkout -b poc/zzz
git push
```

## Release Process

This section explain the release process to follow to tag the release on Github.

### Perform a release

**When:** After successful test campaign.

**Who:** Developer

**How:**
```
git tag -a vx.y.z
# Enter the description as described below
git push origin --tags
```

Description example:
```
[app_name] Web App Version 1.0.0

RELEASE NOTE

Feature:

Improvement:

Fix:
- Remove New Relic agent
```
