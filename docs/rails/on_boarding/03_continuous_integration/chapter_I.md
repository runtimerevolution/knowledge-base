# Chapter I
`(avr. time for this chapter: 1 to 2 days)`

The goal of this exercise is to automatize the process of validating your code, a process known by the name Continuous Integration. Among several other benefits, this will help you reduce the risk of introducing new bugs to your project and allows you to have your project in a state always ready to be deployed.

To do this we suggest you use one of the following three platforms:

- [Github Actions](https://docs.github.com/en/actions)
- [Bibucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)
- [GitLab](https://docs.gitlab.com/ee/ci/)

***Let's dive in to***

Steps to take into consideration for this exercise::

- Project setup
- Run tests
- Code styling
- Test coverage

## Project setup

In this step, you should take into consideration the version of your tools.

For Bitbucket Pipelines and Gitlab, you can use an official docker image from [Docker Hub](https://hub.docker.com) with the required ruby version. But nothing stops you from using a custom docker image.

For Github Actions the strategy is a bit different. Here you choose the system (Linux, Windows, or macOS), and instead of docker images, you rely on actions for the setup. Ruby provides an [official action](https://github.com/ruby/setup-ruby) to perform the setup.

## Run tests

As you keep adding new features it is important to write tests to keep validating your solution. Running these tests on the CI will guarantee you never forget to check your code. At this point of the onboarding, you should already have some Rspec tests you can use.

## Code styling

For consistency and better reading of your code now and in the future it is important to keep using the same code styling. Adding linting to your CI process will force you to respect these rules. The most common gem used for this process is [Rubocop](https://docs.rubocop.org/rubocop/1.18/installation.html).

## Test coverage

Test coverage is another topic you can add to your CI process which will determine whether your test cases cover the application code and how much code is exercised when running those test cases. This comes in handy to let developers know if more tests are needed to have a robust application. The most common gem used for this process is [simplecov](https://github.com/simplecov-ruby/simplecov). Notice that this gem allows you to combine it with other formatters for different output files. For example, you can combine with [simplecov-json](https://github.com/vicentllongo/simplecov-json) for easier file handling of the results. Depending on the service you choose you may need to install packages to read json if you choose this path. The most common package to parse json files from the command-line is [./jq](https://jqlang.github.io/jq/).
