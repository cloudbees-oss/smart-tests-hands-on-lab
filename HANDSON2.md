# Lab 2: PTS integration into a mock CI workflow

## Overview

**Goal**: Gain a better understanding of how to integrate & setup Predictive Test Selection into your CI workflow. We will use a toy Java project in this repository and update its GitHub Actions workflow as an example to do this.

**What you will do**:
* Install Smart Tests CLI & Verify connection
* Add build recording information
* Add subset command
* Update test run job to pick up subset instead of full test list
* Add test result reporting command
* Remove `--observation` flag to “go live” from next run

## Step 1: Open the PR to modify the workflow

Your instructor should have created a pull request to cloudbees-oss/smart-tests-hands-on-lab and shared it with you. Use this PR to modify the workflow sequentially through the lab.

* In the PR description, click **edit CI script** to open the workflow in GitHub’s editor
* Confirm you are editing: `.github/workflows/pre-merge.yml`

## Step 2: Install Smart Tests CLI & Verify connection

### Goal

Install Smart Tests CLI in the workflow and confirm CI can authenticate to Smart Tests.

### Do

First of all, update `.github/workflows/pre-merge.yml` as follows:

```diff
        with:
          java-version: 21
          distribution: "adopt"
+     - uses: actions/setup-python@v5
+       with:
+         python-version: '3.13'
+     - name: Install Smart Tests CLI
+       run: pip install --user --upgrade smart-tests-cli~=2.0
      - name: Compile
        run: mvn compile
```
<details>
<summary>Raw text for copying here</summary>

```
- uses: actions/setup-python@v5
  with:
    python-version: '3.13'
- name: Install Smart Tests CLI
  run: pip install --user --upgrade smart-tests-cli~=2.0
```

</details>
<br>

Next, we will add the `smart-tests verify` command to the workflow as well.

Update `.github/workflows/pre-merge.yml` as follows:
```diff
       - name: Install Smart Tests CLI
         run: pip install --user --upgrade smart-tests-cli~=2.0
+      - name: Smart Tests verify
+        run: smart-tests verify
       - name: Compile
         run: mvn compile
       - name: Test
```

<details>
<summary>Raw texts for copying</summary>

```
- name: Smart Tests verify
  run: smart-tests verify
```

</details>
<br>

Finally, add these changes to the PR by clicking on **Commit changes**.

<img width="407" height="151" alt="image" src="https://github.com/user-attachments/assets/7687ca9e-2c30-4c2b-bf0c-c093d65c7f21" />

(Commit directly to the branch, as opposed to create a new branch)

When the commit goes through, come back to the PR page to see the newly added commit. Depending on the status of the CI run, you will see one of the icons on the left of the commit hash:

<img width="672" height="83" alt="image" src="https://github.com/user-attachments/assets/afb51dd1-353f-4b6d-b453-e47c37f7184e" />

* 🟡 if a CI run is ongoing
* ❌ if a CI run has failed
* ✔️ if a CI run has completed successfully
* None if a CI run hasn't started, or if there was a syntax error in the edit you just made.

> [!TIP]
> You might have to reload the page to see the commit and its status updated.

Using the following screenshot as an example, click the status symbol, and select "details" to jump to the CI log.

<img width="723" height="169" alt="image" src="https://github.com/user-attachments/assets/997dd7ef-87a7-4d8d-b917-b37d5b46895a" />

### Verify

If everything goes as expected, in the "Smart Tests verify" section, you should see a message like this:

```
Organization: launchable-demo
Workspace: hands-on-lab
Proxy: None
Platform: 'Linux-6.8.0-1017-azure-x86_64-with-glibc2.39'
Python version: '3.13.0'
Java command: 'java'
smart-tests version: '2.2.0'
Your CLI configuration is successfully verified 🎉
```

## Step 3: Add record build command

### Goal
Ensure Smart Tests can see commit history and record a build for each CI run.

### Do

First, enable full git history. 

Update `.github/workflows/pre-merge.yml` as follows:
```diff
steps:
       - uses: actions/checkout@v5
+        with:
+          fetch-depth: 0
       - uses: actions/setup-java@v4
         with:
           java-version: 11
```

<details>
<summary>Raw text for copying</summary>

```
with:
  fetch-depth: 0
```

</details>
<br>

Then, add a command for recording build.

Update `.github/workflows/pre-merge.yml` as follows:
```diff
run: pip install --user --upgrade smart-tests-cli~=2.0
       - name: Smart Tests verify
         run: smart-tests verify
+      - name: Smart Tests record build
+        run: smart-tests record build --build ${{ github.run_id }}
       - name: Compile
         run: mvn compile
   worker-node-1:
```

<details>
<summary>Raw text for copying</summary>

```
- name: Smart Tests record build
  run: smart-tests record build --build ${{ github.run_id }}
```

</details>
<br>

### Verify

Commit changes directly to your branch by clicking on **Commit changes**. If the setup is successful, you will see logs similar to the following:

```
Launchable recorded 1 commit from repository /home/runner/work/hands-on/hands-on
Launchable recorded build 3096604891 to workspace organization/workspace with commits from 1 repository:

| Name   | Path   | HEAD Commit                              |
|--------|--------|------------------------------------------|
| .      | .      | 5ea0a739271071dfbdacd330b0cc28c307151a04 |
```

## Step 4: Add subset command (observation mode)

### Goal

Generate a test subset for this CI build and inspect what Smart Tests would select—without fully “going live” yet.

Observation mode is the ["the training wheel mode"](https://docs.cloudbees.com/docs/cloudbees-smart-tests/latest/features/predictive-test-selection/observe-subset-behavior). With this flag, Smart Test will go through all the motions, except for actually returning all the tests. We'll use this mode to observe the behavior/performance of the test selection, hence the name.

### Do

Update `.github/workflows/pre-merge.yml` as follows:
```diff
      - name: Compile
        run: mvn compile
+     - name: Smart Tests subset
+       run: |
+         smart-tests record session --build ${{ github.run_id }} --observation --test-suite unit-test > session.txt
+         smart-tests subset maven --session @session.txt --target 50% src/test/java > smart-tests-subset.txt
+         cat smart-tests-subset.txt
      - name: Test
        run: mvn test
```
<details>
<summary>Raw text for copying</summary>

```
- name: Smart Tests subset
  run: |
    smart-tests record session --build ${{ github.run_id }} --observation --test-suite unit-test > session.txt
    smart-tests subset maven --session @session.txt --target 50% src/test/java > smart-tests-subset.txt
    cat smart-tests-subset.txt
```

</details>
<br>

Commit changes directly to your branch by clicking on **Commit changes**.

### Verify

If the setup is successful, you will see logs similar to the following, although the details might vary:

```
|           |   Candidates |   Estimated duration (%) |   Estimated duration (min) |
|-----------|--------------|--------------------------|----------------------------|
| Subset    |            2 |                  36.4706 |                  0.0516667 |
| Remainder |            2 |                  63.5294 |                  0.09      |
|           |              |                          |                            |
| Total     |            4 |                 100      |                  0.141667  |

Run `launchable inspect subset --subset-id XXX` to view full subset details
example.MulTest
example.DivTest
example.AddTest
example.SubTest
```

## Step 5: Run tests using subset

### Goal
Run only the selected tests instead of the full suite.

### Do
Pass this subset to the test runner.

```diff

      - name: Test
-       run: mvn test
+       run: mvn test -Dsurefire.includesFile=smart-tests-subset.txt
```
<details>
<summary>Raw text for copying</summary>

```
run: mvn test -Dsurefire.includesFile=smart-tests-subset.txt
```

</details>
<br>

Commit changes directly to your branch by clicking on **Commit changes**.

### Verify

In the Actions log, Maven runs with: `-Dsurefire.includesFile=smart-tests-subset.txt`

## Step 6: Report test results back to Smart Tests

### Goal

Record test results so Smart Tests can learn and you can view results in the web app.

> [!TIP]
> Important: if tests fail, GitHub Actions can stop the job early—so we use `if: always()` to ensure results are still reported.

### Do

Update `.github/workflows/pre-merge.yml` as follows:
```diff
      - name: Test
        run: mvn test -Dsurefire.includesFile=smart-tests-subset.txt
+     - name: Smart Tests record tests
+       if: always()
+       run: smart-tests record tests maven --session @session.txt ./**/target/surefire-reports
```
<details>
<summary>Raw text for copying</summary>

```
- name: Smart Tests record tests
  if: always()
  run: smart-tests record tests maven --session @session.txt ./**/target/surefire-reports
```

</details>
<br>

Commit changes directly to your branch by clicking on **Commit changes**.

### Verify

If everything is set up correctly, you can view the test results on Launchable as shown below: (A URL to this page is in the GitHub Actions log)

<img src="https://github.com/user-attachments/assets/f83dd1e6-bf9e-4091-964c-da665ffd764d" width="50%">

You should also see the report from the subset observation:

![image](https://user-images.githubusercontent.com/536667/195477376-500d318a-b67a-4202-8c90-81ca6048dcc4.png)

## Step 7: Go live

If this was a real project, we'd keep the `--observation` flag until we accumulate enough data, then
evaluate its performance & roll out. In this workshop, we can skip this step and go live right away.

```diff
      - name: Smart Tests subset
        run: |
-        smart-tests record session --build ${{ github.run_id }} --observation --test-suite unit-test > session.txt
+        smart-tests record session --build ${{ github.run_id }} --test-suite unit-test > session.txt
         smart-tests subset maven --session @session.txt --target 50% src/test/java > smart-tests-subset.txt
      - name: Test
        run: mvn test
```

Let's apply this change and check the result.

Commit changes directly to your branch by clicking on **Commit changes**.

---

You have completed the hands-on workshop! 




