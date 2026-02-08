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
