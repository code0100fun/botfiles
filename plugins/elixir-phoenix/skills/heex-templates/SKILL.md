---
name: heex-templates
description: Phoenix HEEx template rules covering interpolation syntax, form building, class lists, conditional rendering, comments, and common anti-patterns. Use when writing or reviewing HEEx templates.
---

# Phoenix HTML / HEEx Template Guidelines

## Template Format

- Phoenix templates **always** use `~H` or .html.heex files (known as HEEx), **never** use `~E`

## Forms

- **Always** use the imported `Phoenix.Component.form/1` and `Phoenix.Component.inputs_for/1` function to build forms. **Never** use `Phoenix.HTML.form_for` or `Phoenix.HTML.inputs_for` as they are outdated
- When building forms **always** use the already imported `Phoenix.Component.to_form/2`:

```elixir
assign(socket, form: to_form(...))
```

```heex
<.form for={@form} id="msg-form">
  <.input field={@form[:content]} type="text" />
</.form>
```

- **Always** add unique DOM IDs to key elements (like forms, buttons, etc) when writing templates - these IDs can later be used in tests

## App-Wide Template Imports

- For "app wide" template imports, you can import/alias into the `your_app_web.ex`'s `html_helpers` block, so they will be available to all LiveViews, LiveComponents, and all modules that do `use YourAppWeb, :html`

## Conditional Rendering

- Elixir supports `if/else` but **does NOT support `if/else if` or `if/elsif`**. **Never use `else if` or `elseif` in Elixir`**, **always** use `cond` or `case` for multiple conditionals:

```heex
<%!-- NEVER do this (invalid) --%>
<%= if condition do %>
  ...
<% else if other_condition %>
  ...
<% end %>

<%!-- ALWAYS do this --%>
<%= cond do %>
  <% condition -> %>
    ...
  <% condition2 -> %>
    ...
  <% true -> %>
    ...
<% end %>
```

## Literal Curly Braces

HEEx require special tag annotation if you want to insert literal curly braces. For code snippets in `<pre>` or `<code>` blocks, annotate the parent tag with `phx-no-curly-interpolation`:

```heex
<code phx-no-curly-interpolation>
  let obj = {key: "val"}
</code>
```

Within annotated tags, you can use `{` and `}` without escaping, and dynamic Elixir expressions can still be used with `<%= ... %>` syntax.

## Class Lists

HEEx class attrs support lists, but you must **always** use list `[...]` syntax. Use the class list syntax to conditionally add classes:

```heex
<a class={[
  "px-2 text-white",
  @some_flag && "py-5",
  if(@other_condition, do: "border-red-500", else: "border-blue-100"),
]}>Text</a>
```

**Always** wrap `if`'s inside `{...}` expressions with parens.

**Never** do this (missing `[` and `]`):

```heex
<%!-- INVALID --%>
<a class={
  "px-2 text-white",
  @some_flag && "py-5"
}> ...
```

## Iteration

- **Never** use `<% Enum.each %>` or non-for comprehensions for generating template content, instead **always** use `<%= for item <- @collection do %>`

## Comments

- HEEx HTML comments use `<%!-- comment --%>`. **Always** use this syntax for template comments

## Interpolation

HEEx allows interpolation via `{...}` and `<%= ... %>`, but the `<%= %>` **only** works within tag bodies.

**Always** do this:

```heex
<div id={@id}>
  {@my_assign}
  <%= if @some_block_condition do %>
    {@another_assign}
  <% end %>
</div>
```

**Never** do this (syntax error):

```heex
<%!-- THIS IS INVALID --%>
<div id="<%= @invalid_interpolation %>">
  {if @invalid_block_construct do}
  {end}
</div>
```

Rules:
- Use `{...}` for interpolation within tag attributes
- Use `{...}` for interpolation of values within tag bodies
- Use `<%= ... %>` for block constructs (if, cond, case, for) within tag bodies
