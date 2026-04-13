# Lab 1: SE Guide - Predictive Test Selection (PTS) Local Evaluation

## Purpose & Goals

This lab demonstrates PTS re-ranking behavior to help customers evaluate whether Smart Tests can effectively identify relevant tests for their codebase.

**What we're measuring:**
- **Ranking quality**, NOT time saved
- Do "expected impacted tests" move **meaningfully higher** (e.g., into the top ~50%) after a code change?

**Key concept to explain:**
- In this lab, customers see a **full ranked list** - that's intentional
- In production, they'd set an **optimization target** (e.g., "target 50%") to get a subset
- Today we focus only on **re-ranking behavior** to validate PTS effectiveness

## Step 1: Generate an initial ranked test list

### What's happening
- Creates a **baseline test session** associated with the baseline build
- Requests an initial ranked list using `--get-tests-from-guess` (returns all tests, but ranked)

### Why `--get-tests-from-guess`?
We don't yet have execution history, so we can't calculate an optimization target. This flag tells Smart Tests to return all tests in relevance order so we can observe ranking behavior.

### Common questions
**Q: Why are all tests selected (100%)?**
A: This is expected. We're using `--get-tests-from-guess` to see the full ranking. In production with execution history, you'd specify `--target 50%` or similar.

**Q: Why is duration 0.00?**
A: AI doesn't have test duration data yet to analyze. After recording actual test results, it will show estimated durations based on AI analysis of historical data.

## Step 2: Make a small code change

### What's happening
Customer makes a targeted code change so they can evaluate if Smart Tests detects the impact.

### SE guidance
- Suggest they modify a well-isolated file (e.g., a utility function or specific module)
- This makes evaluation clearer - they'll know which tests SHOULD be affected
- Remind them: this is temporary, won't be merged

### Critical: Identify expected tests upfront
Before they make the code change, have the customer identify 3-5 specific tests they expect PTS should pick up based on the file they plan to modify. They should write these test names down - this creates concrete expectations they'll validate in Step 5.

## Step 3: Record a new build

### What's happening
Records the new code state with a different build name (`mychange`) so Smart Tests can:
1. Calculate the diff from baseline
2. Use that diff to inform test selection

### Key concept
A "build" in Smart Tests = a snapshot of the code state (commit SHAs from all repos involved)

## Step 4: Generate a new ranked test list

### What's happening
- Creates a new test session for the `mychange` build
- Uses `--use-case one-commit` to optimize for "what tests should run for this single commit?"
- Generates a new ranked list informed by the code change

### Why `--use-case one-commit`?
Tells Smart Tests' AI to optimize for "short-lived branch" scenario (vs. long-running feature branches or main branch testing).

## Step 5: Compare impact on test rankings

### What's happening
Shows side-by-side comparison of test rankings before and after the code change.

### How to evaluate PTS performance

Guide customers through this checklist:

**1. Spot-check expected impacted tests (3-5 tests)**
- Reference the 3-5 tests they identified in Step 2
- For each test, check:
  - **After subset rank**: Did it land reasonably high (e.g., top ~50%)?
  - **Rank movement**: Did it move UP compared to baseline?

**2. Sanity-check the top of the list (~10 tests)**
- Look at the highest-ranked tests in "after subset"
- Ask: "Do these tests look related to the files/areas you changed?"
- Check by: test name, folder structure, module, feature area

### Success criteria
- Impacted tests should show meaningful upward movement
- Top-ranked tests should be plausibly related to the change
- If both are true → PTS is working well for their codebase

### What if results are poor?
- Check: Was the code change too broad or cross-cutting?
- Check: Are test names/structure representative of what they test?
- Note: With zero execution history, AI has limited data to analyze - AI analysis improves with more historical data
- Consider: Schedule follow-up after Lab 2 when execution data exists

## Troubleshooting

**Issue:** Customer can't find subset IDs for comparison
**Solution:** Subset ID is in the output table. Also available via: `smart-tests inspect subset --subset-id <ID>` or in the web UI.

**Issue:** All tests rank exactly the same
**Solution:** Likely the code change wasn't detected. Check:
- Did they commit the change?
- Did they run `smart-tests record build --build mychange` AFTER the commit?
- Is the file they changed actually in the repo (not .gitignore'd)?

**Issue:** Rankings seem random
**Solution:** With zero history, AI has limited data to analyze. Emphasize that AI analysis improves after Lab 2 when we record actual test results.
