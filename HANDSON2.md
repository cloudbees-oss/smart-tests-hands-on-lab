# Lab 2. Try predictive test selection

In this section, you will test drive Predictive Test Selection (PTS) on your own computer.
Along the way, you will learn the major concepts of Smart Test.

## Clone your repository
To experiment with Smart Tests, first, let's clone your repository locally.
How to do this depends on your project.

If you already cloned repository, please skip cloning the repository and move to the repo directory.

```
git clone ...
cd your/repository
```

## Capture software under test

In order to select the right tests for your software, Smart Tests need to know what software you are testing. We call this a **build**.

A build is a specific version of your software that you are testing. It can consist of multiple Git repositories, and in each repository, it points to a specific commit. A build is identified by its name.

> **build** represents the software. Each time you send test results to Smart Test, you record them against a specific build so that Smart Tests know that you ran X tests against Y software with Z results.

refs: [Documentation](https://www.launchableinc.com/docs/concepts/build/)

Therefore, before you run your tests, you record a build using `launchable record build`.

Move to the locally checked out copy of your software, check out its main branch,
and run the following command to record a build:
```
launchable record build --name mychange1
```
If you see a message like this, it was successful:

```
Launchable transferred 2 more commits from repository <YOUR PATH>
Launchable recorded build hands-on to workspace <YOUR ORG/WORKSPACE> with commits from 1 repository:

| Name   | Path   | HEAD Commit                              |
|--------|--------|------------------------------------------|
| .      | .      | 3f21bfb3d56148c9dcf9f7e811e146bbc3cbf797 |

Visit https://app.launchableinc.com/organizations/<ORG>/workspaces/<WORKSPACE>/data/builds/<BUILD ID> to view this build and its test sessions
```

What just happened? Smart Tests recorded the current HEAD of your local repository as the build,
using the name given.

>[!NOTE]
> If you have multiple repositories, please check [this document](https://www.launchableinc.com/docs/sending-data-to-launchable/using-the-launchable-cli/recording-builds-with-the-launchable-cli/recording-builds-from-multiple-repositories/) to let Smart Tests know that the build consists of your repositories.


Since this was the first time you recorded a build, Smart Tests needed to transfer relatively
large amount of data to its server, including recent commit history, file contents, etc. It
also has to do a lot of number crunching to prepare for the predictive test selection.

But subsequent calls to `launchable record build` will be much faster, because Smart Tests will only transfer the new commits that you have added since the last build.

## Request and inspect a subset to test
Now, you declare the start of a new test session; A test session is an act of running tests against a specific build. Test selection and recording of test results are done against a test session.

 refs: [Documentation](https://www.launchableinc.com/docs/concepts/test-session/)

 ```
 launchable record session --build mychange1 > session.txt
 ```

When you record a new test session, Smart Tests will return a session ID, which is stored in `session.txt` file.

Now, let's have Smart Tests select the best set of tests to run for this test session.

 ```
 launchable subset --use-case one-commit --session $(cat session.txt) --get-tests-from-guess file > subset.txt
 less subset.txt
```

Since you haven't run any tests yet, Smart Tests will select files in your repository
that looks like tests (`--get-tests-from-guess`), and order them such that tests relevant to most recent change (`--use-case one-commit`) are prioritized.

> [!TIP]
> We'll use this baseline `subset.txt` later to see how additional changes affect the test selection,
> so please keep this file around.


As you run and record test results, Smart Tests will learn from the results and improve its selection.
Among other things, you will be able to specify the size of the subset you'd like to obtain, for example
"give me 10 minutes worth of tests to run".



## Make a change you want to test, and see how that affects the subset
When a developer makes a change to the code, we want Smart Tests to select the tests that
are most relevant to that change. To experiment with this, let's make a small change.
Don't worry, the commit you'll create will stay in your computer.

```
vim <UPDATE YOUR APP or TEST CODE>
git commit --all --message test
```

We now have a new software version to test, so we need to record it as a new build:

```
launchable record build --name mychange2
```

Create a new test session against the new build and request a subset again:

```
launchable record session --build mychange2 > session2.txt
launchable subset --use-case one-commit --session $(cat session2.txt) --get-tests-from-guess file > subset2.txt
```

Compare the results between the first and the second subsets:

```
launchable compare subsets subset.txt subset2.txt
```

The command should display the rank of every test in those two subsets, and highlight the differences in the rank.
You should see that the tests relevant to your change bubble up in the rank.


You can now move on to [the next step](HANDSON3.md).
