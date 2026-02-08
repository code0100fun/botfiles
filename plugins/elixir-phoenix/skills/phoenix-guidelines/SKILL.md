---
name: phoenix-guidelines
description: Phoenix 1.8 framework conventions covering layouts, routing, components, icons, forms, HTTP clients, and design file handling. Use when building or reviewing Phoenix web applications.
---

# Phoenix Guidelines

## General

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included `:req` (`Req`) library for HTTP requests. **Avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

## Phoenix v1.8

- **Always** begin your LiveView templates with `<Layouts.app flash={@flash} ...>` which wraps all inner content
- The `AppWeb.Layouts` module is aliased in the `app_web.ex` file, so you can use it without needing to alias it again
- Anytime you run into errors with no `current_scope` assign:
  - You failed to follow the Authenticated Routes guidelines, or you failed to pass `current_scope` to `<Layouts.app>`
  - **Always** fix the `current_scope` error by moving your routes to the proper `live_session` and ensure you pass `current_scope` as needed
- Phoenix v1.8 moved the `<.flash_group>` component to the `Layouts` module. You are **forbidden** from calling `<.flash_group>` outside of the `layouts.ex` module
- Out of the box, `core_components.ex` imports an `<.icon name="hero-x-mark" class="w-5 h-5"/>` component for hero icons. **Always** use the `<.icon>` component for icons, **never** use `Heroicons` modules or similar
- **Always** use the imported `<.input>` component for form inputs from `core_components.ex` when available. Using it will save steps and prevent errors
- If you override the default input classes (`<.input class="myclass px-2 py-1 rounded-lg">`) with your own values, no default classes are inherited, so your custom classes must fully style the input

## Routing

- Remember Phoenix router `scope` blocks include an optional alias which is prefixed for all routes within the scope. **Always** be mindful of this when creating routes within a scope to avoid duplicate module prefixes
- You **never** need to create your own `alias` for route definitions! The `scope` provides the alias:

```elixir
scope "/admin", AppWeb.Admin do
  pipe_through :browser

  live "/users", UserLive, :index
end
```

The UserLive route would point to the `AppWeb.Admin.UserLive` module.

- `Phoenix.View` no longer is needed or included with Phoenix, don't use it

## Design Files (.pen)

- **Never** edit `.pen` files directly as JSON blobs. These are design files that must only be modified through the Pencil MCP server tools
- Use Pencil MCP tools for inserting, updating, copying, or deleting design elements
- Use Pencil screenshot tools to verify design changes visually
