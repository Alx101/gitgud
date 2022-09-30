# GitGud

A small repo with branches and stuff to test some git commands.

The following will be tried out in this repo:

- `rebase`
- `cherry-pick`
- `rebase-onto`
- fixup commits

## The school

### Initial state

The project currently looks like this

- ec7a41d (origin/dev-branch, dev-branch) More info
- cd11536 merged other changes
  | _ 2646179 (origin/feature/more-info, feature/more-info) more data here
  | _ 7eeec91 (origin/feature/data, feature/data) fixup! some data added
  | \* 95f7494 some data added
  |/
- 22e33eb update dev branch
- 5bb69ea (HEAD -> master, origin/master) more info added
- 157f69f add info
- c77e500 initial commit

We have 4 branches:

- `master` the one you're on now
- `dev-branch` a dev branch, currenly one commit behind and two commits ahead of `master`
- `feature/data` a feature branch, two commits ahead and one commit behind `dev-branch`
- `feature/more-info` another feature branch, branching off of `feature/data` and is one commit ahead

#### Rebasing

Let's go into `feature/data`: `git checkout feature/data`
So say we are finished with `feature/data` and want to merge it into `dev-branch`. We can't currently, because it's one commit behind. We have two options here:

1. Merge the changes fron `dev-branch` into `feature/data`, this will create a merge commit that will try and combine the two branches into one
2. Rebase `feature/data` onto `dev-branch`, basically putting all the commits from `feature/data` on TOP of `dev-branch`

The first option is pretty easy and straightforward, but it makes for an ugly commit history. We always want our feature branches to just include the changes they make, as if they were based off of the dev branch. Therefore we go with option #2

`git rebase dev-branch`

Running `git log` now will show us that the missing commit from `dev-branch` has been added to `feature/data`. We're free to merge!

`git checkout dev-branch`
`git merge feature/data`

#### Fixup commits

Shoot, looks like we made a small error in the data, we forgot some info. We can fix it directly on the dev-branch, as we're the only developers on it:

```
echo "forgot some data" >> data.txt
git add .
```

Normally, we'd just make a normal commit with a message (`git commit -m "some message"`), but this was a very small change and doesn't really require a while new commit. We could commit, then squash, but let's use something simpler: a fixup commit

`git commit --fixup HEAD`

Here we're making a commit, labelling it as a `fixup` commit, and specifying which commit we're fixing up. It can be any commit hash, but we're using `HEAD` (meaning the latest commit), because that's the one we're fixing. This will create a commit with the same
message as the commit you're fixing up (the one you specificed), and adding `fixup! ...` to it. You might think it looks ugly in the git history, and you'd be right, but this is where `--autosquash` comes in.

Our dev branch is currenlty one commit behind master, we need it to be on the same update level as master. Let's rebase, but we specify two extra parameters: `git rebase -i --autosquash master`.
This opens up a list of your current commits, here we can decide to _r_ (reword) or _s_ (squash) a commit, or even omit it entirely. Let's just move on `esc :wq`.

Now if you run `git log` you'll see that the fixup commit is gone! This is because `--autosquash` along with interactive rebase (`-i`) will automatically squash any commits.

#### Rebase --onto

Okay, back to the other feature branch. It's time we get that up to speed as well. It's only got one differing commit though, but currently it has a completely different history to `dev-branch`.

We could just rebase with `dev-branch`, but it's a bit of a hassle with all of the potential merge conflicts, and it'd be easier if we could just plop that last commit directly onto `dev-branch`, right?

Well, you're in luck, this is where `--onto` comes into play. Let's first get some information to use: `git log`. Copy the commit hash of the topmost commit, we're going to need it for later so save it somewhere.

Now let's jump over to the feature-branch: `git checkout feature/more-data` and run `git log` here too. We can see our latest commit `2646179` and then the fixup commit `7eeec91` right after. Copy the has for the fixup commit, this is part two of our rebase.

Time to rebase, run `git rebase --onto <NEW PARENT> <OLD PARENT>` and replace `<NEW PARENT>` with the hash we copied from `dev-branch`, and replace `<OLD PARENT>` with the fixup commit hash.
Donezo!

So, what happened here? Well, we told git to rebase everything that came AFTER `<OLD PARENT>` (in this case, our single commit), and put that on `<NEW PARENT>`. We basically moved the commit from one branch to another. In this very case, a cherry-pick could've done as well, but imagine we had 15 more commits, that'd be a hassle to cherry-pick right?

#### Cherry picking

Ok, let's go over cherry-picking as well. It's very simple, so I'll just give a short brief.

Check out master: `git checkout master`

And create a new branch `git checkout -b new-branch`. Ok, so say we wanted that fixup commit for whatever reason. We can fetch that in by just saying `git cherry-pick <HASH>`, in this case it's `7eeec91` that is the hash: `git cherry-pick 7eeec91`. Running `git log` you'll see we got that commit into the history now.

#### Wrapup

Any questions?
