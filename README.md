# Chef GitLab CI/CD Pipelines ~templates and~ examples

## Policyfile Workflow

Examples of usage in complete pipelines in the `example_pipelines` directory for:

* **Cookbook CI (with a Policyfile for dependency resolution only, instead of Berksfile)**
  * Prep:
    * Git tag for version in `metadata.rb` does not exist yet
  * Lint:
    * Cookstyle
    * ChefSpec
  * Integration Testing
  * Release:
    * Git Tagging with version in `metadata.rb`
* **Policyfile CI/CD**
  * Including promotion using [GitLab Environments](https://docs.gitlab.com/ee/ci/environments/) mapped to Chef Infra Server Policy Groups 
  * Prep:
    * Update Policyfile Lock JSON (`chef install` and `chef update`)
    * Publish updated Policyfile Lock JSON as pipeline artifact
  * Integration Testing
  * Publish:
    * Export Policyfile archive (`chef export -a`)
    * Publish updated Policyfile Lock JSON as build artifact
  * Release/Deploy:
    * Use `deployment` jobs with `environments` mapped to policy groups and push Policyfile archive to Policy group (`chef push-archive`)

### Example Policyfiles

```ruby
# A name that describes what the system you're building with Chef does.
name 'test-policy'

# Where to find external cookbooks:
default_source :supermarket

# run_list: chef-client will run these recipes in the order specified.
run_list 'test::default', 'chef-client::default', 'audit::default'

# Specify a custom source for a single cookbook:
cookbook 'test', git: 'https://GITLABREPOHERE/_git/test', tag: 'v0.1.8'

default['development']['test']['message'] = 'test message for development'
default['production']['test']['message'] = 'test message for production'
```

The Policyfile CI/CD pipeline also works with Policyfiles that use `include_policy` from another repo (by automatically commiting the Policyfile Lock JSON so it's available for other policies to include) e.g.

```ruby
# This is the file server policy for all Chef managed file servers.
name 'another-policy'

# Where to find cookbooks:
default_source :supermarket

# include base policy
include_policy 'test-policy', git: 'https://GITLABREPOHERE/_git/test-policy', path: 'test-policy.lock.json'

# Chef-client will run these recipes in the order specified.
run_list 'openssh::default'
```
