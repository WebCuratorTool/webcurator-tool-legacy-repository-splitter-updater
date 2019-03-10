# webcurator-tool-legacy-repository-splitter-updater
Tool for splitting the legacy webcurator tool repository into multiple component-based repositories.

## Motivation

The WebCurator Tool repository contains several different applications (the tool itself, a storage application, a
Heritrix 1 agent and crawler, and a Heritrix 3 agent and crawler) as well as documentation and other components. These
different applications work together, but are loosely coupled. It makes more sense to work with these different
components with each having its own repository. Another motivation is that the existing WebCurator Tool repository
is extremely large and may contain unnecessary commits.

The first step is to actually split the repository into the different components. Once the split has been made, and
while users are still working in the legacy repository, a patching process keeps the split repositories up to date with
changes in the legacy repository. This updating process is abandoned once the all new development takes place in the split
repositories.

## Requirements

Splitting and updating requires running the nlnz-m11n-tools-repository-splitter-updater with certain parameters.

## Parameters

The default split-target repository parameters are found in `src/main/resources/defaultProjectParameters.json`.

## Example operations

These example operations should be executed from the root folder of the `nlnz-m11n-tools-repository-splitter-updater`
project (currently found at https://github.com/NLNZDigitalPreservation/nlnz-m11n-tools-repository-splitter-updater).
They use the tasks defined by that project to perform the repository splits operations.

## Recommended process

### First time

1. `cloneSourceRepository`.
2. See *Create initial base branch in source repository*.
3. `createSplitTargetRepositories` (first time, since these repositories haven't been created yet).
4. `extractAndPatchProjects`.
5. See *Push newly created repositories to github or other code management system*.
6. See *Create patch-merge branch and merge into master branch*.

### Subsequent updates

1. `cloneSourceRepository`.
2. See *Create appropriate to branch in the source repository*.
3. `cloneSplitTargetRepositories`.
4. See *Create appropriate branch in the split-target source repository*.
5. `extractAndPatchProjects`.
6. See *Push changes to github or other code management system*.
7. See *Create patch-merge branch and merge into master branch*.

### Create initial base branch in source repository

To make this process workable in the long-term, it's best to have a fixed base branch in the source branch to work from.
This branch would be based on the `master` branch at a certain point in time. The GUI provided by github or bitbucket
may not be able to specify the source branch, but since it defaults to `master` that may not be an issue. Otherwise,
we can perform the operation on the command line. The source repository must be cloned first and then perform the
following operation:
```
git checkout -b "<source-initial-patch-base-branch-name>" origin/master
git push origin HEAD
```

For purposes of documentation, we will refer to this initial patch branch as the *source-initial-patch-base*.

### Create appropriate to branch in the source repository

For subsequent patching operations, there needs to be a *to* branch for patching operations to occur. The patches are
created based on the differences between the *from* branch (which is the *source-initial-patch-base* branch) and the
*to* branch. For purposes of documentation we will refer to this subsequent patch branch as the *source-patch-point*
branch. This *to* or *source-patch-point* branch is created the same way as the *source-initial-patch-base* branch,
but this time it is based on the *master* branch at the commit point that we want to capture. Note that we could also
use a specific commit or tag as the commit point.

As before, the source repository must be cloned first. We then perform the following operation:
```
git checkout -b "<source-patch-point-branch-name>" origin/master
git push origin HEAD
```

### Create appropriate branch in the split-target source repository

Case 1:
If the split-target repository is newly created, we create a *split-target-patch-point* branch based off the *master*
branch:
```
git checkout -b "<split-target-patch-point-branch-name>" origin/master
git push origin HEAD
```

Case 2:
If the split-target source repository already exists, it would have a *split-target-patch-point* branch where the
initial patches have been applied. For subsequent patching operations, we want to create a branch based on that *last*
(or most recent) *split-target-patch-point* branch where we apply the new sets of patches. The GUI provided by github or
bitbucket may not be able to specify the source branch, so it may be necessary to perform the operation on the command
line. The split-target repositories must be cloned first and then in each of those cloned split-target repositories,
perform the following operation:
```
git checkout -b "<new-split-target-patch-point>" origin/<most-recent-split-target-patch-point>
git push origin HEAD
```

### Push changes to github or other code management system

Even if there are no changes to the patched branch, we still want this branch to exist in the target repository, as it
becomes the starting point for the next set of patch updates.

If the repository origin has been setup correctly, simply issue the command:
```
git push orgin HEAD
```

### Create patch-merge branch and merge into master branch

After the branch that contains the patches from the source repository (the new *split-target-patch-point* branch) has
been created and patched, this branch will need to be merged into the *master* (or development) branch. Even though the
branch may not be the master branch, for ease of documentation we'll refer to it as the *master* branch. We'll call the
branch that gets merged into the master branch the *patch-merge-branch*. It is *not* the *split-target-patch-point*
branch. The *patch-merge* branch is where the files that came from the source repository are moved and manipulated to
match the structure of the *master* branch. For example, you may need to change the directory structure or alter files
to fit the new project's direction and development plan. Eventually the *patch-merge branch* gets merged into the
*master* branch and the *patch-merge branch* gets deleted.

Because the *patch-merge* branch is based on the *master* branch, when the most recent *split-target-patch-point* branch
is merged into it, git will perform the same manipulations on the files that are being merged in. For example, a file is
moved to a new development structure and updated to reflect an underlying development need. That file movement is done
in commits in the *master* branch. Subsequent merges of a new *patch-merge* branch into the *master* branch will have
those changes applied to that file in its new location.

The only changes that will not occur are new files. New files from the source repository will need to be moved to
their new locations just like the other files that were moved there (most likely by using `git mv`). This means that
the *patch-merge* branch needs to be checked to make sure its structure is correct before it gets merged into the
*master* branch.

We want to keep the *split-target-patch-point* branch in the split-target repository, as these are the difference points
for subsequent updates. In other words, don't delete te *split-target-patch-point* branch! We first checkout the
*master* branch, do a fetch and pull to ensure that it is up to date. We then create a merge branch based on the
*master* branch*:
```
git checkout master
git fetch
git pull
git checkout --no-track -b "patch-branch-<name>-merge-into-master-branch" origin/master
git push origin HEAD
```

We then merge the patch branch into the merge branch that we have just created. The option `--no-edit` is to
automatically keep the generated commit message:
```
git merge origin/<most-recent-split-target-patch-point-branch-name> --no-edit
```

There may be merge conflicts. Resolve those conflicts. Move any necessary files to the correct location. Once the
conflicts have been resolved and committed, push up the merge branch:
```
git push origin HEAD
```

Then create a pull request to merge it into the *master* branch. The merge of the pull request should include the
deletion of the merge branch.

When using github to merge the pull request you may consider using either *Merge pull request* or *Rebase and merge* as
described by https://help.github.com/articles/about-pull-request-merges/. It depends on what kind of commit history
you want on the *master* branch.

If there are no changes between the *patch-merge* branch and the *master* branch (which means that there are no changes
between the most recent *split-target-patch-point* branch and the next most recent *split-target-patch-point* branch,
then there's no need to create a pull request (in fact, it can't be done because there is nothing to merge) and the
*patch-merge* branch can simply be deleted (as it is identical to the *master* branch). 

### Setting common variables

These are the commonly used variables. Note that the `projectParametersJsonFile` will be the absolute path to that file.

```
# For cloneSourceRepository
autocreateSourceRepositoryWorkingFolder=true
preserveBranchNames="split-work-base-branch-2018-10-21"
sourceCodeRepositoryServerUrl="git@github.com:DIA-NZ/webcurator.git"

# For cloneSourceRepository, createSplitTargetRepositories, extractAndPatchProjects, createAndApplyPatchesOnly and applyPatchesOnly
sourceRepositoryGitBranch="split-work-base-branch-2018-10-21"
sourceRepositoryWorkingFolder="/my/repository-split-work"
sourceRepositoryNameBase="webcurator"

# For cloneSplitTargetRepositories
cloneSplitTargetRepositoriesRemoveRemoteOrigin=false

# For cloneSplitTargetRepositories, createSplitTargetRepositories, extractAndPatchProjects, createAndApplyPatchesOnly and applyPatchesOnly
projectParametersJsonFile="/my-git-repositories/webcurator-tool-legacy-repository-splitter-updater/src/main/resources/default-projects-parameters.json"
splitTargetProjectsRootFolder="/my/split-repositories-work"
autocreateSplitTargetProjectsRootFolder="true"
includeProjectNames=
excludeProjectNames=
showCommandOutput="true"

# For createSplitTargetRepositories, extractAndPatchProjects, createAndApplyPatchesOnly and applyPatchesOnly
patchTargetRepositoryFolder="/my/repository-split-work/patch-repositories"
patchTargetRepositoryBranch="split-work-base-branch-2018-xx-xx"
patchTargetRepositoryBaseBranch="split-work-base-branch-2018-10-21"
createPatchTargetRepositoryBranch=true
preserveBranchNames="split-work-base-branch-2018-10-21"
tempWorkFolder="/my/temp-folder"
reportsFolder="/my/reports-folder"
autocreateReportsFolder="true"

# For cloneSplitTargetRepositories, when those repositories already exist
# splitTargetRepositoryServerUrlBase=[split-target-repository-server-url-base]
cloneSplitTargetRepositoriesRemoveRemoteOrigin="false"
# Cloning using HTTPS:
#splitTargetRepositoryServerUrlBase="https://github.com/WebCuratorTool/"
# Cloning using SSH: (if .ssh setup correctly, can easily push to origin)
splitTargetRepositoryServerUrlBase="git@github.com:WebCuratorTool/"

# For applyPatchesOnly and (applyPatchesOnly in combination with cloneRepositories)
repositoryWorkflowStartingSequenceIndex="3"
patchesFolderPath="/path/to/my/special/edited/patches/folder"
```

### cloneSourceRepository

Clones the source repository into sourceRepositoryWorkingFolder. The repository name is
"${sourceRepositoryNameBase}-${repositoryWorkflowStartingSequenceIndex}". By default this would be
"${sourceRepositoryNameBase}-0".

Example:
```
gradle cloneSourceRepository \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PautocreateSourceRepositoryWorkingFolder=true \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PsourceCodeRepositoryServerUrl="${sourceCodeRepositoryServerUrl}" \
  -PshowCommandOutput="${showCommandOutput}" \
  --stacktrace --info
```

#### createSplitTargetRepositories (when the split-target repositories don't yet exist)

Creates the split-target repositories from the source repository based on the given parameteres.
This does all RepositoryWorkflow operations and produces a series of repositories.

*NOTE* that if these repositories already exist then it's better to use `cloneSplitTargetRepositories`.

Example:
```
repositoryWorkflowStartingSequenceIndex="0"

gradle createSplitTargetRepositories \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  -PshowCommandOutput="${showCommandOutput}" \
  --stacktrace --info
```

### cloneSplitTargetRepositories (when the split-target repositories already exist)

Clones the split-target repositories into the given folder. The repository names are determined by the project names
found in the `projectParametersJsonFile` file in combination with the includeProjectNames and excludeProjectNames. Note
that the final branch name in preserveBranchNames is the one that is checked out after the repository is cloned.

*NOTE* that this only works if those repositories already exist. If the split-target repositories don't exist yet,
use `createSplitTargetRepositories`.

Example:
```
gradle cloneSplitTargetRepositories \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder="${autocreateSplitTargetProjectsRootFolder}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PcloneSplitTargetRepositoriesRemoveRemoteOrigin=${cloneSplitTargetRepositoriesRemoveRemoteOrigin} \
  -PsplitTargetRepositoryServerUrlBase=${splitTargetRepositoryServerUrlBase} \
  -PshowCommandOutput="${showCommandOutput}" \
  --stacktrace --info
```

### extractAndPatchProjects

Extracts and patches a given set of projects. This does all RepositoryWorkflow operations.

Example:
```
repositoryWorkflowStartingSequenceIndex="0"

gradle extractAndPatchProjects \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  -PshowCommandOutput="${showCommandOutput}" \
  --stacktrace --info
```

### createAndApplyPatchesOnly

Only creates and applies the patches. Does not do the source repository filter-branch and other operations.

```
repositoryWorkflowStartingSequenceIndex="3"

gradle createAndApplyPatchesOnly \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  -PshowCommandOutput="${showCommandOutput}" \
  --stacktrace --info
```

### applyPatchesOnly

Only applies the patches. Does not do the source repository filter-branch and other operations. This operation would be
used when there has been some manual editing of the patch files and an attempt is made to apply the patches to
the project repository clone. These manually edited patched would be found in `patchesFolderPath`.

```
repositoryWorkflowStartingSequenceIndex="3"
patchesFolderPath="/path/to/my/special/edited/patches/folder"

gradle applyPatchesOnly \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PpatchesFolderPath="${patchesFolderPath}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  -PshowCommandOutput="${showCommandOutput}" \
  --stacktrace --info
```

### applyPatchesOnly in combination with cloneRepositories

When trying to get a sequence of patches to work it's probably best to clone the target project repository and try
to apply the manually edited patches. This operation would be used when there has been some manual editing of the patch
files and an attempt is made to apply the patches to the project repository clone. These manually edited patched would
be found in 'patchesFolderPath'.

```
repositoryWorkflowStartingSequenceIndex="3"
cloneRepositoriesRemoveRemoteOrigin=true

gradle cloneRepositories applyPatchesOnly \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PpatchesFolderPath="${patchesFolderPath}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PcloneRepositoriesRemoveRemoteOrigin="${cloneRepositoriesRemoveRemoteOrigin}"
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  -PshowCommandOutput="${showCommandOutput}" \
  --stacktrace --info
```

## Versioning

This pipeline does not produce any artifacts so no version is required.

## Contributors

See git commits to see who contributors are. Issues are tracked through the github issue tracking.

## License

&copy; 2018 -- 2019 National Library of New Zealand. MIT License. All rights reserved.
