# Jenkins for CMake

We use a Free-style Project in Jenkins.


## Required Plugins

For jenkins, the [**CMake Builder**](https://plugins.jenkins.io/cmakebuilder/) plugin needs to be installed in jenkins server.

'Configure' for a free-style project contains the following sections
1. General
2. Source Code Management (SCM)
3. Build Triggers
4. Build Environemnt
5. Build Steps
6. Post-Build Actions.

In the General section, we add a description of the project and provide options to throttle builds or execute concurrent builds.

For this project, we keep the default 'Build Triggers' and 'Build Environemnt'. Also, there are no post-build actions steps. Other sections are defined as follows:

##  SCM Configuration

Select Git for the SCM and select the following options:

### Link to the Repository

Add link to the git repository and provide branch information, if it is different.

![Link to Repository](/images/scm_link.png)

### Add Recursive 

Make sure to add Recursive options, if the repository depends on third-party software added through a git submdodlue

![Additional Repo Details](/images/scm_recur.png)


##  Build, Test and Coverage

Build step uses thtee build steps:
1. A CMake Step including the Build,
2. A CTest Step and finally, 
3. A step for running the coverage coverage dtep.


First we select the CMake as build step.

![CMake Build Option](/images/build_cmake_option.png)

### CMake Configure and Build.

Use the Debug configure so that the necessary options for generating code with symbols is accomplished.

![](/images/build_cmake.png)

Click "Add Build Tool Invocation" and use the default options for building.

![](/images/build_make.png)

### CMake CTest

Click "Add build step", and add a CTest step for testing. 

![](/images/build_test.png)


### Coverage with lcov

Finally, add an "Execute Shell' build step to run the coverage

![](/images/build_lcov.png)

Here we use coverage with lcov. In our case, we use a target in CMakeLists.txt which executes the following commands.
```
lcov -c -d ./ -o FastANITest.info
lcov -e FastANITest.info "*/FastANI/src/*/*.cpp" "*/FastANI/src/*/*.hpp"  > coverage.info
lcov --list coverage.info |tee coverage_list.txt
lcov --summary coverage.info |tee coverage_summary.txt
```
