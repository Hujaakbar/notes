# GitHub CODEOWNERS

*Official docs: [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners).*

- `CODEOWNERS` file is used to define individuals or teams that are responsible for code in a repository (hence, the name "codeowners").
- `CODEOWNERS` file is evaluated by git hosting platforms like GitHub.
- It is not a git feature. So, git treats `CODEOWNERS` file as a regular file.
- Code owners are automatically requested for review when someone opens a pull request that modifies code that they own.
- In GitHub it is possible to set branch rulesets in away that pull requests have to be approved by code owners before they can be merged.

## Terminology

**Base branch** is the branch that a pull request will modify if the pull request is merged.

Base branch can be said a destination branch.

## Creating CODEOWNERS file

To use a `CODEOWNERS` file, create a new file called `CODEOWNERS` in the `.github/`, root, or `docs/` directory of the repository.

Syntax:

```txt
path/to/file/or/directory    @codeowner_username1  @codeowner_username2
```

- Paths are case sensitive
- If any line in the CODEOWNERS file contains invalid syntax, that line will be skipped

> Note:
> Even though the `CODEOWNERS` file syntax is very similar to `.gitignore` file syntax. They ar not the same, there are some differences.
>
> - Using `!` to negate a pattern doesn't work
> - Using `[ ]` to define a character range doesn't work
> - Escaping a pattern starting with `#` using `\` (so it is treated as a pattern, not a comment) doesn't work

Example:

```txt
# This is a comment.
# Each line is a file pattern followed by one or more owners.

#------------------------------------------------------------

*       @global-owner1   @global-owner2

    #   @global-owner1 and @global-owner2 will be the default owners
    #   for everything in the repo.

    #   Unless a later match takes precedence, @global-owner1
    #   and @global-owner2 will be requested for
    #   review when someone opens a pull request.

#-----------------------------------------------------------

*.js    @js-owner      # This is an inline comment.

    #   Order is important!
    #   The last matching pattern takes the most precedence.
    #   When someone opens a pull request that only
    #   modifies JS files, only @js-owner and not the global
    #   owner(s) will be requested for a review.

#------------------------------------------------------------

/apps/    @userA
/apps/github    @userB

    #   In this example, @userA owns any file in the `/apps`
    #   directory in the root of the repository except for the `/apps/github`
    #   subdirectory, as this subdirectory has its own owner @userB

#------------------------------------------------------------

/apps/   @octocat
/apps/github

    #   In this example, @octocat owns any file in the `/apps`
    #   directory in the root of the repository except for the `/apps/github`
    #   subdirectory, as its owners are left empty.

    #   Without an owner, changes to `apps/github` can be made
    #   with the approval of any user who has write access
    #   to the repository.

#------------------------------------------------------------

/apps/    @octocat

    #   In this example, @octocat owns any file in the `/apps`
    #   directory in the root of the repository and any of nested
    #   subdirectories of `/apps` such as `apps/frontend/index.html`

#-------------------------------------------------------------

apps/ @octocat

    #   In this example, @octocat owns any file in an apps directory
    #   anywhere in the repository.

#-------------------------------------------------------------

docs/*    @userA

    #   The `docs/*` pattern will match files like
    #   `docs/getting-started.md` but not further nested files like
    #   `docs/build-app/troubleshooting.md`.

#-------------------------------------------------------------

**/logs   @octocat

    #   In this example, @octocat owns any file in a `/logs` directory such as
    #   `/build/logs`, `/scripts/logs`, and `/deeply/nested/logs`.

```
