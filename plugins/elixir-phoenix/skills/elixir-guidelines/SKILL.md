---
name: elixir-guidelines
description: Core Elixir language guidelines covering common pitfalls like list access, immutability, module organization, struct access, date/time handling, and OTP patterns. Use when writing or reviewing Elixir code.
---

# Elixir Guidelines

## List Access

Elixir lists **do not support index based access via the access syntax**.

**Never do this (invalid)**:

```elixir
i = 0
mylist = ["blue", "green"]
mylist[i]
```

Instead, **always** use `Enum.at`, pattern matching, or `List` for index based list access:

```elixir
i = 0
mylist = ["blue", "green"]
Enum.at(mylist, i)
```

## Variable Rebinding in Block Expressions

Elixir variables are immutable, but can be rebound. For block expressions like `if`, `case`, `cond`, etc. you *must* bind the result of the expression to a variable if you want to use it. You CANNOT rebind the result inside the expression:

```elixir
# INVALID: rebinding inside the `if` - the result never gets assigned
if connected?(socket) do
  socket = assign(socket, :val, val)
end

# VALID: rebind the result of the `if` to a new variable
socket =
  if connected?(socket) do
    assign(socket, :val, val)
  end
```

## Module Organization

- **Never** nest multiple modules in the same file as it can cause cyclic dependencies and compilation errors

## Struct Access

- **Never** use map access syntax (`changeset[:field]`) on structs as they do not implement the Access behaviour by default. For regular structs, you **must** access the fields directly, such as `my_struct.field` or use higher level APIs that are available on the struct if they exist (e.g., `Ecto.Changeset.get_field/2` for changesets)

## Date and Time

- Elixir's standard library has everything necessary for date and time manipulation. Familiarize yourself with the common `Time`, `Date`, `DateTime`, and `Calendar` interfaces. **Never** install additional dependencies unless asked or for date/time parsing (which you can use the `date_time_parser` package)

## Naming Conventions

- Don't use `String.to_atom/1` on user input (memory leak risk)
- Predicate function names should not start with `is_` and should end in a question mark. Names like `is_thing` should be reserved for guards

## Error Handling

- **Avoid `try/rescue`** — it is an anti-pattern in Elixir. Use `{:ok, result}` / `{:error, reason}` tuples and pattern matching instead
- Use non-bang functions (`Repo.get/2`, `File.read/1`) and match on the result rather than bang functions (`Repo.get!/2`) wrapped in `try/rescue`
- `rescue` should only be used at true system boundaries (e.g., NIF calls, third-party libraries that raise instead of returning error tuples) where no tuple-based alternative exists
- If a function only has a bang variant, wrap it in a helper that returns `{:ok, val} | {:error, reason}` once, then use that everywhere

```elixir
# BAD: using try/rescue for control flow
try do
  agent = Agents.get_agent!(id)
  do_something(agent)
rescue
  Ecto.NoResultsError -> :not_found
end

# GOOD: pattern matching on result tuples
case Agents.get_agent(id) do
  {:ok, agent} -> do_something(agent)
  {:error, :not_found} -> :not_found
end
```

## OTP Patterns

- Elixir's builtin OTP primitives like `DynamicSupervisor` and `Registry` require names in the child spec:

```elixir
{DynamicSupervisor, name: YourApp.MyDynamicSup}
```

Then use:
```elixir
DynamicSupervisor.start_child(YourApp.MyDynamicSup, child_spec)
```

## Concurrency

- Use `Task.async_stream(collection, callback, options)` for concurrent enumeration with back-pressure. The majority of times you will want to pass `timeout: :infinity` as option

## Mix Guidelines

- Read the docs and options before using tasks (by using `mix help task_name`)
- To debug test failures, run tests in a specific file with `mix test test/my_test.exs` or run all previously failed tests with `mix test --failed`
- `mix deps.clean --all` is **almost never needed**. **Avoid** using it unless you have good reason

## Test Guidelines

- **Always use `start_supervised!/1`** to start processes in tests as it guarantees cleanup between tests
- **Avoid** `Process.sleep/1` and `Process.alive?/1` in tests
  - Instead of sleeping to wait for a process to finish, **always** use `Process.monitor/1` and assert on the DOWN message:

    ```elixir
    ref = Process.monitor(pid)
    assert_receive {:DOWN, ^ref, :process, ^pid, :normal}
    ```

  - Instead of sleeping to synchronize before the next call, **always** use `_ = :sys.get_state/1` to ensure the process has handled prior messages
