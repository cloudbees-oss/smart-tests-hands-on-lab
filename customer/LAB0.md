# Lab 0: Pre-requisites & environment setup

## Overview

**Goal**: In this pre-workshop lab, participants are required to check their system requirements, setup their environment with the Smart Tests CLI, and other tasks so that the actual workshop runs smoothly.

**What you will do**:
* Confirm InfoSec policy requirements internally
* Prepare & share test results data (JUnit XML files)
* Install Smart Tests CLI
* Configure API token & connect to your Unify organization
* Clone your repository & record a baseline build

## Step 1: Confirm internal InfoSec policies

Look into your internal InfoSec policies and make sure you can let CloudBees process your source code during this workshop. Note we do not store your source code. See [data privacy and protection policy](https://docs.cloudbees.com/docs/cloudbees-smart-tests/latest/resources/policies/data-privacy-and-protection) for more details.

## Step 2: Prepare & share test results data (JUnit XML files)

### Goal
To personalize your workshop experience, we load your recent test results into Smart Tests so you can explore insights (like _Unhealthy tests_, _Trends_ and _Failure grouping_) using your own real data. This makes the walkthrough immediately relevant – so you can validate exactly how your data will fit into the information model as well as visualize what use-cases you could achieve.

### Do
* Provide recent test results data from your actual project in the standardized JUnit XML format as described [here](https://github.com/testmoapp/junitxml).
* Provide atleast 6 recent runs worth of test results (ideally spanning 1–2 weeks)
* Include runs with both passing and failing tests
* Make sure stdout and stderr logs are captured in your reports
* If your XML files are in a different format, please contact us

### Verify
Once you share these files with our team, Smart Tests' will extrapolate your test execution history and load that data into a demo workspace. You will be presented a live walkthrough of how your test data looks and feels in the dashboard.

## Step 3: Install Smart Tests CLI

### Goal
Install the smart-tests command line tool.

### Do
Install using [`uv`](https://docs.astral.sh/uv/): 
```
uv tool install smart-tests-cli~=2.0
```

### Verify
To check that it's installed correctly, run:
```
smart-tests --help
```

## Step 4: Configure API token & connect to your Unify organization

### Goal
Configure your CLI so it can authenticate to Smart Tests in CloudBees Unify.

### Do
* In the Smart Tests web app, go to: Workspace Settings → API Token (as in screenshot below)
* Generate a token and copy it
* Set it as an environment variable:
```
export SMART_TESTS_TOKEN=<API TOKEN>
```
>[!TIP]
>Note: If you haven’t created a workspace yet, please refer to [SIGN_UP.md](SIGN_UP.md) to set one up.

### Verify
Run:
```
smart-tests verify
```
If you see a message like this, you're all set:
```
Organization: 'organization'
Workspace: 'workspace'
Proxy: None
Platform: 'Linux-6.10.14-linuxkit-aarch64-with-glibc2.36'
Python version: '3.11.13'
Java command: 'java'
smart-tests version: '2.2.0'
Your CLI configuration is successfully verified 🎉
```

## Step 5: Clone your repository

### Goal
Clone the repository you want to evaluate Predictive Test Selection with, onto your local machine and then move into it. 

### Do
If your repo is already cloned, skip to the cd command.
```
git clone ...
cd your/repository
```

### Verify
You should be able to see repo files (for example `ls` shows your project contents)..

## Step 6: Record a baseline build

### Goal
Tell Smart Tests “this is the exact code version I’m about to test”. In order to select the right tests for to run against your change, Smart Tests need to know what is the software you are testing and what’s changed. This information is a called a build.

### Do
Check out the main branch, and run the following command to record a build:
```
smart-tests record build --build baseline
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

----

Congratulations! You finished all the Workshop pre-requisites. During the Smart Tests workshop we will start hands-on with Lab 1.

