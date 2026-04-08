# Backend Patterns

## Command Pattern (Primary API Pattern)

Used for all `api/v1/` controllers. Do not bypass this in favour of direct service calls in controllers.

### Base classes (`lib/core/commands/`)
- `CreateCommand`, `UpdateCommand`, `DeleteCommand`, `FindCommand`, `SearchCommand`
- All via `BaseFlowStarterCommand`

### Auto-wiring with `GenericActionBehavior`
```ruby
include Core::Api::GenericActionBehavior
with_command_namespace "Api::Products"
with_command_call_actions :index, :show, :create, :update, :destroy
```

Auto-wires:
- `index` → `SearchCommand`
- `show` → `FindCommand`
- `create` → `CreateCommand`
- `update` → `UpdateCommand`
- `destroy` → `DeleteCommand`

### Domain layout (`app/commands/api/<domain>/`)
```
create_command.rb
update_command.rb
find_command.rb
search_command.rb
parameters_handle_on_creation_command.rb    # param whitelist/shape only
parameters_handle_on_update_command.rb
delete_command.rb                           (if deletable)
steps/                                      # custom steps only
```

### Command Context
Commands receive:
- `Core::Commands::Context` — holds `current_site`, `current_user`
- `resource_params` / `scope_params`

They return a response object checked with `respond_with_success` / error variants.

### Custom Steps
```ruby
include CommandsFlow::Helpers::StepsHelpers
attr_command_reader :product
on_success_assign_data_to :product

def call
  # ... logic
  :success  # always return symbol, never raise
end
```

---

## Service Pattern (`lib/service.rb`)

For business logic that doesn't fit inside a Command step.

```ruby
class MyService
  include Service

  def call(...)
    # ...
    respond_with_success(data: result)
  end
end

MyService.call(...)
```

---

## Entity Pattern (`lib/core/entity/base.rb`)

Value objects passed between layers — not persisted.

```ruby
class MyEntity < Core::Entity::Base
  with_attributes :name, :amount
end
```

---

## Query Pattern (`app/queries/`)

Complex AR scopes extracted from models and commands.

```ruby
class ProductSearchQuery
  include QueryBase

  def perform(site_id:, keyword:)
    Product.where(site_id: site_id).search(keyword)
  end
end
```

---

## Commands Flow (`lib/commands_flow/`)

Chainable step execution engine underpinning the command framework. Each step is a class with a `call` method. Steps declared and chained inside command definitions. Automatically handles success/failure routing between steps.

---

## Event-Driven (`app/event_driven/`)

Webhook event pub/sub system:
- `events/` — event definitions
- `adapters/` — outbound adapters per integration
- `publisher/` — publishes events
- `subscriber/` — subscribes and routes incoming webhooks
