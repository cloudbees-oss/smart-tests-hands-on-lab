# (CloudBees internal) Workshop guide for SE

> [!TIP]
> Until the successful workshop kata is established, the target audience of this document is assumed to be
> sufficiently familiar with Smart Tests. This document is not yet meant for CloudBees SEs at large yet.

## Collect test data from customer
Before the workshop, ask the customer to share **JUnit XML reports** from several of their actual CI test runs.

- Request atleast 6 recent runs worth of data (spanning 1–2 weeks)
- Include runs with some test failures** and captured stdout/stderr
- Once received, do a sanity check and verify the data
- Then, share the data to engineering team for import and setup

This allows the workshop to start with the customer’s own test data visible in the Smart Tests dashboard (Unhealthy tests, Trends and AI-based failure grouping, etc.)

## 1 week prior: Your prep
* Before the actual workshop, go to [customer success storefront](https://cloudbees.atlassian.net/wiki/spaces/LCHCTS/pages/4448921585/Storefront+Launchable+Customer+Success), create a new workspace, then
  use the workspace flag app to (1) enable PTS, and (2) set the state to "HANDS_ON_LAB_V2"
* Invite the workshop participants to the workspace by creating an invitation link and send it to them
* Hand the URL of this workspace to the participants
* Fork GitHub launchableinc/hands-on-lab to [the cloudbees-days org](https://github.com/cloudbees-days)
  with a unique name, e.g. `hol-20251225`
* Add the workshop participants as collaborators to the forked repository.
* Create a PR from the main branch of the forked repository to `launchableinc/hands-on-lab` main branch.
  (You'll have to create some dummy commit to create a PR.)
* In the description of the PR, add the link to edit page of `.github/workflows/pre-merge.yml` so that the participants
  can easily edit the workflow file without getting lost in the GitHub UI. This link should look like this:
  `https://github.com/cloudbees-days/hol-20251225/edit/main/.github/workflows/pre-merge.yml`

## Actions from participants
* Provide GitHub user ID and email address
* Follow the invitation link to sign up and obtain access to the workspace
* Have them follow [Lab 0](HANDSON0.md) to ensure they have the prerequisites ready
