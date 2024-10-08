# Ruby - Rails - RSpec - Tips - Patterns

![cover](https://raw.githubusercontent.com/xkwd/learning-ruby-rails/master/cover.png)

## Rubypedia

This repository was created with an idea to collect worthy tips about Ruby/Rails development and currently covers the following topics:

- [Ruby tips](#ruby-tips) - rather less obvious coding tips
- [Rails tips](#rails-tips) - difficult to remember basic Rails tips
- [RSpec tips](#rspec-tips) - code samples and workarounds for non-trivial cases
- [Patterns](#patterns) - basic patterns
- [Development Tools](#development-tools) - tips for committing and pushing code
- [Ruby gems](#ruby-gems) - less popular but very useful Ruby gems
- [Glossary](#glossary) - explanations for popular development words

![cover](https://raw.githubusercontent.com/xkwd/learning-ruby-rails/master/gems.png)

## Table of contents

- [Patterns](#patterns)
  - [Service Object](#service-object)
  - [Decorator](#decorator)
  - [Facade](#facade)
- [Data Structures](#data-structures)
  - [Singly Linked List](#singly-linked-list)
- [Algorithms](#algorithms)
  - [Mergesort](#mergesort)
- [Development Tools](#development-tools)
  - [Skipping brew update when running brew install](#skipping-brew-update-when-running-brew-install)
  - [Using Git and GitHub](#using-git-and-github)
    - [Aliases](#aliases)
    - [Safer push force](#safer-push-force)
    - [New feature development](#new-feature-development)
    - [Committing only certain file changes](#committing-only-certain-file-changes)
    - [Rebasing onto master](#rebasing-onto-master)
  - [Continuous Integration](#continuous-integration)
- [RSpec tips](#rspec-tips)
  - [Unit vs integration tests](#unit-vs-integration-tests)
  - [RSpec glossary](#rspec-glossary)
  - [Running a spec with a specified seed](#running-a-spec-with-a-specified-seed)
  - [Stubbing method call via block with access to passed arguments](#stubbing-method-call-via-block-with-access-to-passed-arguments)
  - [Stub vs Mock](#stub-vs-mock)
  - [Adding mocks inside of the raise_error matcher](#adding-mocks-inside-of-the-raise_error-matcher)
  - [Stub for iterative object initialization](#stub-for-iterative-object-initialization)
  - [Stub and Mock for multiple method calls with different arguments](#stub-and-mock-for-multiple-method-calls-with-different-arguments)
  - [Testing retry after rescue](#testing-retry-after-rescue)
  - [Delay method execution](#delay-method-execution)
  - [Customized failure message](#customized-failure-message)
  - [Skipping callbacks](#skipping-callbacks)
  - [Test examples](#test-examples)
    - [Model tests](#model-tests)
    - [Controller tests](#controller-tests)
    - [Decorator tests](#decorator-tests)
    - [Scope tests](#scope-tests)
    - [Module tests](#module-tests)
- [Rails tips](#rails-tips)
  - [Rails commands](#rails-commands)
  - [Gem versions in Gemfile](#gem-versions-in-gemfile)
  - [How to update gems](#how-to-update-gems)
  - [Rails ActiveRecord model structure](#rails-activerecord-model-structure)
  - [DB migrations](#db-migrations)
  - [Multi-line strings](#multi-line-strings)
  - [Different flavors of scopes](#different-flavors-of-scopes)
- [Ruby tips](#ruby-tips)
  - [The method_missing method](#the-method_missing-method)
  - [Initialize - self.name vs @name](#initialize---selfname-vs-name)
  - [Positional and keyword parameters](#positional-and-keyword-parameters)
  - [Alternative to string interpolation](#alternative-to-string-interpolation)
  - [Set object state with a block](#set-object-state-with-a-block)
  - [Product of two arrays](#product-of-two-arrays)
  - [Array generator](#array-generator)
  - [XOR operator](#xor-operator)
  - [Timestamp generator](#timestamp-generator)
  - [Memoization](#memoization)
  - [One-liner nested hash creation](#one-liner-nested-hash-creation)
  - [One-liner hash creation with array as default value](#one-liner-hash-creation-with-array-as-default-value)
  - [One-liner hash creation for counting occurrences of a certain value](#one-liner-hash-creation-for-counting-occurrences-of-a-certain-value)
  - [Gem versioning](#gem-versioning)
  - [Decorator Stack](#decorator-stack)
  - [Memory cache example](#memory-cache-example)
  - [Splatting with destructuring assignment](#splatting-with-destructuring-assignment)
  - [Save console output to a file](#save-console-output-to-a-file)
- [Ruby gems](#ruby-gems)
  - [Bundler](#bundler)
  - [Savon](#savon)
  - [similar_text](#similar_text)
  - [Kdtree](#kdtree)
  - [PgSearch](#pgsearch)
  - [dry-struct](#dry-struct)
- [SQL](#sql)
  - [Query example](#query-example)
- [Debugging](#debugging)
  - [How to debug a Ruby gem](#how-to-debug-a-ruby-gem)
- [Glossary](#glossary)
  - [Object state](#object-state)
  - [Idempotent method](#idempotent-method)
  - [Predicate method](#predicate-method)

## Patterns

### Service Object

#### 1. Overview

If a Rails app is built by following the Rails Way, it is very likely to end up with fat models and bloated controllers, which in turn are difficult both to test and to be understood by new developers. This is a usual scenario when an app gets bigger than a typical CRUD app. Adding a layer of Service Objects (or simply services) is a way to deal with it.

Service Object is a Plain Old Ruby Object (PORO - a simple object without complicated logic, usually not inheriting from anything) and is a pattern allowing to extract business logic from Models and Controllers (in Model-View-Controller pattern) to keep them slim and thus readable. A single Service Object encapsulates a single process of the business logic. Therefore, it becomes easier to write, test and modify that specific business logic.

#### 2. Common scenarios for usage

Anything that can be re-used, should be tested and does not fit within a single method:

- Triggering logic of multiple models within a single action
- Working with an external API
- Complicated calculations/validations
- etc ...

#### 3. Advantages

- SOLID: a single action encapsulated within a single service complies with Single Responsibility Principle.
- Easy to test.
- Reusable.
- Less coupled.
- Controllers and Models become thinner.
- Allows to get familiar with an exiting Rails app quicker: having a look at Service Objects directory usually provides a quite detailed overview of an app.

#### 4. Naming conventions

When working with an existing app there could be already a naming convention for Service Objects to follow. When dealing with a newly created app the following options are available:

- Naming services after **actions**: **Create**Report, **Update**RemotePrice, **Fetch**FinalResults.
- Using '**or**', '**er**' endings for the last noun in a service name: ReportCreat**or**, RemotePriceUpdat**er**, FinalResultsFetch**er**.
- Appending '**Service**' word to a name based on logic description of a service: ReportCreation**Service**, RemotePriceUpdating**Service**, FinalResultsFetching**Service**.

A Service Object has to have a single public method. Naming options for it are:

- `call`
- `perform`
- `run`
- `execute`

Sticking with the convention simplifies understanding of class responsibility across an app.

#### 5. Directory

Services as Domain Objects should go in a corresponding Domain folder - **app/services**.

When you end up with a lot of services within **app/services** Ruby modules could be used for grouping:

```ruby
module Reports
  class CalculateUserStats
    # ...
  end
end
```

Calling of that Service Object would look like:

```ruby
Reports::CalculateUserStats.new(params).call
```

Folder path in this case should reflect a module structure:

```bash
app
|____services
| |____reports
| | |____calculate_user_stats.rb

```

#### 6. Structure

A typical Service Object consists of a constructor and a single public method that triggers the service. Everything else goes into private methods.

```ruby
class CalculateUserStats
  def initialize(user:)
    @user = user
  end

  def call
    calculate_stats
  end

  private

  attr_reader :user

  def calculate_stats
    # ...
  end
end
```

Consider using keyword arguments in a Service Object constructor for better clarity of your code.

#### 7. Sample test with RSpec

```ruby
require "rails_helper"

describe CalculateUserStats do
  let(:service) { described_class.new(user: user) }
  let(:user) { create(:user) }

  describe "#call" do
    subject { service.call }

    it "calculates user stats" do
      expect { subject }.to change { user.reload.stats }.from(nil).to("calculated_data")
    end
  end
end
```

#### 8. Better `call` method

- Make `call` method as readable as possible.
- Wrap steps inside `call` into transactions, to revert any changes if any of the steps fails.
- Also, you could switch from:

```ruby
CalculateUserStats.new(params).call
```

to:

```ruby
CalculateUserStats.call(params)
```
by adding a class `.call` method:

```ruby
def self.call(params)
  new(params).call
end
```

### Decorator

#### 1. Overview

With the time all those instance methods in a Rails model (e.g. `#full_name`, `#short_description`, `#last_accessed_at`, very often required for presentation purposes) lead to so called 'fat' models. And this is assuming that such logic does not go into views. To avoid 'fat' models, the decorator pattern could be used providing an alternative location for methods which extend functionality of an individual extended/decorated object, without affecting other instances of the same class. In other words, a decorator **wraps** an object with extra methods.

A very important detail of the pattern is that a decorated object should have not only extended methods but also all original methods of its class. This is done by delegating not defined by a decorator methods to a decorated object. Thus, an object always responds to the same interface no matter how many times it is decorated.

Also, assuming that a decorator is only used for presentation logic, it could be called a presenter. Presenter is also a decorator, but solely for presentation logic. Say you have a conditional in a view, then it is rather a case for a presenter, and not a decorator, since a conditional method helps with presentation but does not extend a decorated object.

#### 2. Advantages

- SOLID: a class is extended with a decorator, yet is not modified, which is compliant with Open Closed Principle
- Models become thinner
- Reusable

#### 3. Naming conventions

- Appending 'Decorator' to a class name: Article**Decorator**, Invoice**Decorator**

#### 4. Directory

- app/**decorators**

#### 5. Sample decorator

Using `SimpleDelegator`:

```ruby
class CommentDecorator < SimpleDelegator
  # SimpleDelegator ensures that all methods of a Comment instance (comment = Comment.find(params[:comment_id]))
  # passed on initialization (CommentDecorator.new(comment)), are available in an instance of CommentDecorator

  # If you need to preserve a class of a decorated object/component
  # then there is an option to re-define the class method like below:
  # def class
  #   __getobj__.class
  # end

  def added_at
    # The delegator has access to the model.created_at attribute because a passed comment has it
    created_at.strftime('%A, %B %e')
  end

  def upvotes_count
    # Same thing for comment.ratings
    ratings.where(positive: true).count.to_s
  end
end
```

Thanks to `SimpleDelegator` you have access to methods of a passed component (comment in this case) and can for example access an `id` somewhere in a view.

Plain Old Ruby Object - PORO:

```ruby
class CommentDecorator
  def initialize(comment)
    @comment = comment
  end

  # If PORO is your preference, than this is a way to access component interface:
  # def method_missing(method, *args)
  #   comment.send(method, *args)
  # end

  def added_at
    comment.created_at.strftime('%A, %B %e')
  end

  def upvotes_count
    comment.ratings.where(positive: true).count.to_s
  end

  private

  attr_reader :comment
end
```

This PORO implementation above is not really a decorator, unless method_missing is used to get no access to the interface of a passed component.

Those decorators above are able to work with a single object, though it is not unusual to decorate a collection of objects. The following method could be added to make a decorator work with collections:

```ruby
def self.decorate_collection(collection)
  collection.map { |comment| new(comment) }
end
```

Which would allow to decorate a collection with `CommentDecorator.decorate_collection(user_comments)`.

### Facade

The Facade pattern in Rails is based on moving all the logic from a controller action to a facade and creating a single instance variable with that facade, which is then used in views. Below is an example of refactoring a controller action using the facade pattern:

```ruby
class InterviewsController < ApplicationController
  def new
    @interview = current_user.interviews.build
    @countries = Country.all.map { |c| [c.name, c.id] }
    Section.all.each do |section|
      @interview.answers.build(
        content: "",
        section_id: section.id
      )
    end
  end
end
```

`@interview` and `@countries` have been moved to public instance methods in the facade. The logic with built answers has been moved to a private method, with the help of refactoring how a new interview is built.

```ruby
class InterviewsController < ApplicationController
  def new
    @facade = NewFacade.new(current_user.id)
  end
end
```

```ruby
class InterviewsController::NewFacade
  include InterviewsController::Common

  def initialize(user_id)
    @user_id = user_id
  end

  def interview
    Interview.new(user_id: user_id, answers: answers)
  end

  private

  attr_reader :user_id

  def answers
    Section.ids.map do |section_id|
      Answer.new(content: '', section_id: section_id)
    end
  end
end
```

```ruby
module InterviewsController::Common
  # this has been put in the module because of being shared across multiple facades
  def countries
    @countries ||= Country.pluck(:name, :id)
  end
end
```

```ruby
= form_for @facade.interview do |f| # using in a view
```

Pros:

- Makes controllers skinnier, thus easier to read
- Uses a single instance variable for a corresponding facade
- Complies with SRP
- Makes testing of both controllers and facades easier
- Decrease the barrier of changing code

Cons:

- Add an extra layer of abstraction, which in turn requires some time to adjust
- Generates plenty of new classes and corresponding specs

## Data Structures

### Singly Linked List

```ruby
class ListNode
  attr_accessor :val, :next

  def initialize(val = 0, next = nil)
    @val = val
    @next = next
  end
end
```

#### Detecting loop/cycle

```ruby
def detect_cycle(head)
  fast = slow = head

  while fast && fast.next
    slow = slow.next
    fast = fast.next.next
    return true if slow == fast
  end

  return false
end
```

#### Return the node where the cycle begins

```ruby
def detect_cycle(head)
  slow = fast = head

  while fast && fast.next
    slow = slow.next
    fast = fast.next.next
    if slow == fast
      slow = head
      while slow != fast
        slow = slow.next
        fast = fast.next
      end
      return slow
    end
  end
end
```

#### Deleting middle node

```ruby
def delete_middle(head)
  prev = nil
  fast = slow = head

  while fast && fast.next
    fast = fast.next.next
    prev = slow
    slow = slow.next
  end

  return [] unless prev

  prev.next = slow.next
  head
end
```

#### Deleting n-th node from end of linked list

```ruby
def remove_nth_from_end(head, n)
  dummy = ListNode.new(0, head)
  left = dummy
  right = head

  while n > 0 && right
    right = right.next
    n -= 1
  end

  while right
    left = left.next
    right = right.next
  end

  left.next = left.next.next
  dummy.next
end
```

#### Reverse linked list

```ruby
def reverse_list(head)
  curr = head
  prev, nxt = nil
  while curr
    nxt = curr.next
    curr.next = prev
    prev = curr
    curr = nxt
  end
  prev
end
```

## Algorithms

### Mergesort

```ruby
def sort(nums)
    len = nums.size

    return nums if len <= 1

    mid = len / 2
    left = sort(nums[0...mid])
    right = sort(nums[mid...len])

    merge(left, right)
end

def merge(left, right)
    result = []
    until left.size.zero? || right.size.zero?
      if left.first <= right.first
        result << left.shift
      else
        result << right.shift
      end
    end
    result + left + right
end
```

## Development Tools

### Skipping brew update when running brew install

```bash
HOMEBREW_NO_AUTO_UPDATE=1 brew install ...
```

### Using Git and GitHub

#### Aliases

Type less. Use aliases.

Instead of typing an entire command with multiple options you can define an alias for it. With git you can create aliases directly in your `~/.gitconfig` file under the `$HOME` directory or you can add them to `~/.zshrc` or `~/.bash_profile` files depending on your shell of choice.

Aliases are very personal thing and different people prefer different aliases. Below are some examples:

```shell
alias gs='git status'
alias gc='git commit'
alias gcm='git commit -m'
alias ga='git add'
alias go='git checkout'
alias gb='git branch'
alias gd='git diff'
alias gdc='git diff --cached'
alias grsoft='git reset --soft HEAD^'
alias gl='git log --pretty=oneline'
alias gl5='git log --pretty=format:"%h - %s" -5'
alias gpf='git push --force-with-lease'
```

At some point you may forget what a certain alias actually does, and instead of checking the file with all of them you can use `type` command:

```shell
$ type gs
gs is an alias for git status
```

#### Safer push force

Use `git push --force-with-lease` instead of `git push --force`

A force push overwrites a remote branch with your local branch, regardless of the status of that remote branch. It is not safe and you might overwrite other developers commits. Force with lease gives you the flexibility to override new commits on your remote branch, whilst protecting your old commit history
read more how it works [here](http://weiqingtoh.github.io/force-with-lease/).

It is a long command to type, so create an alias for it ;)


#### New feature development

Begin from an updated master branch:

```bash
$ git checkout master
$ git pull origin master
```

Create a new branch for your feature:

```bash
$ git checkout -b new_feature_branch
```

When your feature is ready, push your feature branch to github. Go to the `Pull Requests` tab and create a New Pull Request (PR) selecting your branch as a `compare:` and `master` as a `base`.

```bash
$ git push origin new_feature_branch
```

Let somebody make a review of your PR and merge it to the `master` branch after approval.

In case you need to make some changes in a created PR, just add new commits to your branch and push to see them in that PR.

#### Committing only certain file changes

Use `git add -p` when only a portion of changes in a certain file has to be committed. This might be useful when for example scope of changes in a file is beyond a single commit or when some undesirable local changes have suddenly appeared and require extra time for fixing.

The output will be:

```
Stage this hunk [y,n,q,a,d,/,e,?]?
```

Which means:

```shell
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk nor any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk nor any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks # Type 's' to split if a suggestion is too big
e - manually edit the current hunk
? - print help
```

`git log --grep="keyword"` - shows commits with messages including the keyword

#### Rebasing onto master

It is a very typical situation, when you need some not yet merged feature and you start a new branch based on another branch with that feature, which is not merged yet. It also always comes with a side effect of a base branch being merged to master at some point of time. The following command allows to rebase such branch onto master and preserve commits only with your new feature, without any extra commits.
`git rebase --onto master used_to_be_base_branch_which_is_merged_to_master current_branch_which_is_being_rebased_into_master`

When you happen to have another branch, based on the branch which just was rebased onto master (in this case it is the current_branch_which_is_being_rebased_into_master), you would need a couple of extra steps:
```shell
git pull # update master branch first
git rebase master # rebase your other branch (the 3rd or the 4th etc) to master
git rebase current_branch_which_is_being_rebased_into_master # rebase your other branch (same as above) to current_branch_which_is_being_rebased_into_master
```

### Continuous Integration

#### 1. Overview

**Continuous integration** (**CI**) is the practice of doing multiple merges per day and having an automated verification system to check those merges for issues. The rationale behind this practice is that checking multiple smaller merges is quicker and the corresponding cost, when used with an automated verification system, is small enough to justify avoidance of bigger merges.

#### 2. Rationale for using an automated verification system

- Allows finding bugs immediately upon their introduction.
- Helps to merge more often by avoiding bigger merges.
- Avoids reliance on human verification, which is not as trustworthy as relying on a machine in charge of running tests. Every single change of code now is always triggering a verification system.
- When dealing with bigger projects, it might be extremely time consuming to run all tests locally on each code iteration.
- Remote build machine usually has an environment closer to production in comparison to a local machine, which helps to avoid situations when tests are passing only locally.

#### 3. Popular CI tools (**FREE** for open source projects):

- [CircleCI](https://circleci.com/)

[CircleCI 2.0 for a Rails app](https://circleci.com/docs/2.0/language-ruby/) [article] - this guide provides the sample configuration of `.circleci/config.yml` file with the detailed walkthrough for the Rails demo project maintained by CircleCI.

[CircleCI 2.0 configuration reference](https://circleci.com/docs/2.0/configuration-reference/) [article] - the reference for configuration keys used in the `.circleci/config.yml` file.

- [Travis](https://travis-ci.org/)
- [Jenkins](https://jenkins.io/)

#### 4. CircleCI sample config

```yml
version: 2
jobs:
  build:
    docker:
      - image: circleci/ruby:2.6.2-node-browsers
        environment:
          BUNDLE_PATH: vendor/bundle
          PGHOST: 127.0.0.1
          PGUSER: circleci
          RAILS_ENV: test
      - image: circleci/postgres:10.2
        environment:
          POSTGRES_USER: circleci
          POSTGRES_DB: intapp_test
          POSTGRES_PASSWORD: ""
    working_directory: ~/interview-app
    steps:
      - checkout
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "Gemfile.lock" }}
            - v1-dependencies-
      - run:
          name: install dependencies
          command: |
            bundle install --jobs=4 --retry=3 --path vendor/bundle
      - save_cache:
          paths:
            - ./vendor/bundle
          key: v1-dependencies-{{ checksum "Gemfile.lock" }}
      - run: cp config/database.yml.example config/database.yml
      - run: bundle exec rake db:create
      - run: bundle exec rake db:schema:load
      - run:
          name: run RSpec
          command: |
            mkdir /tmp/test-results
            bundle exec rspec \
              --format progress \
              --format RspecJunitFormatter \
              --out /tmp/test-results/rspec.xml \
              --format progress \
              $(circleci tests glob "spec/**/*_spec.rb" | \
                circleci tests split)
      - store_test_results:
          path: /tmp/test-results
      - store_artifacts:
          path: /tmp/test-results
          destination: test-results
```

#### 5. Travis sample config

```yml
os: linux
dist: xenial
language: ruby
rvm:
  - 2.6.10
  - 2.7.6
  - 3.0.4

cache:
  directories:
    - ./vendor

env:
  - COVERAGE=true

before_install:
  - gem install bundler -v 2.2.27

before_script:
  - cp .env.example .env
  - if [ ! -f ./vendor/cc-test-reporter ]; then
    curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./vendor/cc-test-reporter;
    chmod +x ./vendor/cc-test-reporter;
    fi
  - ./vendor/cc-test-reporter before-build
  - export DATABASE_HOST=localhost
  - psql -c 'CREATE DATABASE app_test;' -U postgres
  - bundle exec sequel -m db/migrations postgres://localhost/app_test

script:
  - bundle exec rake

after_success:
  - ./vendor/cc-test-reporter after-build --exit-code $TRAVIS_TEST_RESULT

addons:
  postgresql: 9.6

notifications:
  email: false
```

### RSpec tips

#### Unit vs integration tests

Unit tests (or specs) are used for testing of a single component (class or module). Only public methods should be tested, not private ones. Public methods do represent a public interface of a component.

Besides units tests there are integration tests. They are used when testing goes beyond the logic of a single component. For instance accessing a database already implies accessing another component, leading to an integration test.

Both unit and integration tests should cover happy path (e.g. with valid arguments), unhappy path (e.g. with invalid arguments) and edge cases (e.g. division by zero).

#### RSpec glossary

`describe` followed by a class name under testing begins a test file.

```ruby
describe ClassName do
  # ...
end
```

`describe` is also used to group tests e.g. for a specific method. When `describe`'ing a method, the convention is to name an instance method like `#instance_method_name` and class method - `.class_method_name`.

```ruby
describe ClassName do
  describe ".class_method_name" do
    # ...
  end

  describe "#instance_method_name" do
    # ...
  end
end
```

`context` is an alias for `describe` and defines a test context and is used to group tests. Proper naming of contexts is important to make test results easier to read. Word `with`, `when`, `if` are good candidates for the first word in a `context` name. Great if contexts are paired, such as "with valid params" followed by "with invalid params".

```ruby
# ...
  context "with valid arguments" do
    # ...
  end

  context "with invalid arguments" do
    # ...
  end
```

Variables are usually defined after `describe` or after `context`. `let` defines a variable which is only built/created when a name of that variable is used in a test (lazy execution) and is not built/created when it is not used. `let!` ensures a variable is created even if it is not used anywhere in a test (eager execution). Lazy execution of `let` allows to use a not yet defined variable within a code block `{ }` (when for example defining other attributes `let(:home) { FactoryBot.create(:house, owner: not_yet_defined) }`) and to define them with different values within multiple contexts `let(:not_yet_defined) { "reader" }`. `subject` is a special variable containing by default an instance of a class under testing.

```ruby
describe ClassName do
  let(:item) { FactoryBot.create(Item, location: location) }
  let!(:item_persisted) { FactoryBot.create(Item) }
  # At this point item_persisted variable is already created in the test databased, while item is not

  # For a Service Object a typical subject could be { ServiceName.new(params).call }
  subject { }

  describe "..." do
    context "with valid arguments" do
      # Here a valid location variable is defined for the lazily executed item from the above
      let(:location) { "Berlin" }

      # ...
    end

    context "with invalid arguments" do
      # This time an invalid value (nil) is used for for the same lazily executed item
      let(:location) { nil }

      # ...
    end
  end
```

`before` is available for execution of some code before all tests or each test, for example preparing something within a specific context.

`it` followed by a description of what is being tested (e.g. `it "successfully generates user stats"`) defines a specific test.

Testing within multiple contexts usually leads to duplications. `shared_contexts` are used to group re-used variables usually defined after a `context` name and `shared_examples` - to group re-used tests. `include_context` and `include_examples` are used to include those previously defined re-used components.

#### Running a spec with a specified seed

`bundle exec rspec spec/models/comment_spec.rb --seed 39103` - run a spec with a specific seed to replicate for example a failed test.

#### Stubbing method call via block with access to passed arguments

```ruby
  # This makes a stub with a return value equal to a passed argument.
  # Such technique, without using the params variable let(:params),
  # allows to test other modifications (e.g. with a setter) of the params.
  allow(ParamsModifier).to receive(:call) do |params|
    params
  end
```

#### Stub vs Mock


```ruby
  # stub - replaces a method (#call) with the code that returns a specified result (true):
  let(:uploader) { instance_double(InterviewUploader) }
  allow(uploader).to receive_messages(call: true)

  # mock - a stub (see above) with an expectation that the method gets called:
  expect(uploader).to have_received(:call).with(:interview)
```

#### Adding mocks inside of the raise_error matcher

When writing a test for a raised error there might be a need to check whether a certain object has been called. Adding a mock to the block of the `raise_error` matcher would do the trick:

```ruby
  expect { result }.to raise_error(CustomErrorName) do |_error|
    expect(Connection).to have_received(:find).twice
  end
```

#### Stub for iterative object initialization

When objects are initialized within an iterator, the following shorter initialization stub for each iteration could be used:

```ruby
allow(Models::Interview).to receive(:new) do |params|
  instance_double(Models::Interview, title: params[:title])
end
```

#### Stub and Mock for multiple method calls with different arguments

```ruby
allow(Service)
  .to receive(:call)
  .and_return(:first, :second)

expect(Service)
  .to have_received(:call)
  .with(:argument1)
  .with(:argument2)
```

#### Testing retry after rescue

```ruby
def call
  Service.call(:params)
rescue StandardError => e
  sleep 1
  @retries += 1
  retry if @retries <= 2
  raise(e)
end
```

```ruby
before do
  call_count = 0
  allow(Service).to receive(:confirm) do
    call_count += 1
    call_count.odd? ? raise(StandardError) : :result
  end

  described_class.call
end

it { expect(Service).to have_received(:call).twice }
```

#### Delay method execution

```ruby
allow(service).to receive(:execute).and_wrap_original do |method, *args|
  sleep 0.2
  method.call(*args)
end
```

#### Customized failure message

We can always improve the readability of the error message in the console by defining a custom message after the matcher:

```ruby
  it 'does this and that' do
    expect(generated_output).to eq(expected_output), expected_output - generated_output
  end
```

#### Skipping callbacks

```ruby
class User < ApplicationRecord
  after_create :assign_titles
end

FactoryBot.define do
  factory :user do
    trait :user_without_assign_titles do
      after(:build) { |user| user.class.skip_callback(:create, :after, :assign_titles, raise: false) }

      after(:create) { |user| user.class.set_callback(:create, :after, :assign_titles, raise: false) }
    end
  end
end
```

#### Test examples

##### Model tests

[Shoulda Matchers](https://github.com/thoughtbot/shoulda-matchers) gem is a must have when testing Rails models. It provides one-liners that would otherwise require more code to test same things.

```ruby
require 'rails_helper'

# Below is a sample model test
describe Article, type: :model do
  describe 'db' do
    # Verifying whether specified db columns exists
    describe 'columns' do
      it { should have_db_column(:title).of_type(:string).with_options(null: false) }
      it { should have_db_column(:description).of_type(:text) }
      it { should have_db_column(:user_id).of_type(:bigint) }
    end

    # Same for indexes
    describe 'indexes' do
      it { should have_db_index(:user_id) }
    end
  end

  # Making sure relations are set in the model
  describe 'relations' do
    it { should belong_to(:user) }
    it { should have_many(:comments) }
  end

  # Testing validations
  describe 'validations' do
    it { should validate_presence_of(:title) }
    it { should validate_presence_of(:description) }
    it { should validate_presence_of(:user) }

    it { should validate_uniqueness_of(:title) }
  end

  # Testing enums
  describe 'enumerated attributes' do
    it { should define_enum_for(:status).with_values([:added, :verified, :published]) }
  end

  # Model instance methods like the one below better to be located in a decorator or a presenter,
  # but for the sake of this example this instance method is kept in the model.
  describe '#short_description' do
    let(:article) { described_class.new(description: description) }
    let(:description) do
      'Such model instance method better to be located in a decorator or a presenter, '\
      'but for the sake of this example it is kept in the model.'
    end
    let(:short_description) { 'Model instance methods b...' }

    it 'returns short description' do
      expect(article.short_description).to eq(short_description)
    end
  end
end
```

##### Controller tests

```ruby
# app/controllers/interviews_controller.rb
class InterviewsController < ApplicationController
  before_action :authenticate_user!, only: [:edit, :user_interview, :update]

  def index
    @facade = IndexFacade.new(params, @search)
  end

  def show
    @facade = ShowFacade.new(params)
  end

  def new
    @facade = NewFacade.new(current_user.id)
  end

  def create
    @facade = CreateFacade.new(params, current_user.id)

    if @facade.save
      flash[:success] =
        "Whoa, your interview has been created. \
        It just needs a review before going public."
      redirect_to user_interviews_path
    else
      render 'new'
    end
  end

  def edit
    @facade = EditFacade.new(params[:id], current_user.id)

    if @facade.authorized?
      render 'edit'
    else
      flash[:notice] = 'Access denied as you are not an owner of this interview'
      redirect_to interview_path
    end
  end

  def update
    @facade = UpdateFacade.new(params)

    if @facade.save
      flash[:success] = 'The interview has been updated'
      redirect_to user_interviews_path
    else
      render 'edit'
    end
  end
end
```

```ruby
# spec/controllers/interviews_controller_spec.rb
require 'rails_helper'

RSpec.describe InterviewsController, type: :controller do
  let(:user) { create(:user) }

  before do
    allow(klass).to receive_messages(new: facade)
  end

  describe 'GET #index' do
    let(:klass) { described_class::IndexFacade }
    let(:facade) { instance_double(klass) }

    context 'without a search query' do
      it 'renders index action' do
        get :index

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:ok)
        expect(klass).to have_received(:new).with(anything, anything)
      end
    end

    context 'with a search query' do
      let(:params) do
        {
          q: {
            title_or_description_or_answers_content_cont: 'yellow'
          }
        }
      end

      it 'renders index action' do
        post :index, params: params

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:ok)
        expect(klass).to have_received(:new).with(anything, anything)
      end
    end
  end

  describe 'GET #show' do
    let(:klass) { described_class::ShowFacade }
    let(:facade) { instance_double(klass) }
    let(:params) { { id: 4 } }

    it 'renders show action' do
      get :show, params: params

      expect(assigns(:facade)).to eq facade
      expect(response).to have_http_status(:ok)
      expect(klass).to have_received(:new).with(anything)
    end
  end

  describe 'GET #new' do
    let(:klass) { described_class::NewFacade }
    let(:facade) { instance_double(klass) }

    before do
      sign_in user
    end

    it 'renders new action' do
      get :new

      expect(assigns(:facade)).to eq facade
      expect(response).to have_http_status(:ok)
      expect(klass).to have_received(:new).with(user.id)
    end
  end

  describe 'POST #create' do
    let(:klass) { described_class::CreateFacade }
    let(:facade) { instance_double(klass) }
    let(:params) { { title: 'title' } }

    before do
      allow(facade).to receive_messages(save: save)

      sign_in user
    end

    context 'with valid attributes' do
      let(:save) { true }

      it 'redirects to user interviews' do
        post :create, params: params

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:redirect)
        expect(klass)
          .to have_received(:new)
          .with(hash_including('title'), user.id)
      end
    end

    context 'with invalid attributes' do
      let(:save) { false }

      it 'renders new action' do
        post :create, params: params

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:ok)
        expect(klass)
          .to have_received(:new)
          .with(hash_including('title'), user.id)
      end
    end
  end

  describe 'GET #edit' do
    let(:klass) { described_class::EditFacade }
    let(:facade) { instance_double(klass) }

    before do
      allow(facade).to receive_messages(authorized?: authorized?)

      sign_in user
    end

    context 'when authorized to edit' do
      let(:authorized?) { true }

      it 'renders edit action' do
        get :edit, params: { id: '2' }

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:ok)
        expect(klass).to have_received(:new).with('2', user.id)
      end
    end

    context 'when NOT authorized to edit' do
      let(:authorized?) { false }

      it 'renders edit action' do
        get :edit, params: { id: '2' }

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:redirect)
        expect(klass).to have_received(:new).with('2', user.id)
      end
    end
  end

  describe 'PATCH #update' do
    let(:klass) { described_class::UpdateFacade }
    let(:facade) { instance_double(klass) }

    let(:params) do
      {
        id: '2',
        interview: {
          description: 'updated'
        }
      }
    end

    before do
      allow(facade).to receive_messages(save: save)

      sign_in user
    end

    context 'with valid attributes' do
      let(:save) { true }

      it 'redirects to user interviews' do
        patch :update, params: params

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:redirect)
        expect(klass).to have_received(:new).with(anything)
      end
    end

    context 'with invalid attributes' do
      let(:save) { false }

      it 'renders edit action' do
        patch :update, params: params

        expect(assigns(:facade)).to eq facade
        expect(response).to have_http_status(:ok)
        expect(klass).to have_received(:new).with(anything)
      end
    end
  end
end
```

##### Decorator tests

```ruby
# app/decorators/interview_decorator.rb
class InterviewDecorator < BaseDecorator
  def comments
    @comments ||= CommentDecorator.decorate_collection(object.comments)
  end

  def publication_status
    return "Published on #{published_at}" if published?

    'Unpublished'
  end

  def published_at
    created_at.strftime('%A, %B %e, %Y')
  end
end
```

```ruby
# spec/decorators/interview_decorator_spec.rb
require 'rails_helper'

RSpec.describe InterviewDecorator do
  let(:decorated_interview) { described_class.new(interview) }
  let(:published?) { true }

  let(:interview) do
    instance_double(
      Interview,
      published?: published?,
      created_at: Time.zone.parse('Wed, 17 Apr 2019 11:00:00'),
      comments: :comments,
    )
  end

  describe '#comments' do
    before do
      allow(CommentDecorator).to receive_messages(decorate_collection: :decorated_collection)
    end

    it 'decorates interview comments' do
      decorated_interview.comments

      expect(CommentDecorator).to have_received(:decorate_collection).with(:comments)
    end
  end

  describe '#publication_status' do
    context 'when an interview has been published' do
      it 'returns correct publication status' do
        expect(decorated_interview.publication_status).to eq('Published on Wednesday, April 17, 2019')
      end
    end

    context 'when an interview has not been published' do
      let(:published?) { false }

      it 'returns correct publication status' do
        expect(decorated_interview.publication_status).to eq 'Unpublished'
      end
    end
  end

  describe '#published_at' do
    it 'returns correct published_at date' do
      expect(decorated_interview.published_at).to eq 'Wednesday, April 17, 2019'
    end
  end
end

```

##### Scope tests

```ruby
# app/models/interview/published_scope.rb
class Interview::PublishedScope
  def call
    module_parent.where(published: true)
  end

  delegate :module_parent, to: :class
end
```

```ruby
require 'rails_helper'

RSpec.describe Interview::PublishedScope do
  describe '#call' do
    let(:scope) { described_class.new }
    let(:sql) { scope.call.to_sql }

    it 'executes a valid SQL' do
      expect { scope.call.load }.not_to raise_error
    end

    it 'generates an expected SQL' do
      expect(sql.squish).to include(<<-SQL.squish), sql
        SELECT "interviews".* FROM "interviews"
        WHERE "interviews"."published" = TRUE
      SQL
    end
  end
end

```

##### Module tests

```ruby
RSpec.describe ModuleName do
  let(:test_class) do
    Class.new do
      include ModuleName

      def some_method
        # if needed
      end
    end
  end

  describe '#module_method' do
    it { expect(test_class.new.module_method).to eq(:some_result) }
  end
end
```

## Rails tips

### Rails commands

```ruby
# App initialization
rails new app_name -T -d=postgresql # generate an app without Test::Unit, using postgresql

# Generators
rails g model Article title:string points:integer description:text user:references # generate a new model with provided attributes
rails g controller ControllerName # generate a controller. ControllerName should be plural
rails g migration AddSlugToArticles # generate a new migration with a unique timestamp

# Working with migrations
rails db:rollback STEP=N # rollback N migrations
rails db:migrate VERSION=20190404124512 # rollback to and including a provided migration version
rails db:migrate:down VERSION=20190404124512 # rollback just a specific migration without affecting other migrations
rails db:migrate:status # display status of all migrations, ids and names

# Working with gems
bundle outdated # list gems that have a newer version available

# Routes
rails routes | grep Keyword # simplifies search of only specific routes
```

### Gem versions in Gemfile

Gem versions and how exactly they are specified in the Gemfile results in what you get after running `bundle update`.

Provide just a gem name to allow any gem version:

```ruby
gem 'devise'
```

Ruby gems have three-level versions - major, minor and patch. Specify an exact gem version with:

```ruby
gem 'devise', '4.3.0' # 4 - major, 3 - minor, 0 - patch
```

Use comparisons operators when only a certain range of a gem version is required:

```ruby
gem 'devise', '>= 4.3.0', '< 4.5.0' # means that versions greater than or equal to 4.3.0 yet lower than 4.5.0 are acceptable
```

The shortest yet the least obvious strategy is using a pessimistic locking operator `~>` which is a shortcut for `>= and <`. How many levels of a provided gem version are used defines behavior of `~>`:

When all three version levels `4.3.0 - major.minor.patch` are provided, gem updates are allowed within a patch version range:

```ruby
gem 'devise', '~> 4.3.2' # here a gem is allowed to be updated between >= 4.3.2 and < 4.4
```

Remove a patch version `4.3 instead of 4.3.2` to allow updates within a minor version range:

```ruby
gem 'devise', '~> 4.3' # here the pessimistic operator ensures gem updates within >= 4.3 and < 5.0 range
```

Specifying patch-level versions like `~> 4.3.1` in your Gemfile using the pessimistic locking operator `~>` ensures that gem fixes are automatically provided on `bundle update`, but major potentially breaking changes are not introduced. The downside of this is that manual bumping of gem versions is required to stay up to date with updates.

### How to update gems

- `bundle outdated` - to see the list of outdated gems.
- `bundle update` - try to update all gems if such update is possible/acceptable.
- `bundle update devise rspec` - or update specific gems by listing their names.
- make sure that your tests are passing after running the update.
- `git checkout Gemfile.lock` if tests has failed and it is hard to figure out which gem was the reason for that. Proceed with updates per a gem.
- otherwise, downgrade a specific gem by editing `Gemfile.lock`. The practice of updating `Gemfile.lock` is not suggested, yet doable.
- run `bundle outdated` again to see which gems couldn't be updated. Look into the source of each gem (issues, PRs) to identify the status of an issue. A fix for example could already be on the master.
- if an issue has been already fixed on the master, yet without a release, you could use git as a source instead of rubygems -> `gem 'pg', git: 'https://github.com/ged/ruby-pg'`

### Rails ActiveRecord model structure

```ruby
class Model < ApplicationRecord
  include # ......................include instance methods from a module
  extend # ......................extend with class methods from a module
  CONSTANT_NAME = 10 # ........................................constants
  devise :registerable, :confirmable # ....................devise macros
  belongs_to; has_many # ...................................associations
  validates :name, presence: true # .........................validations
  before_save :method_name # ..................................callbacks
  accepts_nested_attributes_for :name # ...accepts_nested_attributes_for
  scope :name, where(color: 'green') # ...........................scopes
  delegate # ................................................delegations
  def self.method_name; end # .............................class methods
  def method_name; end # ........................public instance methods
  private # .................................everything below is private
end
```

### DB migrations
```ruby
add_column :interviews, :type, :integer, default: 0, null: false # add a column for `type` enum
add_column :interviews, :format, :string # add a string column
add_column :interviews, :extra, :jsonb, default: {} # add a jsonb column
add_column :interviews, :rate, :decimal, precision: 8, scale: 2 # add a decimal column
rename_column :interviews, :type, :kind # rename type to kind
change_column_default :interviews, :type, 2 # change the type default to 2
change_column_null :interviews, :editor_id, false # add `null: false` to editor_id
add_index :interviews, :uuid, unique: true # add a unique index on the uuid column
add_index :interviews, [:type, :country_id], unique: true # add an index on two columns
add_index :interviews, [:type, :user_id, :country_id, :slug], unique: true, name: :index_on_
remove_index :interviews, :uuid
remove_index :interviews, [:type, :country_id]
remove_index :interviews, name: "index_on_ ..."
add_reference :interviews, :country, foreign_key: true, null: false, index: true # add country_id to interviews table with the FK constraint
# The examples below require verification
# when an association name is different from a table name
# adding a correct foreign key might require an extra step:
add_reference :interviews, :editor, references: :users, index: true # step 1
add_foreign_key :interviews, :users, column: :editor_id, primary_key: :id # step 2
```

When planning to add a null constraint, first add a field, then make sure all records in production have a value other than null, and only then add a null constraint in a new migration file.

### Multi-line strings

```ruby
  'a really long string ' \
  'on multiple lines'
  # => "a really long string on multiple lines"

  %{
    a really long string
    on multiple lines
  }.squish
  # => "a really long string on multiple lines"

  <<-EOS.squish
    a really long string
    on multiple lines
  EOS
  # => "a really long string on multiple lines"
```

### Different flavors of scopes

```ruby
# app/models/interview.rb
class Interview < ApplicationRecord
  # flavor 1
  scope :published, -> { where(published: true) }

  # flavor 2 (see PublishedScope below)
  scope :published, PublishedScope

  # flavor 3
  def self.published
    where(published: true)
  end
end

# app/models/interview/published_scope.rb
class Interview::PublishedScope
  def call
    module_parent.where(published: true)
  end

  delegate :module_parent, to: :class
end
```

## Ruby tips

### The method_missing method

`method_missing` allows to handle calling a method that does not exist. When added to a class, it gives control of what should happen with a missing method instead of raising `NoMethodError`. One of the options is to delegate message call to another object:

```ruby
class Presenter
  def initialize(object)
    @object = object
  end

  private

  def method_missing(name, *args, &block)
    @object.send(name, *args, &block) # this delegates the name method to another object where the name method is implemented
  end
end

Presenter.new(:object).length # This returns the length of :object symbol
# => 6
Presenter.new(:object).to_s # Be careful with methods like `to_s` inherited from Object
# => #<Presenter:0x000...>
```

### Initialize - self.name vs @name

There are two ways to set instance attributes:

```ruby
def initialize(name)
  self.name = name # this makes a method call to name=
end

private

attr_accessor :name # this is required

def name=(value) # an optional custom writer; if used, attr_accessor should be replaced with attr_reader
  @name ||= # ...
end
```

Pros:

- Better error catching: it is possible to access an instance variable without first defining it. There is no raised exception when accessing an undefined instance variable and the default value is `nil`. This may lead to difficult-to-track errors. When using `self.` an error message (e.g. `NameError`) is always explicit, pointing to an attribute which has raised an error.
- Allows to write a custom writer, which is run on object initialization.
- Allows to use `klass.send(:attr_name)`.

```ruby
def initialize(name)
  @name = name # this just sets a private instance variable
end

private

attr_reader :name # this is optional; allows to access an attribute without `@` sign
```

Pros:

- Doesn't require `attr_accessor` to be defined.
- With the private `attr_reader` doesn't require `attr_writer` as `self.name` does, yet still allows to access an attribute with a getter, same as in the case of `self.name`.

Also remember that using `attr_accessor` (same for `attr_reader` and `attr_writer`) instead of writing a getter/setter on your own is faster due to its implementation in C.

Using an instance variable allows to skip `attr_reader`:

```ruby
  def initialize(connection)
    @connection = connection # this just sets a private instance variable
  end

  def method_name
    Connection.find(@connection) # without an attr_reader
  end
```

Pros:

- Doesn't require `attr_reader` to be defined - less code to write

Cons:

- A typo or an unexpected `nil` value may lead to difficult-to-track errors

### Positional and keyword parameters

```ruby
  def print(name) # positional parameter, required
  def print(name = 'John') # positional parameter with a default value
  def print(name:) # keyword parameter, required
  def print(name: 'John') # keyword parameter with a default value
    p name
  end
```

Positional parameters:

- The shortest option
- Require to preserve the order. If the order of the parameters is changed in the original implementation, this order should also be changed everywhere the method is called
- The need to look at the implementation to know more about each parameter

Keyword parameters:

- Verbose
- No coupling to the order of the parameters
- The meaning of the parameters is known without looking at the implementation
- Explicit `ArgumentError` pointing to a missing argument

### Alternative to string interpolation

```ruby
  "coordinates: #{x}, #{y}" # string interpolation
  format('coordinates: %<x>s, %<y>s', x: x, y: y) # Kernel.format
```

### Set object state with a block

```ruby
class Interview
  attr_accessor :title

  def initialize(title = '')
    @title = title
    yield self if block_given?
  end
end

# using a constructor
Interview.new(title = 'Ruby')
# => #<Interview:0x00007f9f79191d90 @title="Ruby">

# using a variable
interview = Interview.new
interview.title = 'Ruby'
interview
# => #<Interview:0x00007fe76a118e98 @title="Ruby">

# using a block
Interview.new { |interview| interview.title = 'Ruby' }
# => #<Interview:0x00007fe76a0f21a8 @title="Ruby">

# using #tap, which allows to omit the interview variable
Interview.new.tap { |interview| interview.title = 'Ruby' }
# => #<Interview:0x00007fe76a1007a8 @title="Ruby">

# using #tap for conditional assignments
[].tap do |list|
  if something?
    list << Builder.call(@article_params)
  end

  if nothing?
    list << @default_article
  end
end
```

### Product of two arrays

```ruby
  [25, 100, 2].product([4, 7]) # => [[25, 4], [25, 7], [100, 4], [100, 7], [2, 4], [2, 7]]
  [25, 100, 2].product([]) # => []
```

### Array generator

```ruby
  Array.new(5) { Random.rand(1..10) } # => [9, 4, 8, 6, 10]
```

### XOR operator
`5 ^ 5 = 0` and `10 ^ 10 = 0`, so the XOR operator can be used for finding a single unique integer:

```ruby
[7, 5, 10, 5, 10].reduce(&:^) # => 7
```

### Timestamp generator
```ruby
Time.now.utc.strftime('%Y%m%d%H%M%S') # => "20191117151018"
```

### Memoization

The most often memoization to avoid computation of the same value multiple times is the following:

```ruby
  def articles
    @articles ||= # ...some computation
  end
```

Where `@articles ||= something` is equivalent to `@articles || @articles = something`. And this means that `@articles` is only set if `@articles` is `falsey` - either `nil` or `false` - meaning that a `falsey` result of a computation would not be memoized and would make that computation run again on every call. Therefore, if a `falsey` value is one of the expected results, the `falsey` protected memoization should be used:

```ruby
def articles
  return @articles if defined?(@articles)
  @articles = # ...some computation
end
```

Having test mocks with the number of calls of the memoized computation would ensure that the memoization is working as expected.

### One-liner nested hash creation
```ruby
Hash.new { |hash, key| hash[key] = Hash.new(&hash.default_proc) }
```

### One-liner hash creation with array as default value
```ruby
Hash.new { |hash, key| hash[key] = [] }
```

### One-liner hash creation for counting occurrences of a certain value
```ruby
articles.each_with_object(Hash.new(0)) { |article, memo| memo[article.type] += 1 }
```

### Gem versioning

Given that a ruby gem version has the format of `MAJOR.MINOR.PATCH` (e.g. `3.5.1`), when releasing a new version bump the corresponding version depending on changes:

- `MAJOR` [`3.5.1` -> `4.0.0`] when making incompatible API changes
- `MINOR` [`3.5.1` -> `3.6.0`] when adding functionality in a backwards compatible manner
- `PATCH` [`3.5.1` -> `3.5.2`] when adding backwards compatible bug fixes

### Decorator Stack


```ruby
class DecoratorStack
  def initialize
    @decorators = []
    yield(self) if block_given?
  end

  def add(klass, *args)
    @decorators << [klass, args]
  end

  def decorate(object)
    @decorators.each do |(klass, args)|
      object = klass.decorate(object, *args)
    end

    object
  end
end
```

```ruby
decorator_stack = DecoratorStack.new do |stack|
  stack.add(Decorator1, optional_parameter_name: optional_parameter_value)
  stack.add(Decorator2)
  ...
  stack.add(DecoratorN)
end

decorator_stack.decorate(some_object)
```

```ruby
describe DecoratorStack do
  subject(:stack) { described_class.new }

  describe '#decorate' do
    context 'with no decorators' do
      it { expect(stack.decorate('object')).to eq('object') }
    end

    context 'with a single decorator' do
      let(:decorator) do
        Class.new do
          def self.decorate(object)
            "decorated-#{object}"
          end
        end
      end

      before { stack.add(decorator) }

      it { expect(stack.decorate('object')).to eq 'decorated-object' }
    end

    context 'with multiple decorators' do
      let(:first_decorator) do
        Class.new do
          def self.decorate(object)
            "#{object}-decorated"
          end
        end
      end

      let(:second_decorator) do
        Class.new do
          def self.decorate(object)
            "decorated-#{object}"
          end
        end
      end

      before do
        stack.add(second_decorator)
        stack.add(first_decorator)
      end

      it { expect(stack.decorate('object')).to eq 'decorated-object-decorated' }
    end
  end
end
```

### Memory cache example

```ruby
class MemoryCache
  EXPIRATION_IN_SECONDS = 60 * 60

  attr_reader :store

  def initialize(options = {})
    @store = {}
    @key_access = {}
    @mutex = Mutex.new
    @expires_in = options&.fetch(:expires_in, EXPIRATION_IN_SECONDS)
  end

  def read(key)
    value = @store[key]
    return unless value

    if @key_access[key] > Time.now
      value
    else
      delete(key)
      nil
    end
  end

  def write(key, entry, expires_in: nil)
    @mutex.synchronize do
      return entry if @store[key]

      @key_access[key] = Time.now + (expires_in || @expires_in)
      @store[key] = entry
    end
  end

  def delete(key)
    @mutex.synchronize do
      @key_access.delete(key)
      @store.delete(key)
    end
  end
end
```

```ruby
describe MemoryCache do
  let(:write_cache) do
    Timecop.freeze(Time.parse('2020-05-14 15:30')) do
      cache.write(:key, :cached_entry, expires_in: 3600)
    end
  end
  let(:cached_entry) { Timecop.freeze(cache_read_time) { cache.read(:key) } }
  let(:cache_read_time) { Time.parse('2020-05-14 16:31') }

  describe '#read' do
    let(:cache) { described_class.new }

    context 'when key is not cached' do
      it { expect(cache.read(:key)).to be_nil }
    end

    context 'when key is cached' do
      context 'when key is not expired' do
        before { cache.write(:key, :cached_entry) }

        it { expect(cache.read(:key)).to eq(:cached_entry) }
      end

      context 'when key is expired' do
        before { write_cache }

        it { expect(cached_entry).to be_nil }
      end
    end
  end

  describe '#write' do
    let(:cache) { described_class.new(options) }

    context 'when global expires_in is not defined' do
      let(:options) { nil }

      before { write_cache }

      it { expect(cached_entry).to be_nil }
    end

    context 'when global expires_in is defined' do
      context 'when write options are defined' do
        let(:options) { {expires_in: 7200} }

        before { write_cache }

        context 'when cache is not expired' do
          let(:cache_read_time) { Time.parse('2020-05-14 16:20') }

          it { expect(cached_entry).to eq :cached_entry }
        end

        context 'when cache is expired' do
          it { expect(cached_entry).to be_nil }
        end
      end

      context 'when write options are not defined' do
        let(:options) { {expires_in: 3600} }

        before do
          Timecop.freeze(Time.parse('2020-05-14 15:30')) do
            cache.write(:key, :cached_entry)
          end
        end

        context 'when cache is not expired' do
          let(:cache_read_time) { Time.parse('2020-05-14 16:20') }

          it { expect(cached_entry).to eq :cached_entry }
        end

        context 'when cache is expired' do
          it { expect(cached_entry).to be_nil }
        end
      end
    end
  end
end
```


### Splatting with destructuring assignment

```ruby
*old, new, current = [1, 2, 3, 4, 5, 6, 7]
old # => [1, 2, 3, 4, 5]
new # => 6
current # => 7

new, *old, current = [1, 2, 3, 4, 5, 6, 7]
new # => 1
old # => [2, 3, 4, 5, 6]
current # => 7
```

### Save console output to a file

```ruby
$stdout = File.new('file.rb', 'w')
$stdout.sync = true
$stdout = STDOUT
puts ['to', 'save,', 'object', 'to', 'file']
```


## Ruby gems

![cover](https://raw.githubusercontent.com/xkwd/learning-ruby-rails/master/gems.png)

### Bundler
```shell
gem list bundler # check installed bundler versions
gem install bundler -v "2.0.2" -N # install a specific version of bundler without documentation
bundle _2.0.2_ install # run 'bundle install' with a specific version of Bundler
EDITOR=atom bundle open <gem_name> # open a gem in a specified editor
```

If you happen to have multiple default bundler versions installed via the `--default` command, the way to remove multiple defaults is to re-install specific versions again:

```shell
bundler (default: 2.4.1, default: 2.2.27, default: 2.1.4)
gem install bundler -v 2.2.27
gem install bundler -v 2.4.1
bundler (2.4.1, 2.2.27, default: 2.1.4)
```

### Savon
```ruby
client = Savon.client(
  wsdl: 'http://external-test.example.com/webservices/searchservice?wsdl',
  endpoint: 'http://external-test.example.com/webservices/searchservice',
  namespace: 'http://example.com/webservices/external',
  namespaces: {
    'xmlns:db'   => 'http://example.com/webservices/external/data/db',
    'xmlns:request' => 'http://example.com/webservices/external/search/execute'
  },
  log: true,
  log_level: :info,
  basic_auth: ['username', 'password'],
  headers: {
    'User-Agent' => 'XMLClient',
    'Content-Type' => 'application/text-plain',
    'Connection' => 'close'
  },
  open_timeout: 20,
  read_timeout: 20
)

client.operations # => [:operation1, :operaton2, :operation3]
client.call(:operation1, message: message)
puts client.build_request(:operation1, message: message).body
client.wsdl.endpoint = '214.140.10.20'

message = {
  'Header' => {
    :'@username' => 'username',
    :'@timestamp' => 'timestamp'
  },
  'RequestData' => {
    :'@from' => 'from',
    :'@to' => 'to',
    :'@count' => '2',
    'Details' => {
      :'@date' => 'date'
    },
    'Items' => {
      'db:Inventory' => {
        :'@sku' => '1440'
      }
    }
  }
}
```

### similar_text

```ruby
'Learning Ruby'.similar('Ruby tips') #=> 36.36363636363637
```

### Kdtree

[Kdtree](https://github.com/gurgeous/kdtree) - the fastest Ruby gem to find the nearest neighbor or a set of the nearest neighbors using latitude and longitude of thousands of locations.

```ruby
locations = [
  {longitude: -39.49, latitude: -15.42}, # location with index [0]
  {longitude: -37.49, latitude: -12.42} # location with index [1]
]
mapped_locations = locations.map.with_index do |location, index|
  [location[:longitude], location[:latitude], index]
end
kd_tree = Kdtree.new(mapped_locations)
target_location = {longitude: -40.02, latitude: -15.10}
number_of_nearest_locations = 1
nearest_location_index = kd_tree.nearestk(
  target_location[:longitude], target_location[:latitude], number_of_nearest_locations
) # nearest_location_index => [0]
```

### PgSearch

[PgSearch](https://github.com/Casecommons/pg_search) - allows to search with PostgreSQL's full text search.

```ruby
class Car < ApplicationRecord
  include PgSearch::Model

  pg_search_scope(
    :find_model,
    against: :model,
    using: {
      tsearch: { any_word: true, normalization: 2 },
      trigram: { word_similarity: true, threshold: 0.3 }
    },
  )
end

class AddTrigramIndexToCars < ActiveRecord::Migration[5.2]
  def change
    enable_extension :pg_trgm
    add_index :cars, :model, using: :gin, opclass: { model: :gin_trgm_ops }
  end
end

Car.find_model('Cx-5') # => 'CX5'
```

### dry-struct

A gem for defining typed struct classes:

```ruby
module Types
  include Dry.Types()
end

class Location < Dry::Struct
  attribute :id, Types::Strict::Integer
  attribute :name, Types::Strict::String
  attribute :longitude, Types::Strict::Float.optional.default(nil)
  attribute :latitude, Types::Strict::Float.optional.default(nil)
  attribute? :active_since, Types::Strict::DateTime
  attribute :indexes, Types::Array.of(Types::Coercible::String).default { [] }
end

Location.new(id: 2, name: 'default', indexes: [:a, :b])
# => #<Location id=2 name="default" longitude=nil latitude=nil active_since=nil indexes=["a", "b"]>
```

## SQL

### Query example

```ruby
class Report
  Stats = Struct.new(
    :created_by_id,
    :created_by_name,
    :created_by_role,
    :source,
    :appointments_total,
    :appointments_average_value,
    :appointments_total_showed,
    :appointments_average_value_showed,
    :appointments_total_not_showed,
    :appointments_average_value_not_showed,
    :appointments_total_canceled,
    :appointments_average_value_canceled,
    :appointments_total_showed_extras,
    keyword_init: true,
  )

  def initialize(agency, from, to)
    @agency = agency
    @from = from.in_time_zone(agency.time_zone).beginning_of_day
    @to = to.in_time_zone(agency.time_zone).end_of_day
  end

  def call
    ActiveRecord::Base
      .connection
      .execute(params)
      .map { |row| Stats.new(row) }
  end

  private

  def params
    ApplicationRecord.sanitize_sql(
      [
        query,
        {
          agency_id: @agency.id,
          from: @from,
          to: @to
        }
      ],
    )
  end

  def query
    <<~SQL.squish
      WITH
        all_agencies AS (
          SELECT
            a.id
          FROM
            agencies a
          WHERE
            a.id = :agency_id
        ),
        appointment_items_prefilter as (
          SELECT
            i.id,
            i.item_id,
            i.priceable_id,
            i.priceable_type,
            i.labor_price,
            i.parts_price
          FROM
            items i
              LEFT JOIN appointments a
                ON a.id = i.priceable_id AND i.priceable_type = 'Appointment' AND a.agency_id = :agency_id
          WHERE
            a.appointment_datetime BETWEEN :from AND :to
        ),
        items_prefilter as (
          SELECT
            ai.id,
            ai.priceable_id,
            ai.priceable_type,
            ai.labor_price,
            ai.parts_price,
            COALESCE(i.extras, '{}') AS extras
          FROM
            appointment_items_prefilter ai
              LEFT JOIN items i
                ON ai.item_id = i.id
        ),
        appointments_prefilter as (
          SELECT
            a.id
              AS appointment_id,
            SUM(COALESCE(ip.labor_price, 0.0) + COALESCE(ip.parts_price, 0.0))
              AS price,
            COUNT(ip.extras) FILTER ( WHERE ip.extras != '{}' AND a.aasm_state NOT IN ('canceled', 'not_shown'))
              AS extras,
            a.aasm_state,
            a.created_by_id,
            e.name
              AS created_by_name,
            e.role
              AS created_by_role
          FROM
            appointments a
              LEFT JOIN items_prefilter ip
                ON a.id = ip.priceable_id AND ip.priceable_type = 'Appointment'
              LEFT JOIN users
                ON users.id = a.created_by_id
              LEFT JOIN employments e
                ON e.agency_id = a.agency_id AND e.email = users.email
              INNER JOIN all_agencies aa
                ON a.agency_id = aa.id
          WHERE
            a.appointment_datetime BETWEEN :from AND :to
          GROUP BY
            a.id, e.name, e.role
        ),
        aggregated_appointments AS (
          SELECT
            ap.created_by_id,
            ap.created_by_name,
            ap.created_by_role,
            COUNT(*)
              AS total,
            COUNT(*) FILTER (WHERE ap.aasm_state NOT IN ('canceled', 'not_shown'))
              AS total_showed,
            COUNT(*) FILTER (WHERE ap.aasm_state = 'not_shown')
              AS total_not_showed,
            COUNT(*) FILTER (WHERE ap.aasm_state = 'canceled')
              AS total_canceled,
            SUM(ap.extras)::integer AS total_extras,
            COALESCE(AVG(ap.price), 0.0)
              AS average_value,
            COALESCE(AVG(ap.price) FILTER (WHERE ap.aasm_state NOT IN ('canceled', 'not_shown')), 0.0)
              AS average_value_showed,
            COALESCE(AVG(ap.price) FILTER (WHERE ap.aasm_state = 'not_shown'), 0.0)
              AS average_value_not_showed,
            COALESCE(AVG(ap.price) FILTER (WHERE ap.aasm_state = 'canceled'), 0.0)
              AS average_value_canceled
          FROM
            appointments_prefilter ap
          GROUP BY
            ap.created_by_id, ap.created_by_name, ap.created_by_role
        )
      SELECT
        aa.created_by_id,
        aa.created_by_name,
        aa.created_by_role,
        'scheduler' AS source,
        COALESCE(aa.total, 0)                                 AS appointments_total,
        ROUND(COALESCE(aa.average_value, 0.0), 2)             AS appointments_average_value,
        COALESCE(aa.total_showed, 0)                          AS appointments_total_showed,
        ROUND(COALESCE(aa.average_value_showed, 0.0), 2)      AS appointments_average_value_showed,
        COALESCE(aa.total_not_showed, 0)                      AS appointments_total_not_showed,
        ROUND(COALESCE(aa.average_value_not_showed, 0.0), 2)  AS appointments_average_value_not_showed,
        COALESCE(aa.total_canceled, 0)                        AS appointments_total_canceled,
        ROUND(COALESCE(aa.average_value_canceled, 0.0), 2)    AS appointments_average_value_canceled,
        COALESCE(aa.total_extras, 0)                          AS appointments_total_showed_extras
      FROM
        aggregated_appointments aa
      WHERE
        aa.created_by_id IS NOT NULL and aa.created_by_role IS NOT NULL
    SQL
  end
end
```

## Debugging

### How to debug a Ruby gem

`Bundler` offers a very convenient way to open a Ruby gem in an editor and be able to see the gem source code. Adding `binding.pry` where needed in the source code is a convenient way to debug:

```shell
EDITOR=atom bundle open <gem_name>
```

## Glossary

### Object state

- The convention is to set the state within a constructor (the initialize method)
- Persists across object's lifetime
- Is defined with an instance variable (`@name`) or a method (`self.name`)
- May change from the moment of object's initialization

### Idempotent method

- Is when the result of a method call is independent of the number of execution times
- Important when building a fault-tolerant API
- POST/PATCH are not idempotent HTTP methods

### Predicate method

- Syntactically ends with a question mark (?)
- Returns only a truthy result (`true`, `false`)
- Does not require prefixing: use `forbidden?`, not `is_forbidden?`
