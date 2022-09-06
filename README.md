<p align="center">
    <a href="https://github.com/DevelopersToolbox/">
        <img src="https://cdn.wolfsoftware.com/assets/images/github/organisations/developerstoolbox/black-and-white-circle-256.png" alt="DevelopersToolbox logo" />
    </a>
    <br />
    <a href="https://github.com/DevelopersToolbox/caretaker-core/actions/workflows/cicd-pipeline.yml">
        <img src="https://img.shields.io/github/workflow/status/DevelopersToolbox/caretaker-core/CICD%20Pipeline/master?style=for-the-badge" alt="Github Build Status">
    </a>
    <a href="https://github.com/DevelopersToolbox/caretaker-core/releases/latest">
        <img src="https://img.shields.io/github/v/release/DevelopersToolbox/caretaker-core?color=blue&label=Latest%20Release&style=for-the-badge" alt="Release">
    </a>
    <a href="https://github.com/DevelopersToolbox/caretaker-core/releases/latest">
        <img src="https://img.shields.io/github/commits-since/DevelopersToolbox/caretaker-core/latest.svg?color=blue&style=for-the-badge" alt="Commits since release">
    </a>
    <br />
    <a href=".github/CODE_OF_CONDUCT.md">
        <img src="https://img.shields.io/badge/Code%20of%20Conduct-blue?style=for-the-badge" />
    </a>
    <a href=".github/CONTRIBUTING.md">
        <img src="https://img.shields.io/badge/Contributing-blue?style=for-the-badge" />
    </a>
    <a href=".github/SECURITY.md">
        <img src="https://img.shields.io/badge/Report%20Security%20Concern-blue?style=for-the-badge" />
    </a>
    <a href="https://github.com/DevelopersToolbox/caretaker-core/issues">
        <img src="https://img.shields.io/badge/Get%20Support-blue?style=for-the-badge" />
    </a>
    <br />
    <a href="https://wolfsoftware.com/">
        <img src="https://img.shields.io/badge/Created%20by%20Wolf%20Software-blue?style=for-the-badge" />
    </a>
</p>

## Overview

Caretaker Core is the new heart of the [Caretaker](https://github.com/DevelopersToolbox/caretaker) tool. The aim is to remove all of nasty parts of Caretaker and move them into this new clean core.

The core processes the complete git log for a given repository and returns all of the required information as a single JSON object.

[Caretaker](https://github.com/DevelopersToolbox/caretaker) makes use of this object in order to dynamically build a CHANGELOG.md file for the given project.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'caretaker-core'
```

And then execute:

    $ bundle install

Or install it yourself as:

    $ gem install caretaker-core

## Usage

The following is a very [simple snippet](testing/get-raw.rb) showing how to integrate the core into your own code. The key is the single call to [CaretakerCore.run](lib/caretaker-core.rb#L20).

```ruby
#!/usr/bin/env ruby

require 'json'
require 'caretaker-core'

begin
    results = CaretakerCore.run
rescue StandardError => e
    puts e.message
    exit
end

puts JSON.pretty_generate(JSON.parse(results))
```

### Output

The output from [CaretakerCore.run](lib/caretaker-core.rb#L20) is a JSON formatted object. In the case of an error it will raise a `StandardError`. 

The basic structure of the JSON is as follows:

```yaml
{
    "tags": [ ],
    "commits": {
        "chronological": { },
        "categorised": { },
    },
    "repo": {
        "url": "",
        "slug": ""
    }
}
```
* tags - An array of tag names, it will also include 'untagged' for all commits that are not part of a tag.
* commits - A hash with 2 elements
    * chronological - All commits in chronological order.
    * categorised - All commits split by category.
* repo - A hash with information relating to the repository
    * url - The full base url to the repo (e.g. https://github.com/DevelopersToolbox/caretaker-core)
    * slug - The github organisation / repository name (e.g. DevelopersToolbox/caretaker-core)

A more full example (Showing a single commit)

```yaml
{
    "tags": [
        "untagged"
    ],
    "commits": {
        "chronological": {
            "untagged": [
                {
                    "hash": "11781d3",
                    "hash_full": "11781d3cbdac68a003492fc0d318d402dd241579",
                    "subject": "The initial commit",
                    "extra": false,
                    "commit_type": "commit",
                    "category": "Uncategorised:",
                    "date": "2021-03-03"
                }
            ]
        },
        "categorised": {
            "untagged": {
                "New Features:": [ ],
                "Improvements:": [ ],
                "Bug Fixes:": [ ],
                "Security Fixes:": [ ],
                "Refactor:": [ ],
                "Style:": [ ],
                "Deprecated:": [ ],
                "Removed:": [ ],
                "Tests:": [ ],
                "Documentation:": [ ],
                "Chores:": [ ],
                "Experiments:": [ ],
                "Miscellaneous:": [ ],
                "Uncategorised:": [
                    {
                        "hash": "11781d3",
                        "hash_full": "11781d3cbdac68a003492fc0d318d402dd241579",
                        "subject": "The initial commit",
                        "extra": false,
                        "commit_type": "commit",
                        "category": "Uncategorised:",
                        "date": "2021-03-03"
                    }
                ],
                "Initial Commit:": [ ],
                "Skip:": [ ]
            }
        }
    },
    "repo": {
        "url": "https://github.com/DevelopersToolbox/caretaker-core",
        "slug": "DevelopersToolbox/caretaker-core"
    }
}
```

#### Other Values

| Name | Purpose | Possible Values |
| ---- | ------- | --------------- |
| hash | Stores the short hash for the commit. | 7 character hexidecimal string |
| hash\_full | Stores the full hash for the commit. | 40 character hexodecimal string|
| commit\_message | The commit message message. | Anything alphanumberic |
| child\_commit\_messages | The commit messages of any commits that form part of a pull/merge request | Anything alphanumberic or false |
| commit\_type | The type of commit | pr or commit |
| category | The category the commit belongs to | [Category List](lib/caretaker-core/config.rb#L13) |
| date | The date the commit was made | YYYY-MM-DD format |

> For more information about the use of categories - please refer to the caretaker documentation.

### Limitations with regards to Pull Requests

GitHub has introduced new options when it comes to pull requests, see the list below for more details.

Although the code will handle any of the options, we recommend that you use 'squashed commits' and use the default setting. This will maximise the usefulness of the 'child_commit' messages.

#### Merge Commits

1. Default
2. Pull request title
3. Pull request title and description

#### Squash Merging

1. Default
2. Pull request title
3. Pull request title and commit details
4. Pull request title and descrption

#### Rebased Commits

1. Default - show as local commits as there is no way to see where they came from.

#### Further Reading

[Github Documentation](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github)

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).
