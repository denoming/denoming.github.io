---
title: "CTest cheatsheet"
date: 2021-08-31
categories: [Development,Tools]
tags: [syncthing,tools]
---

## Add tests

Ordinary template of project with tests:
```cmake
cmake_minimum_required(VERSION 3.0)
project(CTestExample)
enable_testing()

add_executable(TestApp testapp.cpp)
add_test(NAME noArgs COMMAND TestApp)

set_tests_properties(noArgs PROPERTIES
	ENVIRONMENT "Foo=bar;HAVE_BAZ=1"
	LABELS "Foo"
)
```

Or with correctly expressions substitution:
```cmake
...
add_test(NAME withArgs
	COMMAND someCommand $<$<CONFIG:Debug>:-g>
	COMMAND_EXPAND_LISTS
)

...
```

### Useful test properties

Specifying environment variables for tests:
```cmake
set_tests_properties(noArgs PROPERTIES
	ENVIRONMENT "Foo=bar;HAVE_BAZ=1"
)
```

Specifying certain labels for tests:
```cmake
set_tests_properties(noArgs PROPERTIES
	LABELS "Foo;Bar"
)
```

Specifying inter-tests dependencies:
```cmake
### Setup/cleanup
# Declare test with certain requirement declaration
set_tests_properties(StartServer
	PROPERTIES FIXTURES_SETUP Server
)
set_tests_properties(StopServer
	PROPERTIES FIXTURES_CLEANUP Server
)
# Declare test with certain requirement declaration
set_tests_properties(EnsureDatabaseAvailable
	PROPERTIES FIXTURES_SETUP Database # Declare requirement
)
### Client tests
set_tests_properties(ClientNoDb
	PROPERTIES FIXTURES_REQUIRED Server
)
# Declare test and specify needed requirements
set_tests_properties(ClientWithDb
	PROPERTIES FIXTURES_REQUIRED "Server;Database"
)
```

Note: Check the full list of available properties at [Properties on Tests](https://cmake.org/cmake/help/v3.20/manual/cmake-properties.7.html#test-properties).

## Run tests

Commands below should be invoked under build directory or starting from cmake version 3.20 particular directory can be specified by `--test-dir` option.
```sh
### Run tests
$ cd <path-to-project-build-dir>
$ ctest
$ ctest -j$(nproc) # Run tests in parallel mode
### Run tests with custom behaviour
$ ctest --output-on-failure
$ ctest --stop-on-failure
$ ctest --timeout 30 
$ ctest --timeout 30 --stop-time 13:00
$ ctest --repeat-until-fail 3
$ ctest --repeat until-fail:3 # repeat 3 times until fail
$ ctest --repeat until-pass:3 # repeat 3 times until pass
### Run tests in verbose mode
$ ctest -V
### Tests selection
$ ctest -R Only        # Includes tests with 'Only' in name
$ ctest -R '^Foo'      # Includes tests with name match given expression
$ ctest -R 'Foo|Bar'
$ ctest -E Bar         # Excludes tests which names contains 'Bar'
$ ctest -N             # Print the list of tests
$ ctest -N -R '^Bar'   # Print the list of test with matched name
$ ctest -L Foo         # Includes tests with 'Foo' label
### Preset selection
$ ctest --preset <name>
$ ctest --list-presets
```

### Run tests in complete mode:

Complete mode has next look:
```sh
$ cd <path-to-project-build-dir>
$ ctest --build-and-test <sourceDir> <buildDir> \
        --build-generator <generator> \
        [options...] \
        [--test-command testCommand [args...]]
```

For example:
```sh
$ cd <path-to-project-build-dir>
$ ctest --build-and-test . build \
        --build-generator Ninja  \
        --build-options -DCMAKE_BUILD_TYPE=Debug \
                        -DBUILD_SHARED_LIBS=ON   \
        --test-command ctest -j$(nproc)
```

The given above command perform full configure-clean-build-test pipeline. To disable certain steps the next additional options can be used: `--build-nocmake` and `--build-noclean`. These options disable the configure and clean steps respectively.

### Run tests with resource management

Resource management provides opportunity to control system resources during testing and prevent excess of them. Particular runner uses system resources in certain amount that was defined in specific file.

#### Defining  test resource requirements

```cmake
...
add_test(NAME withResourseManagement COMMAND someCommand)

set_tests_properties(withResourceManagement PROPERTIES
	RESOURSE_GROUPS
		mem_gb:16,cpus:4
		producers:1,consumers:1
)
```

#### Defining available system resources

For this purpose you should create certain file in JSON format:
```json
{
   "version": {
      "major": 1,
      "minor": 0
   },
   "local": [
      "mem_gb": [
         {
            "id": "pool_0",
            "slots": 64
         }
      ],
      "cpus": [
         {
            "id": "pool_0",
            "slots": 8
         }
      ],
      "producers": [
         {
            "id": "0",
            "slots": 2
         },
         {
            "id": "1",
            "slots": 8
         }
      ],
      "consumers": [
         {
            "id": "0",
            "slots": 8
         }
      ]
   ]
}
```
Then this file can be used in particular `ctest` invocation.

Note: Check full description about resourse specification file at [Resource Specification File](https://cmake.org/cmake/help/v3.20/manual/ctest.1.html#id35).

#### Perform testing with resource management

```sh
$ cd <path-to-project-build-dir>
$ ctest -R withResourceManagement --resource-spec-file resources.json
```
