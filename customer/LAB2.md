# Lab 2: PTS integration into a mock CI workflow

## Overview

You'll integrate Predictive Test Selection into a GitHub Actions workflow using this repository's example Java project.

## Step 1: Open the PR to modify the workflow

Your instructor has created a pull request and shared it with you.

- In the PR description, click **edit CI script** to open the workflow
- Confirm you are editing: `.github/workflows/pre-merge.yml`

## Step 2: Install Smart Tests CLI

Add Python setup and Smart Tests CLI installation.

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.13'
- name: Install Smart Tests CLI
  run: pip install --user --upgrade smart-tests-cli~=2.0
```

Add verification step:

```yaml
- name: Smart Tests verify
  run: smart-tests verify
```

Commit these changes to the PR by clicking **Commit changes**.

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

**Verify:** In the workflow log, you should see:

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

Enable full git history:

```yaml
- uses: actions/checkout@v5
  with:
    fetch-depth: 0
```

Add build recording:

```yaml
- name: Smart Tests record build
  run: smart-tests record build --build ${{ github.run_id }}
```

**Verify:** You should see output like:

```
Smart Tests recorded 1 commit from repository /home/runner/work/hands-on/hands-on
Smart Tests recorded build 3096604891 to workspace organization/workspace with commits from 1 repository:

| Name   | Path   | HEAD Commit                              |
|--------|--------|------------------------------------------|
| .      | .      | 5ea0a739271071dfbdacd330b0cc28c307151a04 |
```

## Step 4: Add subset command (observation mode)

Add the subset generation step:

```yaml
- name: Smart Tests subset
  run: |
    smart-tests record session --build ${{ github.run_id }} --observation --test-suite unit-test > session.txt
    smart-tests subset maven --session @session.txt --target 50% src/test/java > smart-tests-subset.txt
    cat smart-tests-subset.txt
```

**Verify:** You should see output like:

```
|           |   Candidates |   Estimated duration (%) |   Estimated duration (min) |
|-----------|--------------|--------------------------|----------------------------|
| Subset    |            2 |                  36.4706 |                  0.0516667 |
| Remainder |            2 |                  63.5294 |                  0.09      |
|           |              |                          |                            |
| Total     |            4 |                 100      |                  0.141667  |

Run `smart-tests inspect subset --subset-id XXX` to view full subset details
example.MulTest
example.DivTest
example.AddTest
example.SubTest
```

## Step 5: Run tests using subset

Update the test command to use the subset:

```yaml
- name: Test
  run: mvn test -Dsurefire.includesFile=smart-tests-subset.txt
```

**Verify:** Maven runs with the subset file: `-Dsurefire.includesFile=smart-tests-subset.txt`

## Step 6: Report test results back to Smart Tests

Add result reporting:

```yaml
- name: Smart Tests record tests
  if: always()
  run: smart-tests record tests maven --session @session.txt ./**/target/surefire-reports
```

**Verify:**

You can view the test results in Smart Tests as shown below (URL is in the GitHub Actions log):

<img src="https://github.com/user-attachments/assets/f83dd1e6-bf9e-4091-964c-da665ffd764d" width="50%">

You should also see the report from the subset observation:

![image](https://user-images.githubusercontent.com/536667/195477376-500d318a-b67a-4202-8c90-81ca6048dcc4.png)

## Step 7: Go live

Remove the `--observation` flag:

```yaml
- name: Smart Tests subset
  run: |
    smart-tests record session --build ${{ github.run_id }} --test-suite unit-test > session.txt
    smart-tests subset maven --session @session.txt --target 50% src/test/java > smart-tests-subset.txt
```

**Verify:** The workflow now runs with actual test subsetting enabled.

---

**Lab 2 Complete** - You have successfully integrated Smart Tests into your CI pipeline!
