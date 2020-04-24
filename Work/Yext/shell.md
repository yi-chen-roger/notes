## Yext git Command

git (new-branch | nb)

    Creates a new branch from the latest master (step 1 of the below workflows).

git (upload | up) [options...]

    Upload one or more commits for review

git (pull-master | pm):

    Fetch and update the master branch to the latest revision (second-last step of the below workflows). If master is checked out, it will also rebase any unmerged commits onto the tip of origin/master.

git (housekeeping | hk):

    Delete stale branches (i.e. those whose commits have all been shipped and backup refs created by the upload script. If a stale branch is checked out, or is the master br


## Yext edward
 edward start devlab

 http://localhost:9068/devlab 




 oh-my-z shell 在很大的repo下太慢
https://stackoverflow.com/questions/12765344/oh-my-zsh-slow-but-only-for-certain-git-repo
You can add this to your git config and zsh won't check the status anymore

git config --add oh-my-zsh.hide-status 1
git config --add oh-my-zsh.hide-dirty 1