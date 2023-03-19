
This document explains the steps for using [Github Actions](https://docs.github.com/en/actions) for a [CMake](https://cmake.org/) project.
First, we discuss how to add a target in CMake file to generate coverage information. Next, we explain how to define github actions. We use [FastANI](https://github.com/srirampc/FastANI) project as an example. 

# Coverage for C++ projects

To perform Conitnous Integration with CMake, we first add a target to build the coverage files. Coverage files are built after the project is built with appropriate flags and the unit tests are run.

## Compiling for Coverage 

In order for the coverage files to be generated, the code should be compiled with optimizations turned off and the compiler produces necessary information to gather coverage data.
For `gcc` compiler, these flags are: `-fprofile-arcs -ftest-coverage -g -O0`.


## Generate Coverage Information
Running the instrumented binary generates a profile output. For each source file, compiled with -fprofile-arcs, a `.gcda` profile output file is created in the object directory.

To generate the coverage information from the profile output files, we add an new target in the CMakeLists.txt that generates coverage. 
We use `lcov`, which is a graphical front-end to the to GCC's `gcov` coverage tool. It collects the line, function and branch coverage data from mutlitple files and can create HTML pages with source code annotated coverage information. It can also filter the coverage data pertaining to standard libraries and third-party libraries and generate overview and summary information.

For our project, the `lcov` target executes the following set of commands.
```
add_custom_target(lcov  lcov -c -d ./ -o FastANITest.info
                        COMMAND lcov -e FastANITest.info "*/FastANI/src/*/*.cpp" "*/FastANI/src/*/*.hpp"  > coverage.info
                        COMMAND lcov --list coverage.info |tee coverage_list.txt
                        COMMAND lcov --summary coverage.info |tee coverage_summary.txt
                        DEPENDS FastANITest)
```
The first command above compiles the coverage information from all the `.gcda` files in `./` directory tree. The second command selects the coverage information pertaining only to the `FastANI` project i.e., covearge of files in the `FastANI/src` directories. The third and fourth commands lists and summarizes the coverage coverage data.


# Github Actions

Github Actions require the definition of a `.yml` file that should be placed in `.github/actions` directory at the root of the repository.

## Name, Triggers and Enviroment.

First section of the yml file lists the name of the action and the triggers. 
In the following configuration, a build, test and CI job is triggered  when 
(A)  commit is pushed to the master branch and 
(B) a pull request is created for the master branch.

We also set the `workflow_dispatch` option under the `on` sub-head, because it
is useful to initiate a Build, Test and CI job from the Github->Actions dashboard
without a trigger action.

We use the `Debug` environment so that the necessary compilation options
required for building with symbols are set for coverage to be estimated 
correctly.  Other options are `Release`, `Debug`, `RelWithDebInfo`, etc.

```
name: Build with CMake and Test

run-name: Build FastANI with CMake and Test by ${{github.actor}}

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

  workflow_dispatch:

env:
  BUILD_TYPE: Debug
```

## Configure, Static Analysis, Build, Test and Coverage.

All the next steps are defined under `jobs` header in the yaml. First, we define the 
github runner on which the soruce is built and tested. For public projects, the runners
are free. For private projects, users need to purchase credits. More info on runners is
available [here](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)
```
jobs:
  Build-Test:
    runs-on: ubuntu-latest
```
All of the following  steps in the workflow are defined underneath a `steps` subheader 
of the `jobs` header.

## Setup Enviroment

Define the steps for setting up enviroment in the runner. Here,
we checkout the source code and install the required libraries
for building, testing and static analysis and code coverage.
```
    steps:
    - uses: actions/checkout@v3
      with:
        submodules: recursive
        lfs: true
    - name: Install zlib, gsl, cppcheck and lcov
      run: |
          sudo apt-get update
          sudo apt-get install zlib1g-dev libgsl-dev cppcheck lcov
```

## Static Analysis

Static analysis is performed with with cpp check and the report is published 
to the repository as a commit.

```
    - name: cppcheck
      run: cppcheck --enable=all --inconclusive --force --std=c++20 -iext/ -I src/ --output-file=cppcheck_report.txt . 
          
    - name: publish report    
      uses: mikeal/publish-to-github-action@master
      env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          BRANCH_NAME: 'master' # your branch name goes here
```

## Configure, Build and Test
  Configure, Build and Test with [CMake](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html?highlight=cmake_build_type)
```
    # Building
    - name: Configure CMake
      run: cmake -B ${{github.workspace}}/build -DCMAKE_BUILD_TYPE=${{env.BUILD_TYPE}} -DBUILD_TESTING=ON
    - name: Build
      run: cmake --build ${{github.workspace}}/build --config ${{env.BUILD_TYPE}}
    - name: Test
      working-directory: ${{github.workspace}}/build
      run: ctest -C ${{env.BUILD_TYPE}}
```
## Running coverage and Updating in codecov

To compute and upload coverage, the following steps are defined.
 1. Run lcov to generate coverage. Call the lcov target in CMake.
 2. Report code coverage.
 3. Upload to Codecov.

```
    - name: Run lcov to generate coverage
      working-directory: ${{github.workspace}}/build
      run: make lcov
    - name: Report code coverage
      uses: zgosalvez/github-actions-report-lcov@v2
      with:
        working-directory: ${{github.workspace}}/build
        coverage-files: ${{github.workspace}}/build/coverage.info
        artifact-name: code-coverage-report
        github-token: ${{ secrets.GITHUB_TOKEN }}
    - name: Upload coverage reports to Codecov with GitHub Action
      uses: codecov/codecov-action@v3
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        directory: ${{github.workspace}}/build
        files: ${{github.workspace}}/build/coverage.info
        fail_ci_if_error: true
        verbose: true
```

As mentioned above, `make lcov` builds the traget `lcov` defined in 
the CMakeLists.txt file. 
The second step uploads the coverage summary to the Github Action execution corresponding the 
current commit or pull request. The third step uploads it to the [codecov.io](www.codecov.io).

# Adding Badges.

Build and CI Badges can be added to the repository README.md as follows:

```
![Build with CMake and Test](https://github.com/srirampc/FastANI/actions/workflows/main.yml/badge.svg)
[![codecov](https://codecov.io/gh/srirampc/FastANI/branch/master/graph/badge.svg?token=B7YFZ56BW2)](https://codecov.io/gh/srirampc/FastANI)
```

