# Parallelizing and randomizing

<!-- TOC -->

- [Parallelizing and randomizing](#parallelizing-and-randomizing)
  - [Parallelizing tests](#parallelizing-tests)
    - [Setup](#setup)
  - [Randomizing tests](#randomizing-tests)
    - [Setup](#setup-1)

<!-- /TOC -->

In unit testing, we want our tests to be fast and independent of any other anything else in the codebase. There are two main ways that we can do this.

### Parallelizing tests

This is useful for running tests to speed up execution of the testing. It is not necessary whenever there are a relatively small number of unit tests, but if your project runs tests slowly this could be a good move to increase their speed.

#### Setup
1. Edit your Scheme
2. Navigate to Test (Debug)
3. Verify that you’re in the “Info” tab
4. In Tests, Select “Options” for your specific test module
5. Select "Execute in parallel(if possible)”
6. Close, and now bask in the quickness of your unit tests



### Randomizing tests
When tests are randomized, they are run in a different sequence each time they run. This is perfect when trying to prevent tests from relying on something else outside of the test.

For exmaple, if we have a singleton that is used in Test A and Test B which it is modified in test A, then Test B would be dependent on whatever Test A did. If Test A changed how it modified the singleton test B would be at risk of breaking.

If the tests are randomized so that Test B could sometimes occur before Test A, Test B would break far sooner so that we could find the flaw in our test setup.

#### Setup

1. Edit your Scheme
2. Navigate to Test (Debug)
3. Verify that you’re in the “Info” tab
4. In Tests, Select “Options” for your specific test module
5. Select “Randomize execution order”
6. Close, and now bask in the randomness of your unit tests
