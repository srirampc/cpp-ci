
The following describes the structure of yml file for CI with CMake.

## Name, Triggers and Enviroment.

Apart from the name, we also provide the triggers. 
In the following configuration, a build and test is triggered  when 
(A)  commit is pushed to the master branch and 
(B) a pull request is created for the master branch.

The `workflow_dispatch` option is useful to initiate a Build, Test and CI 
workflow without the trigger action.

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

All the steps are defined under `jobs` header in the yaml. First, we define the 
github runner on which the soruce is built and tested. More info on runners is 
[here](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)

```
jobs:
  Build-Test:
    runs-on: ubuntu-latest
```
All of the following  steps in the workflow are defined underneath a `steps` subheader.

## Setup Enviroment

Define the steps for setting up enviroment in the runner. Here,
we checkout the source code and install the required libraries.
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

Perform Static analysis with cpp check and publish the report to the repository.

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
  Configure,Build and Test with [CMake](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html?highlight=cmake_build_type)
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

To compute and upload coverage we do the following steps
 1. Run lcov to generate coverage
 2. Report code coverage
 3. Uploading to Codecov

```
    - name: Run lcov to generate coverage
      working-directory: ${{github.workspace}}/build
      run: make lcov2
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

Here the first step `make lcov2` is building a traget defined in the CMake file. 
The target executes the following commands.

```
lcov -c -d ./ -o FastANITest.info
lcov -e FastANITest.info "*/FastANI/src/*/*.cpp" "*/FastANI/src/*/*.hpp"  > coverage.info
lcov --list coverage.info |tee coverage_list.txt
lcov --summary coverage.info |tee coverage_summary.txt
```

The second step uploads the coverage summary to the Github Action execution corresponding the 
current commit. The third step uploads it to the [codecov.io](www.codecov.io).
