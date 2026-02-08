---
name: liveview-guidelines
description: Phoenix LiveView patterns covering streams, JavaScript interop (hooks, colocated scripts, push events), testing, and form handling. Use when building or reviewing LiveView features.
---

# Phoenix LiveView Guidelines

## General

- **Never** use the deprecated `live_redirect` and `live_patch` functions, instead **always** use `<.link navigate={href}>` and `<.link patch={href}>` in templates, and `push_navigate` and `push_patch` in LiveViews
- **Avoid LiveComponents** unless you have a strong, specific need for them
- LiveViews should be named like `AppWeb.WeatherLive`, with a `Live` suffix. When adding routes, the default `:browser` scope is **already aliased** with the `AppWeb` module, so you can just do `live "/weather", WeatherLive`

## LiveView Streams

- **Always** use LiveView streams for collections instead of assigning regular lists to avoid memory ballooning and runtime termination:
  - Basic append: `stream(socket, :messages, [new_msg])`
  - Reset stream: `stream(socket, :messages, [new_msg], reset: true)` (e.g. for filtering)
  - Prepend: `stream(socket, :messages, [new_msg], at: -1)`
  - Delete: `stream_delete(socket, :messages, msg)`

- When using streams, the template must:
  1. Set `phx-update="stream"` on the parent element with a DOM id
  2. Consume `@streams.stream_name` and use the id as the DOM id for each child:

```heex
<div id="messages" phx-update="stream">
  <div :for={{id, msg} <- @streams.messages} id={id}>
    {msg.text}
  </div>
</div>
```

- LiveView streams are *not* enumerable. You cannot use `Enum.filter/2` or `Enum.reject/2` on them. To filter or refresh, **refetch the data and re-stream with `reset: true`**:

```elixir
def handle_event("filter", %{"filter" => filter}, socket) do
  messages = list_messages(filter)

  {:noreply,
   socket
   |> assign(:messages_empty?, messages == [])
   |> stream(:messages, messages, reset: true)}
end
```

- Streams *do not support counting or empty states*. Track counts with a separate assign. For empty states, use Tailwind:

```heex
<div id="tasks" phx-update="stream">
  <div class="hidden only:block">No tasks yet</div>
  <div :for={{id, task} <- @streams.tasks} id={id}>
    {task.name}
  </div>
</div>
```

The empty state only works if it's the only HTML block alongside the stream for-comprehension.

- When updating an assign that should change content inside streamed items, you MUST re-stream the items along with the updated assign:

```elixir
def handle_event("edit_message", %{"message_id" => message_id}, socket) do
  message = Chat.get_message!(message_id)
  edit_form = to_form(Chat.change_message(message, %{content: message.content}))

  {:noreply,
   socket
   |> stream_insert(:messages, message)
   |> assign(:editing_message_id, String.to_integer(message_id))
   |> assign(:edit_form, edit_form)}
end
```

- **Never** use the deprecated `phx-update="append"` or `phx-update="prepend"` for collections

## JavaScript Interop

- When using `phx-hook="MyHook"` and that JS hook manages its own DOM, you **must** also set `phx-update="ignore"`
- **Always** provide a unique DOM id alongside `phx-hook`

### Inline Colocated JS Hooks

**Never** write raw embedded `<script>` tags in HEEx. Instead, **always use colocated js hook script tags**:

```heex
<input type="text" name="user[phone_number]" id="user-phone-number" phx-hook=".PhoneNumber" />
<script :type={Phoenix.LiveView.ColocatedHook} name=".PhoneNumber">
  export default {
    mounted() {
      this.el.addEventListener("input", e => {
        let match = this.el.value.replace(/\D/g, "").match(/^(\d{3})(\d{3})(\d{4})$/)
        if(match) {
          this.el.value = `${match[1]}-${match[2]}-${match[3]}`
        }
      })
    }
  }
</script>
```

- Colocated hooks are automatically integrated into the app.js bundle
- Colocated hook names **MUST ALWAYS** start with a `.` prefix (e.g., `.PhoneNumber`)

### External phx-hook

External JS hooks must be placed in `assets/js/` and passed to the LiveSocket constructor:

```javascript
const MyHook = {
  mounted() { ... }
}
let liveSocket = new LiveSocket("/live", Socket, {
  hooks: { MyHook }
});
```

### Pushing Events Between Client and Server

Use `push_event/3` to push events/data to the client for a phx-hook to handle. **Always** return or rebind the socket:

```elixir
socket = push_event(socket, "my_event", %{...})

# or return directly:
def handle_event("some_event", _, socket) do
  {:noreply, push_event(socket, "my_event", %{...})}
end
```

Receive in JS hook with `this.handleEvent`:

```javascript
mounted() {
  this.handleEvent("my_event", data => console.log("from server:", data));
}
```

Push from client to server with reply:

```javascript
mounted() {
  this.el.addEventListener("click", e => {
    this.pushEvent("my_event", { one: 1 }, reply => console.log("got reply:", reply));
  })
}
```

Server handles with reply:

```elixir
def handle_event("my_event", %{"one" => 1}, socket) do
  {:reply, %{two: 2}, socket}
end
```

## LiveView Tests

- Use `Phoenix.LiveViewTest` module and `LazyHTML` (included) for assertions
- Form tests are driven by `render_submit/2` and `render_change/2`
- Split test cases into small, isolated files. Start with simpler content-existence tests, then add interaction tests
- **Always reference key element IDs** from your templates in tests for `element/2`, `has_element/2`, etc.
- **Never** test against raw HTML; **always** use `element/2`, `has_element/2`, and similar
- Focus on testing outcomes rather than implementation details
- Be aware that `Phoenix.Component` functions may produce different HTML than expected. Test against output structure
- When facing test failures with selectors, debug with `LazyHTML`:

```elixir
html = render(view)
document = LazyHTML.from_fragment(html)
matches = LazyHTML.filter(document, "your-complex-selector")
IO.inspect(matches, label: "Matches")
```

## Form Handling

### Creating a Form from Params

```elixir
def handle_event("submitted", params, socket) do
  {:noreply, assign(socket, form: to_form(params))}
end
```

When passing a map to `to_form/1`, it assumes string keys. You can specify a name:

```elixir
def handle_event("submitted", %{"user" => user_params}, socket) do
  {:noreply, assign(socket, form: to_form(user_params, as: :user))}
end
```

### Creating a Form from Changesets

```elixir
%YourApp.Users.User{}
|> Ecto.Changeset.change()
|> to_form()
```

In the template:

```heex
<.form for={@form} id="todo-form" phx-change="validate" phx-submit="save">
  <.input field={@form[:field]} type="text" />
</.form>
```

Always give the form an explicit, unique DOM ID.

### Avoiding Form Errors

**Always** use a form assigned via `to_form/2` in the LiveView, and the `<.input>` component in the template:

```heex
<%!-- ALWAYS do this (valid) --%>
<.form for={@form} id="my-form">
  <.input field={@form[:field]} type="text" />
</.form>
```

**Never** do this:

```heex
<%!-- NEVER do this (invalid) --%>
<.form for={@changeset} id="my-form">
  <.input field={@changeset[:field]} type="text" />
</.form>
```

- You are FORBIDDEN from accessing the changeset in the template
- **Never** use `<.form let={f} ...>`, **always** use `<.form for={@form} ...>` and drive all form references from the form assign
