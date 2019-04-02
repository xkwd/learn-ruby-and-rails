### Service Object

#### 1. Overview

If a Rails app is built by following the Rails Way, it is very likely to end up with fat models and bloated controllers, which in turn are difficult both to test and to be understood by new developers. This is a usual scenario when an app gets bigger than a typical CRUD app. Adding a layer of Service Objects (or simply services) is a way to deal with it.

Service Object is a Plain Old Ruby Object (PORO) and is a pattern allowing to extract business logic from Models and Controllers (in Model-View-Controller pattern) to keep them slim and thus readable. A single Service Object encapsulates a single process of the business logic. Therefore, it becomes easier to write, test and modify that specific business logic.

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

- Naming services after **commands**: **Create**Report, **Update**RemotePrice, **Fetch**FinalResults.
- Using '**or**', '**er**' endings for the last noun in a service name: ReportCreat**or**, RemotePriceUpdat**er**, FinalResultsFetch**er**.
- Appending '**Service**' word to a name based on logic description of a service: ReportCreation**Service**, RemotePriceUpdating**Service**, FinalResultsFetching**Service**.

Service Object always has to have a single public method. Naming options for it are:

- `call`
- `perform`
- `run`
- `execute`

Sticking with the convention simplifies understanding of class responsibility across an app.

#### 5. Directory

Suggested directory for services is **app/services**.

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
