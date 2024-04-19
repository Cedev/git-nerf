# git-nerf

*Buff your release notes*

`git-nerf` is a tool for making changelogs and release notes that are relevant to your users. It strikes a balance between automatically tracking and compiling changes from commits and manually curating your release notes.

`git-nerf` collects changes from commits and memorializes them into release/change files that you can edit. 

## Vision

The following is example documentation for what `git-nerf` might be.

### Memorializing changes

Documentation: `git-nerf memo`

Memorializing changes collects changes from commits and writes them to a file that can be manually edited. The resulting files are also changes that can be collected and modified by `git-nerf`.

`git-nerf --tag <version>`
Compile all of the changes from commits into release notes in preparation for releasing a version with the specified tag. The rusulting memorialized changes will be stored in `changes/tag/<version>/.md`

`git-nerf --branch`
Compile all of the changes on this branch in preparation for merging. This is only necessary if a merge will lose the git history, such as when squashing and mergings. The resulting memorialized changes will be stored in `changes/branch/<branch-name>.md`.

`git-nerf amend <ref>`
Extract the changes from a single commit to be manually editted/amended. By default they are stored in `changes/commit/<hash>.md`

`git-nerf revert <ref>`
If `<ref>` isn't memorialized, record an empty changeset for `<ref>` so that it isn't reflected in release notes. If `<ref>` is memorialized outputs a list of memorialized changes that may need to be edited to reflect that the change has been reverted.

### Querrying changes

`git-nerf status`

Lists unmemorialized changes and memorialized but unreleased changes.

`git-nerf unreleased`

Collects and compiles all of the unreleased changes from git commits and change files and displays it on stdout.

### Publishing changes

`git-nerf publish`
How to publish changes for releases, a changelog, etc

## Parsing

### Git commits

`git-nerf` attempts to detect multiple commits in the same commit message, such as the result of a rebase or a github squash and merge. Identifying them correctly isn't strictly necessary. If you use conventional commit style messages all of the concatenated messages will form a new, correct set of change messages. If you separate changes with `---` or another element that is treated as being top-level by the scoping rules, then the concatenated messages will also form a new, correct set of change messages.

To split multiple commit messages apart `git-nerf` looks for lines that would be treated as git [`trailers`](https://git-scm.com/docs/git-interpret-trailers). These are lines that start with one of the following strings and any subsequent lines that start with at least once whitepsace.

```c
"BREAKING-CHANGE: "
"BREAKING CHANGE: "
"Signed-off-by: "
"(cherry picked from commit "
```
See [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) and [`git/trailer.c`](https://github.com/git/git/blob/ae3196a5ea84a9e88991d576020cf66512487088/trailer.c#L51)

You can change these lines via configuration. 

Multiple consecutive trailers will be treated as being part of the previous message. The contents of a `BREAKING CHANGE: ` will be included with the changes in the previous message.

### Change files

Change files start with an optional metadata heading. If the first line of the file is `---` then a yaml metadata heading will be read from it until the first yaml document separator `---`. This heading is primarily used to store the set of git hashes that a change file memorializes the changes from.

The remainder of the changefile is markdown-formatted changes.

#### Conventional commit

## Data Model

Changes are either unmemorialized `git commit` messages or memorialized change files that replace them.

Changes are markdown structured text that lists the changes in a change set. The headings in the file describe/tag the structure of the changes. Specific heading levels in a file don't have specific meanings, due to developer preferences for markdown presentation. Instead only the outline of the headings matter. For example, the following two files are equivalent.

```
Features
========

###### Changed

- *Swords to plowshares* Melee weapons can now be used for agriculture but not for warfare
```

```
### Features

#### Changed

- *Swords to plowshares* Melee weapons can now be used for agriculture but not for warfare
```

Horizontal rules are treated as headings above the top level, resetting the outline of the file. They are useful for separating additional data that isn't a change.

Inline links in the file link to where the change was made/what issue it's related to. 

A small amount of metadata in each change file contains the set of git hashes the file represents changes for, and some links in the file link to git changes. When the git history is rewriten, via a squash merge, rebase, or other operation, these hashes can become incorrect and refer to hashes outside of the git history. To update these hashes there is a `git-nerf rebase` tool which takes the changed hashes from a rebase and updates all of the hashes in every memorialized change in the repository.

### Configuration

#### Templates

The `memorialize` template will be the default used when memorializing changes.

The `release-notes` template will be the default used when publishing release notes.

The `changelog` template will be the default used when publishing a changelog.

The `maintainence` template will be the default used when publishing a changelog.

## Best Practices

Memorialize changes for releases. Except for corrections in release notes, which would be made manually by editing the memorialized release notes, release notes for a release shouldn't change after a release.

Memorialize changes before performing git operations that remove history, like squash merging.

Don't memorialize changes until you need to. Try to keep each change as small as possible for as long as possible to simplify merging and compiling changes. Prefer using `git-nerf amend` over memorializing all of the changes and editing them. Prefer using `git amend` over `git-nerf amend` if the change is the most recent, hasn't been seen by other users/repos, and hasn't been memorialized yet.

Create change files for your front matter and appendixes of releases that are separate from the compiled changes. You can add any change files you want anywhere in the `changes/` directory and they will be collected along with changes from commits, other memorialized changes. A good place to keep changes for a branch is in `changes/branch/<branch-name>/`.

Don't flood your users with impertinent details like the build and maintainence chores for your repository. Instead collect them into a separate maintainence log and publish it as soon as relevant changes have been merged into the repository. You can control which topics are collected and published into which files by the use of templates.

