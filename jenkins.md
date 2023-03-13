# Jenkins for CMake

We use a Free-style Project in Jenkins.


## Required Plugins

For jenkins, the [**CMake Builder**](https://plugins.jenkins.io/cmakebuilder/) plugin needs to be installed in jenkins server.


##  SCM Configuration


### Link to the Repository

Add link to the git repository and provide branch information, if it is different.

![Link to Repository](/images/scm_link.png)

### Add Recursive 

Make sure to add Recursive options, if the repository depends on t 

![Additional Repo Details](/images/scm_recur.png)


##  Build, Test and Coverage

Build step uses a CMake/CTest Step, a Build Step, a Test Step and finally, a coverage Step.

## CMake Configure.

Use the Debug configure so that the necessary options for generating code with symbols is accomplished.

![](/images/build_cmake.png)

## CMake Build

Use the default options for building.

![](/images/build_make.png)

## CMake Test

Use the CTest option for testing.

![](/images/build_test.png)


## Coverage with lcov

Here we use coverage with lcov, with the nessary commands already provided in CMakeLists.txt.

```
lcov
```

![](/images/build_lcov.png)
