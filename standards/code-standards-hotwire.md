---
name: code-standards-hotwire
description: Circle V2 Hotwire (Turbo + Stimulus) development standards for Rails views, components, and controllers.
---

# Hotwire Code Standards — Circle V2

Guidelines for building Hotwire views, Stimulus controllers, and Turbo interactions in Circle V2.

## 🎯 Scope

This document covers:
- **Turbo Frames & Streams** — Partial page updates via `turbo-frame` and `turbo-stream`
- **Stimulus Controllers** — Client-side interactivity (`@hotwired/stimulus`)
- **Rails View Components** — Reusable ERB + Ruby component classes
- **Form Interactions** — Turbo-powered form submissions and validations

---

## 📁 Directory Structure

```
app/
├── views/                          # ERB templates (Hotwire default)
│   ├── shared/                     # Shared partials
│   ├── settings/                   # Feature-specific views
│   └── [feature]/
│       ├── index.html.erb
│       ├── show.html.erb
│       ├── _form.html.erb          # ← Forms loaded into Turbo frames
│       └── _item.html.erb          # ← Partial for Turbo stream updates
│
├── components/                     # Rails View Components (reusable)
│   ├── ui/
│   │   ├── pagination/
│   │   │   ├── pagination.rb       # Component logic
│   │   │   └── pagination.html.erb # ERB template
│   │   └── [component]/
│   └── [feature]/
│
└── javascript/
    └── controllers/                # Stimulus controllers
        ├── index.js                # Stimulus auto-loader
        ├── [feature]/
        │   └── form_controller.js
        └── backoffice/
            └── toast_controller.js
```

---

## 🏗️ Turbo Frames & Streams

### Turbo Frame Usage

**Pattern: Scoped page updates without full reload**

```erb
<!-- app/views/settings/external_services/index.html.erb -->
<div class="container">
  <h1>External Services</h1>
  
  <!-- Isolated update zone -->
  <%= turbo_frame_tag "payment_integrations" do %>
    <%= render "payment_integrations/list", integrations: @integrations %>
  <% end %>
</div>
```

**Rules:**
- ✅ Use `turbo_frame_tag` for isolated update regions
- ✅ Give frames meaningful IDs: `turbo_frame_tag "payment_integrations"` (not generic `id="content"`)
- ✅ Nest frames only 1-2 levels deep (avoid complexity)
- ❌ Don't nest `turbo_frame_tag` without reason

### Turbo Action & Link Behavior

```erb
<!-- pagination.html.erb -->
<%= link_to page, page_url(page), 
      class: "btn",
      data: { 
        turbo_frame: "results",        # ← Target frame ID
        turbo_action: "advance"        # ← Update browser history
      }
%>
```

**Turbo Actions:**
- `advance` — Load response into frame, update browser history
- `replace` — Load response into frame, update history
- `stream` — Process `turbo-stream` responses
- (default) — Navigate page normally

### Turbo Streams for Real-Time Updates

```erb
<!-- app/views/orders/_item.html.erb -->
<div id="order-<%= order.id %>" class="order-card">
  <%= order.name %>
</div>

<!-- Stimulus event triggers server to send turbo-stream: -->
<!-- Server response: app/views/orders/update.turbo_stream.erb -->
<turbo-stream action="update" target="order-<%= @order.id %>">
  <template>
    <%= render "item", order: @order %>
  </template>
</turbo-stream>
```

**Guidelines:**
- ✅ Use turbo-stream for multi-element updates
- ✅ Keep stream responses focused (1-3 actions max)
- ✅ Use `action="update"` for replacements, `action="append"` for additions
- ❌ Don't send entire page as turbo-stream (use turbo-frame instead)

---

## ⚡ Stimulus Controllers

### Controller Structure

```javascript
// app/javascript/controllers/backoffice/toast_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static targets = ['message', 'container']  // ← DOM targets
  static values = { duration: { type: Number, default: 2500 } }  // ← Configuration

  connect() {
    // Called when controller connects to DOM
    this.setupEventListener()
  }

  disconnect() {
    // Called when controller removed from DOM
    this.cleanup()
  }

  setupEventListener() {
    window.addEventListener('toast:notification', (event) => {
      this.show(event.detail.message, event.detail.kind)
    })
  }

  show(message, kind = 'success') {
    const wrapper = document.createElement('div')
    wrapper.innerHTML = `<div class="alert alert-${kind}">${message}</div>`
    this.containerTarget.appendChild(wrapper)
    setTimeout(() => wrapper.remove(), this.durationValue)
  }

  cleanup() {
    // Remove listeners if needed
  }
}
```

**Rules:**
- ✅ Define `targets` and `values` as static properties
- ✅ Use kebab-case for controller names: `toast_controller.js`
- ✅ Use camelCase for actions: `show`, `toggle`, `validate`
- ✅ Always implement `disconnect()` for cleanup
- ❌ Don't add complex business logic (keep in Rails model/service)
- ❌ Don't make blocking API calls without feedback

### HTML Markup for Stimulus

```erb
<!-- Stimulus data attributes -->
<div data-controller="backoffice--toast"
     data-backoffice--toast-duration-value="3000">
  
  <!-- Action triggers -->
  <button data-action="click->backoffice--toast#show">Show Toast</button>
  
  <!-- Targets for element access -->
  <div data-backoffice--toast-target="container"></div>
</div>
```

**Naming Convention:**
- Controller: `data-controller="backoffice--toast"` (double dash for nested paths)
- Action: `data-action="click->backoffice--toast#show"`
- Target: `data-backoffice--toast-target="container"`
- Value: `data-backoffice--toast-duration-value="2500"`

### Event-Driven Communication

```javascript
// Stimulus dispatching custom events
show(message) {
  this.dispatch('shown', { detail: { message } })
}

// JavaScript listening to Stimulus events
document.addEventListener('backoffice-toast:shown', (event) => {
  console.log('Toast shown:', event.detail.message)
})
```

**Pattern:**
- Stimulus fires events → JavaScript listeners react
- Keep controllers independent (no direct DOM manipulation of other controllers)

---

## 🧩 Rails View Components

### Component Class + Template

```ruby
# app/components/ui/pagination/pagination.rb
class Ui::Pagination::PaginationComponent < ViewComponent::Base
  attr_reader :current_page, :total_pages, :turbo_frame

  def initialize(current_page:, total_pages:, turbo_frame: nil)
    @current_page = current_page
    @total_pages = total_pages
    @turbo_frame = turbo_frame
  end

  def window_pages
    # Calculate pagination window (e.g., 1 2 3 ... 10 11 12)
    (1..total_pages).to_a
  end

  def page_url(page)
    # Override in child class
    raise NotImplementedError
  end
end
```

```erb
<!-- app/components/ui/pagination/pagination.html.erb -->
<div class="flex justify-center">
  <div class="join">
    <% window_pages.each do |page| %>
      <%= link_to page, page_url(page),
            class: "btn #{'btn-active' if page == current_page}",
            data: { turbo_frame: turbo_frame, turbo_action: "advance" }.compact
      %>
    <% end %>
  </div>
</div>
```

**Rules:**
- ✅ Use ViewComponent for reusable UI elements
- ✅ Keep component logic minimal (calculations only)
- ✅ Pass data via `initialize` parameters
- ✅ Make optional parameters explicit: `turbo_frame: nil`
- ❌ Don't fetch data in components (fetch in controller)
- ❌ Don't have side effects in components

---

## 📝 Forms with Turbo

### Basic Form Pattern

```erb
<!-- app/views/settings/external_services/_form.html.erb -->
<%= form_with(model: @service, local: true) do |form| %>
  <% if @service.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@service.errors.count, "error") %> prohibited this service:</h2>
      <ul>
        <% @service.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="form-group">
    <%= form.label :name %>
    <%= form.text_field :name, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= form.submit class: "btn btn-primary" %>
  </div>
<% end %>
```

### Form in Turbo Frame (Partial Page Update)

```erb
<!-- app/views/settings/index.html.erb -->
<h1>Settings</h1>

<!-- Form loaded into frame, submission returns stream response -->
<%= turbo_frame_tag "edit_service" do %>
  <%= render "form", service: @service %>
<% end %>
```

```ruby
# app/controllers/settings_controller.rb
def update
  @service = Service.find(params[:id])
  if @service.update(service_params)
    respond_to do |format|
      format.turbo_stream { 
        render turbo_stream: turbo_stream.update("edit_service", 
          partial: "form", locals: { service: @service })
      }
    end
  else
    respond_to do |format|
      format.turbo_stream { 
        render turbo_stream: turbo_stream.replace("edit_service", 
          partial: "form", locals: { service: @service })
      }
    end
  end
end
```

**Rules:**
- ✅ Use Rails form builders: `form_with`, `form.text_field`, etc.
- ✅ Keep forms wrapped in turbo-frames for isolated updates
- ✅ Respond with turbo-stream on validation errors
- ✅ Include error messages in stream response
- ❌ Don't submit forms outside turbo-frames (defeats purpose)
- ❌ Don't redirect on form errors (return stream with updated form)

---

## 🧪 System Test Patterns

### Testing Turbo Frames

```ruby
# spec/system/hotwire/settings_spec.rb
describe "Settings Page" do
  it "updates payment integrations without full reload" do
    visit settings_path
    
    # Initial page loads
    expect(page).to have_content("External Services")
    
    # Click pagination link (Turbo frame update)
    find_link("2", inside: turbo_frame("payment_integrations")).click
    
    # Frame updated without full page reload
    expect(page).to have_current_path(settings_path)  # URL unchanged
    expect(page).to have_selector("turbo-frame#payment_integrations")
    expect(page).to have_content("Page 2")
  end
end
```

### Testing Stimulus Interactions

```ruby
# spec/system/hotwire/toast_spec.rb
describe "Toast Controller", js: true do
  it "shows notification on custom event", :js do
    visit root_path
    
    # Fire custom event from JavaScript
    execute_script(
      "window.dispatchEvent(new CustomEvent('toast:notification', { " +
      "  detail: { message: 'Test', kind: 'success' } " +
      "}))"
    )
    
    # Check toast appears
    expect(page).to have_text("Test")
    expect(page).to have_selector(".alert-success")
  end
end
```

**Rules:**
- ✅ Add `js: true` to tests requiring JavaScript/Stimulus
- ✅ Use `turbo_frame()` matcher for frame assertions
- ✅ Use `execute_script()` for custom events
- ✅ Test both success and error states
- ❌ Don't test internal Stimulus details (test behavior)

---

## ✅ Code Standards Checklist

**Before submitting Hotwire PR:**

- [ ] All Stimulus controllers in `app/javascript/controllers/`
- [ ] All Views in `app/views/` (ERB, not Vue 3)
- [ ] All View Components have `.rb` + `.html.erb` files
- [ ] Turbo frames have meaningful IDs
- [ ] Forms respond with turbo-stream on errors
- [ ] Stimulus controllers clean up on `disconnect()`
- [ ] System tests use `js: true` where needed
- [ ] No console errors in browser (test locally)
- [ ] `/code-standards-check` passes
- [ ] Full regression test passes (`/regression-tester`)

---

## 📚 Resources

- **Turbo Documentation**: https://turbo.hotwired.dev/
- **Stimulus Documentation**: https://stimulus.hotwired.dev/
- **Rails ViewComponent**: https://viewcomponent.org/
- **Circle V2 Examples**: `app/javascript/controllers/` and `app/components/`

---

## 🔄 Related Standards

- **Backend Standards**: `code-standards-backend.md` (models, controllers, services)
- **Testing Standards**: `testing-standards.md` (RSpec + system tests)
- **Architecture**: `frontend-patterns.md` (Hotwire section)
