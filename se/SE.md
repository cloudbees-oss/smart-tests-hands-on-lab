# (CloudBees internal) Workshop guide for SE

> [!TIP]
> Until the successful workshop kata is established, the target audience of this document is assumed to be
> sufficiently familiar with Smart Tests. This document is not yet meant for CloudBees SEs at large yet.

## ASAP: Collect sample test reports from customer
Before the workshop, ask the customer to share **JUnit XML reports** from several of their actual CI test runs.

- Request at least 2, preferably 6 recent runs worth of data (spanning 1–2 weeks)
- Include runs with some test failures** and captured stdout/stderr
- [File LCHIB ticket](https://cloudbees.atlassian.net/wiki/x/GoE_IgE) to ask the product team for data import

This allows the workshop to start with the customer’s own test data visible in the Smart Tests dashboard (Unhealthy tests, Trends and AI-based failure grouping, etc.)

## 1 week prior: Your prep
* Create an organization in [Unify Admin tools](http://admin-tools.cloudbees.io/) if it doesn't exist. Add yourself.
* Invite the workshop participants to the organization
* Go to [customer success storefront](https://cloudbees.atlassian.net/wiki/spaces/LCHCTS/pages/4448921585/Storefront+Launchable+Customer+Success),
  use the workspace flag app to (1) enable PTS v2, and (2) failure fast.
* Fork GitHub [cloudbees-oss/smart-tests-hands-on-lab](http://github.com/cloudbees-oss/smart-tests-hands-on-lab/) to
  [the cloudbees-days org](https://github.com/cloudbees-days)
  with a unique name, e.g. `workshop-20251225`
* Add the workshop participants as collaborators to the forked repository.
* Create a PR from the main branch of the forked repository to `cloudbees-oss/smart-tests-hands-on-lab` main branch.
  (You'll have to create some dummy commit to create a PR.)
* In the description of the PR, add the link to edit page of `.github/workflows/pre-merge.yml` so that the participants
  can easily edit the workflow file without getting lost in the GitHub UI. This link should look like this:
  `https://github.com/cloudbees-days/workshop-20251225/edit/main/.github/workflows/pre-merge.yml`

## Actions from participants
* Provide GitHub user ID and email address
* Follow the invitation link to sign up and obtain access to the workspace
* Have them follow [Lab 0](HANDSON0.md) to ensure they have the prerequisites ready
