# Project 2 Assignment: Java + Maven CI Pipeline

**Due: [set your deadline]**

Follow the `README.md` to build and run the Java pipeline, then complete the tasks below and submit the evidence listed.

## What to do

1. Get the project running in your own GitHub repo and create the Jenkins pipeline job from SCM (README Steps 1 to 3).
2. Run the pipeline until you get one fully green build across all five stages that produces a Docker image (Step 4).
3. Complete Exercise 1: break a test on purpose, run the pipeline, then fix it.
4. Complete Exercise 2: add a `multiply` method and a JUnit test for it.
5. Complete Exercise 3: cause a compile failure and observe that the pipeline fails at `Build`, not `Test`, then fix it.

## What to submit

1. The link to your GitHub repository.
2. A screenshot of a green pipeline run showing all five stages passing: `Checkout`, `Build`, `Test`, `Package`, `Docker Build`.
3. A screenshot from Exercise 1 showing the `Test` stage failed with `Package` and `Docker Build` skipped, plus a screenshot of the green run after the fix.
4. A screenshot from Exercise 2 showing the pipeline passing with your new `multiply` test in the results.
5. A screenshot from Exercise 3 showing the pipeline failing at the `Build` stage because of the compile error.
6. One or two sentences in your own words: what is the difference between a build that fails at `Build` and one that fails at `Test`?

## Grading (100 points)

- Repo set up and pipeline job configured from SCM: 15
- First fully green five-stage build producing a Docker image: 30
- Exercise 1 evidence (failed `Test`, skipped `Package` and `Docker Build`, then fixed): 20
- Exercise 2 evidence (new `multiply` method and passing test): 20
- Exercise 3 evidence (build fails at `Build`) plus the written explanation: 15

## How to submit

Reply to the assignment email with your repository link and the screenshots attached.
