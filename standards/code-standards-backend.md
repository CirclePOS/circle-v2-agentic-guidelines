# Code Standards — Backend (Rails)

## Core Principle

**Business logic belongs in Commands and Services — never in models or controllers.**

---

## Layer Rules

### Models (`app/models/`)
Allowed: associations, scopes, validations, AASM state machines, `store` accessors, Elasticsearch index config, derived display/presentation values.

Not allowed: business logic, complex queries, external service calls.

Apply **"Tell don't ask"**: views call `address.country_representation`, not `ISO3166::Country[address.country]&.iso_short_name || address.country.presence`.

### Controllers (`app/controllers/`)
Pattern: authenticate → extract params → call command → render result. Nothing else.

Declare: `ACTIONS_WITH_READ_PERMISSION` / `ACTIONS_WITH_MANAGE_PERMISSION` per controller.

For `api/v1/` controllers — use `Core::Api::GenericActionBehavior`:
```ruby
include Core::Api::GenericActionBehavior
with_command_namespace "Api::Products"
with_command_call_actions :index, :show, :create, :update, :destroy
```
This auto-wires: `index` → `SearchCommand`, `show` → `FindCommand`, `create` → `CreateCommand`, `update` → `UpdateCommand`, `destroy` → `DeleteCommand`.

### Serializers (`app/serializers/`)
- Inherit from `Core::Serializers::ResourceSerializer`
- Declare fields: `serialize :field1, :field2`
- Do NOT use `ActiveModel::Serializers` or `jbuilder`

### Background Jobs (`app/workers/`, `app/jobs/`)
- Heavy logic delegated to a service or command — not written inline in the job

### AASM State Machines
- State transition side effects go in command steps, not AASM callbacks

---

## Command Pattern

### Domain Layout

Every resource domain under `app/commands/api/<domain>/` must contain:

```
create_command.rb
update_command.rb
find_command.rb
search_command.rb
parameters_handle_on_creation_command.rb    # param whitelist/shape only — no business logic
parameters_handle_on_update_command.rb
delete_command.rb                           (if deletable)
steps/                                      # custom steps only when base steps are insufficient
```

### Custom Steps
- Include `CommandsFlow::Helpers::StepsHelpers`
- Use `attr_command_reader` / `on_success_assign_data_to`
- Return `:success` or an error symbol — **never raise**
- Commands receive `Core::Commands::Context` (holds `current_site`, `current_user`) and `resource_params`/`scope_params`
- They execute a chain of steps and return a response checked with `respond_with_success` / error variants
- Specs in `spec/commands/` mirror `app/commands/`

---

## Core Infrastructure Patterns (`lib/`)

### Service (`lib/service.rb`)
Include `Service` mixin; invoke via `.call(...)`. Returns `respond_with_success` / `respond_with_failure`. Used for business logic outside the command layer.

### Entity (`lib/core/entity/base.rb`)
Inherit from `Core::Entity::Base` for value objects. Declare attributes with `with_attributes`. Used for structured data passed between layers — not persisted.

### Query (`app/queries/`)
Include `QueryBase`; implement `perform(**args)` returning an AR relation. Used to compose complex scopes called from commands or services.

### Commands Flow (`lib/commands_flow/`)
Chainable step execution engine underlying the command framework. Each step is a class with a `call` method; steps declared and chained inside command definitions.

---

## Rails Directory Reference

| Directory | Purpose |
|-----------|---------|
| `app/controllers/api/v1/` | REST API consumed by Vue3 apps |
| `app/controllers/backoffice/` | Legacy ERB backoffice controllers |
| `app/services/` | Business logic; autoloaded, call via `.call(...)` |
| `app/queries/` | Query objects using `QueryBase` mixin |
| `app/entities/` | Value objects and complex data structures |
| `app/data_providers/repositories/` | Repository pattern for data access abstraction |
| `app/forms/` | Form objects; autoloaded |
| `app/serializers/` | Data serializers; autoloaded |
| `app/decorators/` | Decorator objects wrapping AR models for presentation |
| `app/presenters/` | Presenter objects for view/API response shaping |
| `app/workers/` | Sidekiq background workers |
| `app/jobs/` | ActiveJob (queued via Sidekiq) |
| `app/components/` | ViewComponent server-rendered components |
| `app/event_driven/` | Webhook event pub/sub (events, adapters, publisher, subscriber) |
