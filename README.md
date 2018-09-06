# wercker-tests

This repository defines a number of wercker pipelines that test various aspects of wercker.

See the [latest runs on app.wercker.com](https://app.wercker.com/wercker/wercker-tests/runs)

See the [latest runs on dev.wercker.com](https://dev.wercker.com/wercker/wercker-tests/runs)

The table below lists the pipelines and how they should be connected in a workflow:

```
build --> test-docker-build1        --> test-docker-build2         Tests internal/docker-build
          test-docker-push-classic1 --> test-docker-push-classic2  Tests internal/docker-push (committing and pushing pipeline container)
          test-docker-scratch-push1 --> test-docker-scratch-push2  Tests internal/docker-scratch-push
          test-non-rdd-artifacts1   --> test-non-rdd-artifacts2    Tests propagation of /pipeline/cache and /pipeline/output to next pipeline when no RDD
          test-rdd-artifacts1       --> test-rdd-artifacts2        Tests propagation of /pipeline/cache and /pipeline/output to next pipeline when using a RDD
          test-rdd1                 --> test-rdd2                  Tests using a RDD to perform native docker build, run, kill and push
          test-fn1                                                 Tests using a RDD to run fn server in DinD mode and build, deploy and run a function
          test-fn2                                                 Tests using a RDD to run fn server in normal mode and build, deploy and run a function
          test-rdd-volumes1                                        Test the use of bind mounts with a RDD
          test-rdd-volumes2                                        Test the use of volumes with a RDD
```

The following environment variables must be set:
* `USERNAME` - Any docker Hub username
* `PASSWORD` - Any docker Hub password
* `PROD_OR_STAGING` (Optional) - A string used to distinguish between any workflows that might be run concurrently using the same username. This is appended to any image tags that are created in order to avoid name clashes on Docker Hub. Use lower case characters only. 

## Running these tests locally

You can run these tests locally using the Wercker CLI. To do this, clone this repository and set the following environment variables (see above for a description of each):
```
export X_USERNAME=<username>
export X_PASSWORD=<password>
export X_PROD_OR_STAGING=dev
```
then navigate to the root directory of your cloned repository and run
```
wercker workflow tests
```

# Adding a new test pipeline

If you add a new pipeline to `wercker.yml` then before creating a PR and merging your changes you should
* Update the workflow in `wercker.yml` (so that it will get run if someone does `wercker workflow tests`):
  * Your new pipeline should be dependent on the `build` pipeline
  * No need to follow it with a fan-in the `all-tests-passed` pipeline
* Add your new pipeline to the summary table above 
* Test your new pipeline by running it locally (including `wercker workflow tests`)

After your PR has been approved and merged you can then add it to the automated build:

* Update the workflow using the UI: your new pipeline should be dependent on the `build` pipeline
* Manually trigger a build to verify that your new pipeline works. 
  
After you have confirmed that your new pipeline passes 

* Further update the workflow using the UI: your new pipeline should be followed by a fan-in to the `all-tests-passed` pipeline

Since this means that your new pipeline will not be tested in a hosted environment until after your PR has been merged
it is probably best to create an empty pipeline initially and go through the process above to add it to the workflow.
Then create a new feature branch and start making changes to your pipeline. Each time you push changes to your feature branch
the entire workflow, including your updated pipeline, will be run in hosted wercker.



