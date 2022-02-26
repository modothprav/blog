---
title: How to Move files among different repos and Retain Git History
date: 2022-02-26
tags: ["git", "programming"]
---

When working in a multi-repository or large mono-repo project eventually developers will have to refactor out some files and folders across from one project to another.

As developers reach a point where they have to start splitting up the large repository into smaller sub-repos, preserving the commit history during this migration process can be very handy for future bugs and adjustments. 

As you may have already noticed, when simply trying to move files from one repo to another we lose any history correlated to that file and are unable to trace back any changes.

The following steps outlined below will demonstrate how to achieve history retention during file migration.

![Cover](https://images.unsplash.com/photo-1556075798-4825dfaaf498?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2076&q=80)

# Prerequisites

An additional tool called `git-filter-repo` will be used to prepare the repository *to be* migrated by filtering out only the necessary files and commits.

For this tool to function as intended git and python should be installed and added to the systems path.

## Installation

A more detailed set of instructions can be found in the tool’s [GitHub](https://github.com/newren/git-filter-repo.git) repo

### Step 1

Clone the toll from [GitHub](https://github.com/newren/git-filter-repo.git) or download the files manually.

```bash
git clone https://github.com/newren/git-filter-repo.git
```

### Step 2 (Optional)

Edit the first line of the file called `git-filter-repo` and replace `python3` with `python` depending on your Python installation you may skip this step.

> **Note:** Come back to this step if the setup was unsuccessful.
> 

### Step 3

Next type in the command `git --exec-path` and paste the file `git-filter-repo` into the outputted location. In this case the outputted location being `/usr/local/libexec/git-core`

```bash
git --exec-path
> /usr/local/libexec/git-core
```

```bash
cp git-filter-repo /usr/local/libexec/git-core/git-filter-repo
```

> **Note:** Ensure the file permissions on `git-filter-repo` are the same as all the other executables under the `git-core` folder.
>

```bash
sudo chmod 755 /usr/local/libexec/git-core/git-filter-repo
```

### Step 4

Check if the installation was successful, the following output should be observed

```bash
git-filter-repo
No arguments specified.
```

> **Note:** The `--help` option will not work but there is [documentation](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html) available online.
> 

# Moving Files while preserving Git History

![Cover2](https://images.unsplash.com/photo-1447069387593-a5de0862481e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1469&q=80)

### Step 1

Clone or download the source repository from which the files are being filtered and extracted e.g.

```bash
git clone <Original repo URL>
```

### Step 2

Next, create *another* directory and initialise it as a Git repository with no commits. 

```bash
mkdir filtered-repo
cd filtered-repo
git init
```

### Step 3

Perform the filtering on the repo that was initially cloned and save the changes in the newly initialised Git repo. This is done by setting the `--source` and `--target` repo options when using `git filter-repo` e.g.

```bash
--source <Original repo path> --target filtered-repo/
```

There are a number of filtering options that can be found in the [documentation](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html), the most common being `--path` and `--path-rename`

> **Note:** Much like git itself renames are not followed through. Hence let's say during development a file was renamed from `index.js` to `server.js` and the filter command only specified the path for `server.js`, the tool will now only filter out the commit history up until the point it was renamed.
> 

Therefore all file history prior to the rename when `server.js` was called `index.js` will ***not*** be preserved.

To get around this issue multiple paths will need to be specified in the filter command using the `--path` option.

```bash
--path olddir/ --path newdir/
```

In this case, to apply the command for the situation described above would be as shown below.

```bash
git filter-repo --path server.js --path index.js
```

> **Note:** The `--path-rename` option is also very useful as it helps contain all the files within a resultant sub-directory of your filtered repo, which makes pulling commits from the filtered repo to the new project to be much smoother.
> 

```bash
--path-rename <old path>:<new path>
```

#### Examples

Below are some examples of what the final filtering command may look like. More can be found in the [user manual](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html). 

This command ensures that commits and files relating only to the `lib-encrypt` directory will be filtered into the target repo (filtered-repo).

```bash
git filter-repo --source <Original>  --target filtered-repo
    --path libs/lib-encrypt
    --path-rename libs/lib-encrypt:lib-encrypt
    --preserve-commit-hashes
```

However the command above will not retain any commit history prior to any renames, hence multiple paths will need to be specified as shown below.

```bash
git filter-repo --source <Original>  --target filtered-repo
    --path libs/lib-encrypt 
    --path libs/lib-base64 
    --path-rename libs/lib-encrypt:lib-encrypt
    --path-rename libs/lib-base64:lib-encrypt
    --preserve-commit-hashes
```

As can be seen from the command above `lib-encrypt` used to be named `lib-base64` hence in order to retain all history prior to this rename multiple paths have been specified.

With the command above the outcome can be observed in the target repo (filtered-repo) where all the filtered files will be saved in the following structure `<project root>/lib-encrypt/*` as specified by the`--path-rename` option.

> **Note:** When specifying multiple `paths` to filter the same number of `renames` should be specified to bring all filtered files into one subdirectory.
> 

### Step 4

Once all the filtered files have been saved in the target repo (filtered-repo) within a sub-directory (e.g. lib-encrypt), navigate to a cloned location of your new project where you want to migrate the files to.

```bash
git clone <New Project URL>
cd <new project>
```

Once present add another remote repository and set the local location of `filtered-repo` as the remote repo location.

```bash
git remote add modified-source ../filtered-repo
```

Once added pull commits from this repo. This will merge the commits from the `filtered-repo` into your new project.

```bash
git pull modified-source master --allow-unrelated-histories
```

Finally, make any necessary final changes and push to origin.

```bash
git remote rm modified-source
git push origin -u -all
```