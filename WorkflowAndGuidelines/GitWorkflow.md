# Git workflow

If you're a developer who wants to work on the MuseScore source code and submit your changes to be merged into the main code, here's how.

## Git references

* [Got 15 minutes and want to learn Git?](http://try.github.com)
* [Fork a repo at GitHub](https://help.github.com/articles/fork-a-repo) - most of the text on this page is taken from here.
* [Git documentation](http://git-scm.com/documentation) - includes videos.
* [Git Ready](http://gitready.com) - learn Git one commit at a time.
* [Pro Git book](http://git-scm.com/book) - free online and can be bought in paper form.
* [Git Cheat Sheet](http://cheat.errtheblog.com/s/git) - from setup to advanced command in a condensed form, colour your git diff, status, etc...
* [How to Write a (great) Git Commit Message](https://chris.beams.io/posts/git-commit) - as trivial as it may sound, those [7 rules](https://chris.beams.io/posts/git-commit/#seven-rules) help to get the most out of the commit history.

## Suggested workflow

If you don't have an account on GitHub, [create one for free](https://github.com/signup/free) first. Also, make sure you [set up git](https://help.github.com/articles/set-up-git) on your computer. It's recommended to use [SSH to access your git fork](https://help.github.com/articles/generating-ssh-keys). This workflow is a command-line workflow. If you prefer using a UI, GitHub also provides [a UI tool](https://desktop.github.com) for Mac and Windows that can automate some of the following operations.

### Summary

1. Fork on GitHub (click Fork button)
2. Clone to computer, use SSH URL: `git clone git@github.com:you/MuseScore.git`
3. Don't forget to cd into your repo: `cd MuseScore/`
4. (optional but recommended) In your clone directory, copy the file `build/git/hooks/post-checkout` to the directory `.git/hooks` in order for `mscore/revision.h` to be maintained automatically by git. See `build/git/hooks/README` for details. Note that the `.git` directory is hidden.
5. Set up remote upstream: `git remote add musescore git://github.com/musescore/MuseScore.git`
6. Create a branch for new issue: `git checkout -b 1234_new_feature`
7. Develop on issue branch. [Time passes, the main MuseScore repository accumulates new commits]
8. Commit changes to your local issue branch: `git add . ; git commit -m 'commit message'`
9. Fetch (download) upstream's master branch: `git fetch musescore master`
10. Update local master to match upstream's master: `git checkout master; git rebase musescore/master`
11. Rebase issue branch: `git checkout 1234_new_feature; git rebase master`
12. Repeat steps 7-11 until dev is complete
13. Push branch to GitHub: `git push origin 1234_new_feature`
14. Start your browser, go to your GitHub repo, switch to "1234_new_feature" branch and press the [Pull Request] button

After having made a Pull Request don't pull/merge anymore, it'll mess up the commit history. If you (have to) rebase, use `git push --force` to send it up to your GitHub repository, this will update the PR too.  
Be careful not to do this while the core team is working on merging in your PR.
