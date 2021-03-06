# Node.js Core Utilities
[![npm](https://img.shields.io/npm/v/node-core-utils.svg?style=flat-square)](https://npmjs.org/package/node-core-utils)
[![Build Status](https://img.shields.io/travis/nodejs/node-core-utils.svg?style=flat-square)](https://travis-ci.org/nodejs/node-core-utils)
[![AppVeyor Build Status](https://img.shields.io/appveyor/ci/joyeecheung/node-core-utils/master.svg?style=flat-square&logo=appveyor)](https://ci.appveyor.com/project/nodejs/node-core-utils/history)
[![codecov](https://img.shields.io/codecov/c/github/nodejs/node-core-utils.svg?style=flat-square)](https://codecov.io/gh/nodejs/node-core-utils)
[![Known Vulnerabilities](https://snyk.io/test/github/nodejs/node-core-utils/badge.svg?style=flat-square)](https://snyk.io/test/github/nodejs/node-core-utils)

CLI tools for Node.js Core collaborators.

<!-- TOC -->

- [Usage](#usage)
  - [Install](#install)
  - [Setting up credentials](#setting-up-credentials)
  - [Make sure your credentials won't be committed](#make-sure-your-credentials-wont-be-committed)
- [`ncu-config`](#ncu-config)
- [`git-node`](#git-node)
  - [Prerequistes](#prerequistes)
  - [Demo & Usage](#demo--usage)
- [`get-metadata`](#get-metadata)
  - [Git bash for Windows](#git-bash-for-windows)
  - [Features](#features)
- [Contributing](#contributing)
- [License](#license)

<!-- /TOC -->

## Usage

### Install

```
npm install -g node-core-utils
```

If you would prefer to build from the source, install and link:

```
git clone git@github.com:nodejs/node-core-utils.git
cd node-core-utils
npm install
npm link
```

### Setting up credentials

Most of the tools need your GitHub credentials to work. You can either

1. Run any of the tools and you will be asked in a prompt to provide your
  username and password in order to create a personal access token.
2. Or, create a personal access token yourself on GitHub, then set them up
  using an editor.

If you prefer option 2, [follow these instructions](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
to create the token.

Note: We need to read the email of the PR author in order to check if it matches
the email of the commit author. This requires checking the box `user:email` when
you create the personal access token (you can edit the permission later as well).

Then create an rc file (`~/.ncurc` or `$XDG_CONFIG_HOME/ncurc`):

```json
{
  "username": "your_github_username",
  "token": "token_that_you_created"
}
```

Note: you could use `ncu-config` to configure these variables, but it's not
recommended to leave your tokens in your command line history.

### Make sure your credentials won't be committed

Put the following entries into `~/.gitignore_global`

```
.ncurc  # node-core-utils configuration file
.ncu    # node-core-utils working directory
```

Mind that`.ncu/land` could contain your access token since it contains the
serialized configurations.

If you ever accidentally commit your access token on GitHub, you can simply
revoke that token and use a new one.

## `ncu-config`

Configure variables for node-core-utils to use. Global variables are stored
in `~/.ncurc` while local variabels are stored in `$PWD/.ncu/config`.

```
ncu-config <command>

Commands:
  ncu-config set <key> <value>  Set a config variable
  ncu-config get <key>          Get a config variable
  ncu-config list               List the configurations

Options:
  --version  Show version number                                       [boolean]
  --help     Show help                                                 [boolean]
  --global                                            [boolean] [default: false]
```

## `git-node`

A custom Git command for landing pull requests. You can run it as
`git-node` or `git node`. To see the help text, run `git node help`.

### Prerequistes

1. It's a Git command, so make sure you have Git installed, of course.
2. Install [core-validate-commit](https://github.com/nodejs/core-validate-commit)

    ```
    $ npm install -g core-validate-commit
    ```
3. Configure your upstream remote and branch name. By default it assumes your
  remote pointing to https://github.com/nodejs/node is called `upstream`, and
  the branch that you are trying to land PRs on is `master`. If that's not the
  case:

    ```
    $ cd path/to/node/project
    $ ncu-config set upstream your-remote-name
    $ ncu-config set branch your-branch-name 
    ```

### Demo & Usage

1. Landing multiple commits: https://asciinema.org/a/148627
2. Landing one commit: https://asciinema.org/a/157445

```
$ cd path/to/node/project
$ git node land --abort          # Abort a landing session, just in case
$ git node land $PRID            # Start a new landing session

$ git rebase -i upstream/master  # Put `edit` on every commit that's gonna stay

$ git node land --amend          # Regenerate commit messages in HEAD
$ git rebase --continue          # Repeat until the rebase is done

$ git node land --final          # Verify all the commit messages
```

## `get-metadata`

This tool is inspired by Evan Lucas's [node-review](https://github.com/evanlucas/node-review),
although it is a CLI implemented with the GitHub GraphQL API.

```
get-metadata <identifier>

Retrieves metadata for a PR and validates them against nodejs/node PR rules

Options:
  --version         Show version number                                [boolean]
  --owner, -o       GitHub owner of the PR repository                   [string]
  --repo, -r        GitHub repository of the PR                         [string]
  --file, -f        File to write the metadata in                       [string]
  --check-comments  Check for 'LGTM' in comments                       [boolean]
  --max-commits     Number of commits to warn              [number] [default: 3]
  --help, -h        Show help                                          [boolean]
```

Examples:

```bash
PRID=12345

# fetch metadata and run checks on nodejs/node/pull/$PRID
$ get-metadata $PRID
# is equivalent to
$ get-metadata https://github.com/nodejs/node/pull/$PRID
# is equivalent to
$ get-metadata $PRID -o nodejs -r node

# Or, redirect the metadata to a file while see the checks in stderr
$ get-metadata $PRID > msg.txt

# Using it to amend commit messages:
$ get-metadata $PRID -f msg.txt
$ echo -e "$(git show -s --format=%B)\n\n$(cat msg.txt)" > msg.txt
$ git commit --amend -F msg.txt
```

### Git bash for Windows
If you are using `git bash` and having trouble with output use `winpty get-metadata.cmd $PRID`.

current known issues with git bash:
- git bash Lacks colors.
- git bash output duplicates metadata.

### Features

- [x] Generate `PR-URL`
- [x] Generate `Reviewed-By`
- [x] Generate `Fixes`
- [x] Generate `Refs`
- [x] Check for CI runs
- [x] Check if committers match authors
- [x] Check 48-hour wait
- [x] Check two TSC approval for semver-major
- [x] Warn new commits after reviews
- [ ] Check number of files changed (request pre-backport)

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md).

## License

MIT. See [LICENSE](./LICENSE).
