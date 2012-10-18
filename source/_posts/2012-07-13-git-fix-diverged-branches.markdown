---
layout: post
title: "Git Fix Diverged Branches"
date: 2012-07-13 12:00
comments: true
categories: 
- git
---

## Situation

You are working on a branch and wanted to merge it back into master. However, something went wrong and now you've got a master that is a certain number of commits ahead of where you branched off and still no succesful merge.

So the current situation looks like

    A-B-C-F-G origin/master
         \D-E otherBranch
         
while you want

    A-B-C-D-E origin/master
              otherbranch

## Fixing the situation

### First: The master branch

You need to get the hashcode of the commit that is the branching point (commit C).

Make sure you are in the master branch and then do:

    git reset --hard hashcodeOfBranchingPoint
    
* _git reset_ will roll back the current branch to commit that was provided
* Using reset will also make the changes in the filesystem instead of only in git (if not, git would see the difference as unchanged files, I think)

After you've done this, you repository will look like this

    A-B-C     master
         \D-E otherBranch
         \F-G origin
So you've basically thrown away any commits made to master after the branching point you've resetted to. However, origin is still where it used to be.

### Second: The remote's master branch

You will probably want the remote to reflect all the local changes, and to not get in trouble later, it's best to reset it as well.

    git push --force origin
    
* push --force will make the remote look identical to the local copy of the reposity (be careful using this, as it will overwrite any commits that happened after the branching point on the remote)

This will cause the repository to look like so

    A-B-C     origin/master
         \D-E otherBranch

### Third: The merge

To finally merge with the other branch, do

    git merge otherBranch
    
and again to update the remote

    git push origin
    
making the repository look like

    A-B-C-D-E origin/master
              otherBranch
    
And everything should be fine now.



