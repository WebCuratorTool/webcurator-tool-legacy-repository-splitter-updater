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
project. They use the tasks defined by that project to perform the repository splits operations.

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

# For createSplitTargetRepositories, extractAndPatchProjects, createAndApplyPatchesOnly and applyPatchesOnly
patchTargetRepositoryFolder="/my/repository-split-work/patch-repositories"
patchTargetRepositoryBranch="split-work-base-branch-2018-xx-xx"
patchTargetRepositoryBaseBranch="split-work-base-branch-2018-10-21"
createPatchTargetRepositoryBranch=true
preserveBranchNames="split-work-base-branch-2018-10-21"
tempWorkFolder="/my/temp-folder"
reportsFolder="/my/reports-folder"
autocreateReportsFolder="true"

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
 --stacktrace --info
```

### cloneSplitTargetRepositories

Clones the split-target repositories into the given folder. The repository names are determined by the project names
found in the `projectParametersJsonFile` file in combination with the includeProjectNames and excludeProjectNames. Note
that the final branch name in preserveBranchNames is the one that is checked out after the repository is cloned.

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
  --stacktrace --info
```

#### createSplitTargetRepositories

Creates the split-target repositories from the source repository based on the given parameteres.
This does all RepositoryWorkflow operations and produces a series of repositories.

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
  --stacktrace --info
```

## Versioning

This pipeline does not produce any artifacts so no version is required.

## Contributors

See git commits to see who contributors are. Issues are tracked through the github issue tracking.

## License

&copy; 2018 National Library of New Zealand. MIT License. All rights reserved.
