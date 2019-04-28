![cover](https://raw.githubusercontent.com/xkwd/learning-ruby-rails/master/cover.png)

## Table of contents

- [Patterns](#patterns)
  - [Service Object](#service-object)
  - [Decorator](#decorator)
- [Development Tools](#development-tools)
  - [Mastering GIT](#mastering-git)
  - [Continuous Integration](#continuous-integration)
- [RSpec basics](#rspec-basics)
  - [Model tests](#3-model-tests)
- [Rails tips](#rails-tips)
  - [Rails commands](#rails-commands)

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


## Development Tools

### Mastering GIT

#### 1. Use GIT aliases

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

#### 2. Push force carefully

Use `git push --force-with-lease` instead of `git push --force`

A force push overwrites a remote branch with your local branch, regardless of the status of that remote branch. It is not safe and you might overwrite other developers commits. Force with lease gives you the flexibility to override new commits on your remote branch, whilst protecting your old commit history
read more how it works [here](http://weiqingtoh.github.io/force-with-lease/).

It is a long command to type, so create an alias for it ;)


#### 3. Stick to the general work flow

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

#### 4. Useful tips

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

### RSpec basics

#### 1. Overview

Unit tests (or specs) are used for testing of a single component (class or module). Only public methods should be tested, not private ones. Public methods do represent a public interface of a component.

Besides units tests there are integration tests. They are used when testing goes beyond the logic of a single component. For instance accessing a database already implies accessing another component, leading to an integration test.

Both unit and integration tests should cover happy path (e.g. with valid arguments), unhappy path (e.g. with invalid arguments) and edge cases (e.g. division by zero).

#### 2. RSpec vocabulary

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

#### 3. Model tests

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
