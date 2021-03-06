# Terraform Provider: Snowflake

**Please note**: If you believe you have found a security issue, _please responsibly disclose_ by contacting us at [security@chanzuckerberg.com](mailto:security@chanzuckerberg.com).

----

[![Join the chat at https://gitter.im/chanzuckerberg/terraform-provider-snowflake](https://badges.gitter.im/chanzuckerberg/terraform-provider-snowflake.svg)](https://gitter.im/chanzuckerberg/terraform-provider-snowflake?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![Build Status](https://travis-ci.com/chanzuckerberg/terraform-provider-snowflake.svg?branch=master)](https://travis-ci.com/chanzuckerberg/terraform-provider-snowflake) [![codecov](https://codecov.io/gh/chanzuckerberg/terraform-provider-snowflake/branch/master/graph/badge.svg)](https://codecov.io/gh/chanzuckerberg/terraform-provider-snowflake)

This is a terraform provider plugin for managing [Snowflake](http://snowflakedb.com) accounts.

## Install

The easiest way is to run this command:

```shell
curl https://raw.githubusercontent.com/chanzuckerberg/terraform-provider-snowflake/master/download.sh | bash -s -- -b $HOME/.terraform.d/plugins
```

**Note that this will only work with recent releases, for older releaes, use the version of download.sh that corresponds to that release (replace master in that curl with the version).**

It runs a script generated by [godownloader](https://github.com/goreleaser/godownloader) which installs into the proper directory for terraform (~/.terraform.d/plugins).

You can also just download a binary from our [releases](https://github.com/chanzuckerberg/terraform-provider-snowflake/releases) and follow the [Terraform directions for installing 3rd party plugins](https://www.terraform.io/docs/configuration/providers.html#third-party-plugins).

TODO fogg config

## Usage

In-depth docs are available [on the Terraform registry](https://registry.terraform.io/providers/chanzuckerberg/snowflake/latest).

## Development

To do development you need Go installed, this repo cloned and that's about it. It has not been tested on Windows, so if you find problems let us know.

If you want to build and test the provider locally there is a make target `make install-tf` that will build the provider binary and install it in a location that terraform can find.

## Testing

For the Terraform resources, there are 3 levels of testing - internal, unit and acceptance tests.

The 'internal' tests are run in the `github.com/chanzuckerberg/terraform-provider-snowflake/pkg/resources` package so that they can test functions that are not exported. These tests are intended to be limited to unit tests for simple functions.

The 'unit' tests are run in  `github.com/chanzuckerberg/terraform-provider-snowflake/pkg/resources_test`, so they only have access to the exported methods of `resources`. These tests exercise the CRUD methods that on the terraform resources. Note that all tests here make use of database mocking and are run locally. This means the tests are fast, but are liable to be wrong in suble ways (since the mocks are unlikely to be perfect).

You can run these first two sets of tests with `make test`.

The 'acceptance' tests run the full stack, creating, modifying and destroying resources in a live snowflake account. To run them you need a snowflake account and the proper authentication set up. These tests are slower but have higher fidelity.

To run all tests, including the acceptance tests, run `make test-acceptance`.

We also run all tests in our [Travis-CI account](https://travis-ci.com/chanzuckerberg/terraform-provider-snowflake).

### Pull Request CI

Our CI jobs run the full acceptence test suite, which involves creating and destroying resources in a live snowflake account. Travis-CI is configured with environment variables to authenticate to our test snowflake account. For security reasons, those variables are not available to forks of this repo.

If you are making a PR from a forked repo, you can create a new Snowflake trial account and set up Travis to build it by setting these environement variables:

* `SNOWFLAKE_ACCOUNT` - The account name
* `SNOWFLAKE_USER` - A snowflake user for running tests.
* `SNOWFLAKE_PASSWORD` - Password for that user.
* `SNOWFLAKE_ROLE` - Needs to be ACCOUNTADMIN or similar.
* `SNOWFLAKE_REGION` - Default is us-west-2, set this if your snowflake account is in a different region.

If you are using the Standard Snowflake plan, it's recommended you also set up the following environment variables to skip tests for features not enabled for it:

* `SKIP_DATABASE_TESTS` - to skip tests with retention time larger than 1 day
* `SKIP_WAREHOUSE_TESTS` - to skip tests with multi warehouses

## Releasing

**Note: Currently only @ryanking and @edulop91 have keys that are whitelisted in the terraform registry, so only they can do releases.**

Releases are done by [goreleaser](https://goreleaser.com/) and run by our make files. There two goreleaser configs, `.goreleaser.yml` for regular releases and `.goreleaser.prerelease.yml` for doing prereleases (for testing).

As of recently releases are also [published to the terraform registry](https://registry.terraform.io/providers/chanzuckerberg/snowflake/latest). That publishing requires that releases by signed. We do that signing via goreleaser using individual keybase keys. They key you want to use to sign must be passed in with the `KEYBASE_KEY_ID` environment variable.
