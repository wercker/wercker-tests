# wercker-tests
A  repository for testing pipelines in wercker
 
To run these tests:

Fork this repository and add it as an application in Wercker. 

Configure the new application to run the following workflow:
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
Configure your application with the following environment variables 
* `USERNAME` - Your Docker Hub username
* `PASSWORD` - Your Docker Hub password
* `PROD_OR_STAGING` (Optional) - A string used to distinguish between any workflows that might be run concurrently using the same username. This is appended to any image tags that are created in order to avoid name clashes on Docker Hub. Use lower case characters only. 

Then run the `build` pipeline,  either manually (easier) or by making a token edit to your clone of this repo and pushing it.

See the [latest runs on app.wercker.com](https://app.wercker.com/nigeldeakin/wercker-tests/runs)

See the [latest runs on dev.wercker.com](https://dev.wercker.com/nigeldeakin/wercker-tests/runs)

## Running these tests locally

To run these tests locally using the wercker CLI, set 
```
export X_USERNAME=<username>
export X_PASSWORD=<password>
export X_PROD_OR_STAGING=dev
```
and then run
```
wercker workflow tests
```
