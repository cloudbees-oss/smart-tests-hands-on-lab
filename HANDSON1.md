# Lab 1. Try Predictive Test Selection (PTS) locally

## Overview

You’ll use the Smart Tests CLI on your own repository to see how PTS re-ranks tests based on a code change.

**Goal (what we’re measuring)**:
* **Ranking quality**, not time saved
* Do “expected impacted tests” move **meaningfully higher** (e.g., into the top ~50%) after a code change?

**What you will do**:
* Create a **test session** & generate an **initial ranked test list**
* Make a small **code change**
* Record a **new build** & generate a **new ranked test list**
* Compare **before** vs. **after** rankings to see which tests moved

>[!TIP]
>**What to expect**
>* You’ll see a **full ranked list** in this lab – that’s normal
>* In production, you’d set an **optimization target** (e.g., “target 50%”) to get a subset. However, today we focus only on **re-ranking behaviour**

## Step 1: Generate an initial ranked test list

### Goal
Create a test session for the baseline build and request an initial ranked list of tests. Test selection and recording of test results are done against a test session.

### Do
```
smart-tests record session 
  --build baseline 
  --test-suite my-test-suite > session.txt

smart-tests subset file 
  --session @session.txt 
  --get-tests-from-guess > subset.txt
```
>[!TIP]
>We don't yet know the right size of the subset to create, so instead for this workshop run we'll select all the tests. Smart Tests still produce tests in the relevance order, so this way we can still see how tests are ranked.

### Verify
The output will look like this:
```
Smart Tests created subset <SUBSET_ID> for build baseline (test session <TEST_SESSION_ID>) in workspace <ORG>/<WORKSPACE>

|           |   Candidates |   Estimated duration (%) |   Estimated duration (min) |
|-----------|--------------|--------------------------|----------------------------|
| Subset    |          120 |                   100.00 |                       0.00 |
| Remainder |            0 |                     0.00 |                       0.00 |
|           |              |                          |                            |
| Total     |          120 |                   100.00 |                       0.00 |

Run `smart-tests inspect subset --subset-id <SUBSET_ID>` to view full subset details
```
>[!TIP]
>As you run and record test results, Smart Tests will learn from the results and improve its selection. Among other things, you will be able to specify the size of the subset you'd like to obtain, for example "give me 10 minutes worth of tests to run".

## Step 2: Make a small code change

### Goal
Now that we have recorded a baseline, let’s make some code change to see if Smart Tests picks relevant tests in later steps.

### Do
Make a small edit and create a new commit. Don’t worry, we won’t merge these changes!
```
vim <UPDATE YOUR APP or TEST CODE>
git commit --all --message test
```

### Verify
You will see something like:
```
1 file changed, 1 insertion (+), 1 deletion (-)
```

## Step 3: Record a new build

### Goal
Record the changed code version as a new build, so that Smart Tests' knows which files changed.

### Do
First, record a new build version called mychange
```
smart-tests record build --build mychange
```
and then start a new test session session 2 and request a subset for it,
```
smart-tests record session 
  --build mychange 
  --test-suite my-test-suite > session2.txt

smart-tests subset file 
  --session @session2.txt 
  --use-case one-commit 
  --get-tests-from-guess > subset2.txt
```

### Verify
If you see a message like this, that means it ran successfully:
```
Smart Tests recorded 2 more commits from repository <YOUR PATH>
Smart Tests recorded build hands-on to workspace <YOUR ORG/WORKSPACE> with commits from 1 repository:

| Name   | Path   | HEAD Commit                              |
|--------|--------|------------------------------------------|
| .      | .      | 3f21bfb3d56148c9dcf9f7e811e146bbc3cbf797 |
```

## Step 4: Generate a new ranked test list

### Goal
Request Smart Tests' for a new ranked test list based on the latest recorded build.

### Do
Start a new test session session 2 and then request a subset for it,
```
smart-tests record session 
  --build mychange 
  --test-suite my-test-suite > session2.txt

smart-tests subset file 
  --session @session2.txt 
  --use-case one-commit 
  --get-tests-from-guess > subset2.txt
```

### Verify
Similar to the verification output in Step 1, you will see another table with details for the new subset.

## Step 5: Compare impact on test rankings

### Goal
Compare the baseline ranking to the post-change ranking and see what moved.

### Do
First, run this command to pull up a comparison list
```
smart-tests compare subsets 
  --subset-id-before <SUBSET_ID_1> 
  --subset-id-after <SUBSET_ID_2>
```
Then, evaluate PTS performance using this checklist:
* **Spot-check tests you expected to be impacted**
  * Search for 3 - 5 tests you believe should be affected by your code change
  * For each one, note:
    * **After subset rank**: did it land reasonably high (e.g. in the top ~50%)?
    * **Rank movement**: did it react positively by moving up in rank compared to “before subset”
* **Sanity-check the top of the list**
  * Look at the top ~10 tests in “after subset”
  * Ask: do these tests look related to the files/areas you changed (by name, folder, module, feature)?

### Verify
The command should display the rank of every test in those two subsets, and highlight the differences in the rank. You should see that the tests relevant to your change bubble up in rank.

---

Once your evaluation is complete, please proceed to Lab 2 by clicking here.



