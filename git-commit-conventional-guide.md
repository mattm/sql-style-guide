# Conventional Commit Messages 
See how a minor change to your commit message style can make a difference. [Examples](#examples)

**Have a look at CLI util [git-conventional-commits](https://github.com/qoomon/git-conventional-commits) to ensure this conventions and generate changelogs**

<img src="https://img.icons8.com/dusk/1600/commit-git.png" width="200" height="200" />

## Commit Formats

### Default
<pre>
<b><a href="#types">&lt;type&gt;</a></b></font>(<b><a href="#scopes">&lt;optional scope&gt;</a></b>): <b><a href="#subject">&lt;subject&gt;</a></b>
<sub>empty separator line</sub>
<b><a href="#body">&lt;optional body&gt;</a></b>
<sub>empty separator line</sub>
<b><a href="#footer">&lt;optional footer&gt;</a></b>
</pre>

### Merge
<pre>
Merge branch '<b>&lt;branch name&gt;</b>'
</pre>
<sup>Follows default git merge message</sup>


### Types
* API relevant changes
    * `feat` Commits, that adds a new feature
    * `fix` Commits, that fixes a bug
* `refactor` Commits, that rewrite/restructure your code, however does not change any behaviour
    * `perf` Commits are special `refactor` commits, that improve performance
* `style` Commits, that do not affect the meaning (white-space, formatting, missing semi-colons, etc)
* `test` Commits, that add missing tests or correcting existing tests
* `docs` Commits, that affect documentation only
* `build` Commits, that affect build components like build tool, ci pipeline, dependencies, project version, ...
* `ops` Commits, that affect operational components like infrastructure, deployment, backup, recovery, ...
* `chore` Miscellaneous commits e.g. modifying `.gitignore`

### Scopes
The `scope` provides additional contextual information.
* Is an **optional** part of the format
* Allowed Scopes depends on the specific project
* Don't use issue identifiers as scopes

### Breaking Changes Indicator
Breaking changes should be indicated by an `!` before the `:` in the subject line e.g. `feat(api)!: remove status endpoint`
* Is an **optional** part of the format

### Subject
The `subject` contains a succinct description of the change.
* Is a **mandatory** part of the format
* Use the imperative, present tense: "change" not "changed" nor "changes"
  * Think of `This commit will <subject>`
* Don't capitalize the first letter
* No dot (.) at the end

### Body
The `body` should include the motivation for the change and contrast this with previous behavior.
* Is an **optional** part of the format
* Use the imperative, present tense: "change" not "changed" nor "changes"
* This is the place to mention issue identifiers and their relations

### Footer
The `footer` should contain any information about **Breaking Changes** and is also the place to **reference Issues** that this commit refers to.
* Is an **optional** part of the format
* **optionally** reference an issue by its id.
* **Breaking Changes** should start with the word `BREAKING CHANGES:` followed by space or two newlines. The rest of the commit message is then used for this.


### Examples
* ```
  feat(shopping cart): add the amazing button
  ```
* ```
  feat: remove ticket list endpoint
  
  refers to JIRA-1337
  BREAKING CHANGES: ticket enpoints no longer supports list all entites.
  ```
* ```
  fix: add missing parameter to service call
  
  The error occurred because of <reasons>.
  ```
* ```
  build(release): bump version to 1.0.0
  ```
* ```
  build: update dependencies
  ```
* ```
  refactor: implement calculation method as recursion
  ```
* ```
  style: remove empty line
  ```
  
  
## Git Hook Scripts to ensure commit message header format

### commit-msg Hook (local)
* create following file in your local repository folder`.git-hooks/commit-msg`
  ```shell
  #!/usr/bin/env sh
  
  commit_message="$1"
  # exit with a non zero exit code incase of an invalid commit message
  
  # use git-conventional-commits, see https://github.com/qoomon/git-conventional-commits
  git-conventional-commits commit-msg-hook "$commit_message"
  
  # or verify $commit_message with your own tooling
  # ...
  
  ```
* ⚠ make `.git-hooks/commit-msg` executable (unix: `chmod +x '.git-hooks/commit-msg'`)
* set git hook directory to `.githooks`  `git config core.hooksPath '.git-hooks'`
* commit `.git-hooks` directory if you want to share them with your team, they only need to call the git config command once after cloning the repository

### pre-receive Hook (server side)
* create following file in your repository folder `.git/hooks/pre-receive`
  ```shell
  #!/usr/bin/env sh

  # Pre-receive hook that will block commits with messges that do not follow regex rule

  commit_msg_type_regex='feat|fix|refactor|style|test|docs|build'
  commit_msg_scope_regex='.{1,20}'
  commit_msg_subject_regex='.{1,100}'
  commit_msg_regex="^(${commit_msg_type_regex})(\(${commit_msg_scope_regex}\))?: (${commit_msg_subject_regex})\$"
  merge_msg_regex="^Merge branch '.+'\$"

  zero_commit="0000000000000000000000000000000000000000"

  # Do not traverse over commits that are already in the repository
  excludeExisting="--not --all"

  error=""
  while read oldrev newrev refname; do
    # branch or tag get deleted
    if [ "$newrev" = "$zero_commit" ]; then
      continue
    fi

    # Check for new branch or tag
    if [ "$oldrev" = "$zero_commit" ]; then
      rev_span=`git rev-list $newrev $excludeExisting`
    else
      rev_span=`git rev-list $oldrev..$newrev $excludeExisting`
    fi

    for commit in $rev_span; do
      commit_msg_header=$(git show -s --format=%s $commit)
      if ! [[ "$commit_msg_header" =~ (${commit_msg_regex})|(${merge_msg_regex}) ]]; then
        echo "$commit" >&2
        echo "ERROR: Invalid commit message format" >&2
        echo "$commit_msg_header" >&2
        error="true"
      fi
    done
  done

  if [ -n "$error" ]; then
    exit 1
  fi
  ```
* ⚠ make `.git/hooks/pre-receive` executable (unix: `chmod +x '.git/hooks/pre-receive'`)

-----
## References
* https://www.conventionalcommits.org/
* https://github.com/angular/angular/blob/master/CONTRIBUTING.md
* http://karma-runner.github.io/1.0/dev/git-commit-msg.html
<br>

* https://github.com/github/platform-samples/tree/master/pre-receive-hooks  
* https://github.community/t5/GitHub-Enterprise-Best-Practices/Using-pre-receive-hooks-in-GitHub-Enterprise/ba-p/13863
