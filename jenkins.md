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


##  Build Configuration

Build step uses a CMake/CTest Step, for which we select the following options.

## CMake configuration
![](/images/build_cmake.png)

## CMake Build and Test

![](/images/build_make.png)

## CMake Coverage

![](/images/build_coverage.png)
