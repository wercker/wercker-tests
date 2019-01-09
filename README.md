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

Each of the above sequences is followed by a fan-in to the `all-tests-passed` pipeline.

The following environment variables must be set:

* `PROD_OR_STAGING` - A string used to distinguish between any workflows that might be run concurrently using the same username. This is appended to any image tags that are created in order to avoid name clashes on Docker Hub. Use lower case characters only. 

For tests that use Docker Hub:

* `USERNAME` - Any Docker Hub username
* `PASSWORD` - Docker Hub password for $USERNAME

For tests that use Amazon ECR:

* `AWS_REGION` - AWS region (for example `us-east-2`)
* `AWS_ACCESS_KEY_ID` - AWS access key
* `AWS_SECRET_ACCESS_KEY` - AWS secret access key
* `ECR_REGISTRY` - ECR registry. For example https://<aws_account_id>.dkr.ecr.<region>.amazonaws.com for the AWS defaiult registry.

For tests that use Oracle Cloud Infrastructure Registry (OCIR):

* `OCI_REGION` - OCI region (for example `iad`)
* `OCI_TENANCY_NAME` - OCI tenancy
* `OCIR_USERNAME` - OCI username, in the form $OCI_TENANCY_NAME/foo@bar.com
* `OCIR_AUTH_TOKEN` - OCI auth token for $OCIR_USERNAME

The following environment variables are optional.  They can be used to configure pagerduty notifications:

* `PAGERDUTY_SERVICE_KEY` - PagerDuty service key that defines the notification will be sent to. If not set then the notification after-step will fail harmlessly.
* `PAGERDUTY_NOTIFY_ON` - Leave unset or set to `failed` to send notifications when the pipeline fails. Set to anything else (such as `all`) to send pagerduty notifications when the pipeline passes as well.
* `PAGERDUTY_BRANCH` - Leave unset to send notifications for builds on all branches. Set to a branch name (such as `master` to send notifications only for builds on the specified branch. 

## Running these tests locally

You can run these tests locally using the Wercker CLI. To do this, clone this repository and set the following environment variables (see above for a description of each):
```
export X_USERNAME=<username>
export X_PASSWORD=<password>
export X_PROD_OR_STAGING=dev

export X_AWS_REGION=<aws-region>
export X_AWS_ACCESS_KEY_ID=<aws-access-key>
export X_AWS_SECRET_ACCESS_KEY=<aws-secret-access-key>
export X_ECR_REGISTRY=<ecr-registry>

export X_OCI_REGION=<region>
export X_OCI_TENANCY_NAME=<oci-tenancy>
export X_OCIR_USERNAME=<oci-username>
export X_OCIR_AUTH_TOKEN=<oci-auth-token>
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



