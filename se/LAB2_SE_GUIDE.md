# Lab 2: SE Guide - PTS CI Integration

## Purpose & Goals

This lab walks customers through the complete Smart Tests CI integration pattern using a toy project. The goal is to show them the exact workflow steps they'll replicate in their own CI pipelines.

**Key outcome:**
- Customers see the full integration lifecycle
- They understand "observation mode" before going live
- They have a working reference implementation to copy

## Step 1: Open the PR to modify the workflow

### Setup (done by SE before workshop)
- Fork repository to `cloudbees-days` org with unique name (e.g., `workshop-20251225`)
- Create PR from fork to `cloudbees-oss/smart-tests-hands-on-lab`
- Add edit link to PR description for easy access

### Why use GitHub's web editor?
- Faster than local cloning/editing
- Avoids Git configuration issues on customer machines
- Live CI feedback loop is immediate and visible

## Step 2: Install Smart Tests CLI

### What's happening
- Sets up Python 3.13 (CLI requirement)
- Installs Smart Tests CLI via pip
- Verifies authentication and configuration

### Why verify in CI?
- Confirms the `SMART_TESTS_TOKEN` secret is set correctly
- Validates network connectivity to Smart Tests API
- Catches configuration issues before test execution

### Common questions

**Q: Why --user flag in pip install?**
A: GitHub Actions runners don't need sudo, but --user is a safe default that works in more environments.

**Q: Can we use a specific CLI version?**
A: Yes, replace `~=2.0` with exact version like `==2.2.0`. The `~=` means "compatible version" (2.x.x).

### Troubleshooting

**Issue:** `smart-tests verify` fails with authentication error
**Solution:** Check repository secrets:
1. Go to Settings → Secrets → Actions
2. Ensure `SMART_TESTS_TOKEN` exists and is valid
3. Token should be from: Workspace Settings → API Token

**Issue:** Wrong organization/workspace shown
**Solution:** Check `SMART_TESTS_ORGANIZATION` and `SMART_TESTS_WORKSPACE` environment variables in workflow.

## Step 3: Add record build command

### What's happening
- `fetch-depth: 0` clones full git history (not just HEAD)
- `smart-tests record build` sends commit metadata to Smart Tests
- Uses `github.run_id` as unique build identifier

### Why full git history?
Smart Tests needs commit history to:
1. Calculate diffs between builds
2. Analyze historical test-to-code relationships
3. Provide commit-level insights in the UI

**Default behavior:** GitHub Actions uses `fetch-depth: 1` (shallow clone) for speed. We override this.

### Why github.run_id?
- Unique identifier for each workflow run
- Easy to correlate CI logs with Smart Tests UI
- Alternative: use `github.sha` (commit hash) or custom build number

### Common questions

**Q: Does this send our source code to Smart Tests?**
A: No. Only commit metadata (SHA, author, message, timestamp). No file contents. See [data privacy policy](https://docs.cloudbees.com/docs/cloudbees-smart-tests/latest/resources/policies/data-privacy-and-protection).

**Q: What if we have a monorepo with multiple projects?**
A: Use `--source` flag to specify multiple repo paths. See docs for multi-repo builds.

## Step 4: Add subset command (observation mode)

### What's happening
- `smart-tests record session` creates a test session
- `--observation` flag enables "training wheels mode"
- `smart-tests subset maven` generates subset but returns ALL tests (due to observation mode)
- `--target 50%` specifies desired subset size (50% of estimated duration)

### Why observation mode?

**Critical concept to explain:**

Observation mode lets you:
1. ✅ See what Smart Tests WOULD select
2. ✅ Run all tests (no risk)
3. ✅ Record results to build ML model
4. ✅ Evaluate performance before going live

**Without observation mode:**
- Only subset runs → if selection is bad, tests are missed
- Can't evaluate performance safely

**When to remove observation mode:**
- After accumulating enough data (typically 10-20 builds)
- After validating subset quality in the UI
- After stakeholder review/approval

### Why --target 50%?
Example value for this workshop. In production, customers choose based on:
- How much time they want to save
- Risk tolerance (higher % = safer, slower)
- Typical failure patterns

Common targets: 30-50% for fast feedback, 60-80% for cautious rollout.

### Understanding the output

```
| Subset    |            2 |                  36.4706 |                  0.0516667 |
| Remainder |            2 |                  63.5294 |                  0.09      |
```

- **In observation mode:** All 4 tests run (despite "Subset" showing 2)
- **After going live:** Only the 2 tests in "Subset" will run
- **Estimated duration:** Based on historical test execution data

### Common questions

**Q: Why does it show a subset if all tests run?**
A: Observation mode simulates selection without enforcement. This lets you validate quality before going live.

**Q: What's the difference between --test-suite and --session?**
A:
- `--test-suite`: Logical grouping (e.g., "unit-test", "integration-test")
- `--session`: Unique identifier for this specific test run

**Q: Can we have multiple subsets in one build?**
A: Yes! Create separate sessions for different test suites (unit, integration, e2e, etc.).

### Troubleshooting

**Issue:** Subset shows 0 tests or 100% of tests
**Solution:** This is normal initially. Smart Tests needs execution history to optimize. After recording results in Step 6, subsequent builds will show better subsetting.

**Issue:** `smart-tests subset maven` fails
**Solution:** Check that `src/test/java` path is correct for the project structure. Adjust path as needed.

## Step 5: Run tests using subset

### What's happening
Maven Surefire plugin reads test class names from `smart-tests-subset.txt` and runs only those tests.

### Maven Surefire integration
The `-Dsurefire.includesFile` parameter tells Surefire:
- Read test names from specified file
- One test per line (format: `example.MulTest`)
- Skip all other tests

**Alternative approaches:**
- JUnit 5: Use `--select-class-file` with JUnit Platform Console Launcher
- Gradle: Use `--tests-file` or build filter programmatically
- Other runners: See [Smart Tests documentation](https://docs.cloudbees.com/docs/cloudbees-smart-tests/latest/)

### What happens in observation mode?
Even though we use the subset file, ALL tests run because `--observation` flag was used in Step 4. The subset file contains all tests.

### Common questions

**Q: What if subset is empty?**
A: Maven will run zero tests. Smart Tests only returns empty subsets if there's truly no relevant tests (rare) or if there's a configuration issue.

**Q: Can we run remainder tests too?**
A: Yes, use `smart-tests subset maven --output-exclusion-rules` to get the inverse. Useful for parallelization strategies.

## Step 6: Report test results back to Smart Tests

### What's happening
- `smart-tests record tests maven` parses JUnit XML reports
- Sends test results (pass/fail/skip, duration, failure messages) to Smart Tests
- Uses `--session @session.txt` to associate results with the earlier session

### Why if: always()?
**Critical for ML learning:**
- If tests fail, GitHub Actions stops the job by default
- Without `if: always()`, Smart Tests never receives failure data
- ML model needs both passes AND failures to learn

### What data is sent?
From Surefire reports (XML files):
- Test names and outcomes (pass/fail/skip)
- Execution duration
- Failure messages and stack traces
- stdout/stderr logs (if captured)

**No source code is sent.**

### Why is this important?
This is how Smart Tests learns:
1. Which tests are slow/fast
2. Which tests fail frequently
3. Historical test-to-code relationships
4. Test flakiness patterns

**Without this step:** PTS quality stays static and doesn't improve.

### Common questions

**Q: What if tests didn't run due to compilation failure?**
A: Use `if: always()` but the glob pattern `./**/target/surefire-reports` won't match anything. Smart Tests records "no tests executed" for this session. This is fine.

**Q: Can we record partial results if some tests failed?**
A: Yes, that's exactly why `if: always()` is critical. Partial results are valuable.

**Q: Where can we see the results?**
A: Link is printed in the workflow log. Also accessible in Smart Tests UI → Sessions → [specific session].

### Verification

**In the workflow log:**
Look for successful upload message with session link.

**In Smart Tests UI:**
1. Follow link from log (or go to Workspace → Sessions)
2. See test results, duration, pass/fail status
3. See observation mode report showing subset performance

### Observation mode report

**What to review with customer:**
- **Subset accuracy:** Did high-ranked tests include actual failures (if any)?
- **Time savings:** What % of time would have been saved?
- **Safety margin:** How many tests would have been skipped?

**Key insight:** This report shows what WOULD HAVE HAPPENED if not in observation mode.

## Step 7: Go live

### What's happening
Remove `--observation` flag to enable actual subsetting. Now only selected tests run.

### When to do this
**In this workshop:** Do it immediately (for demonstration).

**In production:** Only after:
1. Accumulating 10-20 builds with observation mode
2. Reviewing observation reports for quality
3. Getting stakeholder approval
4. Communicating with team about the change

### What changes
- `smart-tests-subset.txt` now contains only selected tests (not all tests)
- CI runs faster (only subset executes)
- Some tests may be skipped
- Remainder tests can run separately (e.g., nightly) or in parallel

### Risk mitigation strategies

**For customers concerned about going live:**

1. **Start conservative:** Use `--target 80%` (only skip 20%)
2. **Staged rollout:** Enable on feature branches only, not main
3. **Parallel execution:** Run subset + remainder in parallel jobs
4. **Monitoring:** Watch for unexpected failures, adjust target as needed
5. **Escape hatch:** Keep a "full test" workflow trigger available

### Common questions

**Q: What if we miss a critical test?**
A:
- Smart Tests uses ML trained on historical data
- Selection improves over time
- Use higher `--target` percentage if risk-averse
- Consider running remainder tests nightly
- Monitor initial rollout closely

**Q: Can we go back to observation mode?**
A: Yes, just add `--observation` flag back. Useful if you want to re-evaluate after significant codebase changes.

**Q: What about flaky tests?**
A: Smart Tests has built-in flaky test detection. Flaky tests are often ranked higher to catch regressions. See "Unhealthy Tests" in UI.

## Integration patterns

### Pattern 1: Subset for PRs, full for main
```yaml
- name: Smart Tests subset
  run: |
    smart-tests record session --build ${{ github.run_id }} --test-suite unit-test > session.txt
    smart-tests subset maven --session @session.txt --target 50% src/test/java > smart-tests-subset.txt
  if: github.event_name == 'pull_request'

- name: Test
  run: mvn test -Dsurefire.includesFile=smart-tests-subset.txt
  if: github.event_name == 'pull_request'

- name: Test (full)
  run: mvn test
  if: github.ref == 'refs/heads/main'
```

### Pattern 2: Parallel subset + remainder
```yaml
subset-tests:
  steps:
    - name: Run subset
      run: mvn test -Dsurefire.includesFile=smart-tests-subset.txt

remainder-tests:
  steps:
    - name: Run remainder
      run: mvn test -Dsurefire.excludesFile=smart-tests-subset.txt
```

### Pattern 3: Conditional subsetting
```yaml
- name: Smart Tests subset
  run: |
    smart-tests record session --build ${{ github.run_id }} --test-suite unit-test > session.txt
    smart-tests subset maven --session @session.txt --confidence 75% src/test/java > smart-tests-subset.txt
```

(Use `--confidence` instead of `--target` to optimize for accuracy rather than speed)

## Troubleshooting

**Issue:** Tests run but no results recorded
**Solution:** Check glob pattern `./**/target/surefire-reports` matches actual report location. Use `find . -name "*.xml"` to locate reports.

**Issue:** Subset file is empty
**Solution:**
- Check if build was recorded successfully (Step 3)
- Verify test paths in subset command match project structure
- Review session in UI to see if Smart Tests detected tests

**Issue:** All tests run even without --observation
**Solution:** Check that subset command isn't failing silently. Verify `smart-tests-subset.txt` actually contains subset (not all tests).

## Next steps for customers

After completing Lab 2:
1. Explore Smart Tests UI features (Trends, Unhealthy Tests, Failure Triage)
2. Review observation reports for their data
3. Plan rollout strategy for their production CI
4. Set up integrations for their test framework (if not Maven)

## Additional resources
- [Smart Tests documentation](https://docs.cloudbees.com/docs/cloudbees-smart-tests/latest/)
- [CI integration guides](https://docs.cloudbees.com/docs/cloudbees-smart-tests/latest/integrations/)
- [Observation mode best practices](https://docs.cloudbees.com/docs/cloudbees-smart-tests/latest/features/predictive-test-selection/observe-subset-behavior)
