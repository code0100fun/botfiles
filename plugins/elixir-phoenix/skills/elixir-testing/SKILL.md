---
name: elixir-testing
description: Best practices for testing Elixir applications with Ecto.SQL.Sandbox, including background process handling, Oban testing, and test output quality. Use when writing or reviewing Elixir tests.
---

# Elixir Testing with Ecto.SQL.Sandbox - Best Practices

## Overview

This guide covers production-ready patterns for testing Elixir applications with Ecto.SQL.Sandbox, particularly when dealing with background processes (GenServers, Oban workers, etc.) that need database access.

## Core Principle: Prefer Manual Mode with Selective Shared Mode

**The modern best practice** is to use `:manual` mode globally with selective shared mode via test tags:

```elixir
# test/support/data_case.ex
setup tags do
  pid = Ecto.Adapters.SQL.Sandbox.start_owner!(YourApp.Repo,
    shared: not tags[:async])
  on_exit(fn -> Ecto.Adapters.SQL.Sandbox.stop_owner(pid) end)
  :ok
end
```

This provides:
- **Async tests** (concurrent, fast) for simple unit tests
- **Shared mode** (automatic DB access) for complex integration tests with `async: false`
- **Automatic cleanup** via `start_owner!/2`

## Sandbox Modes Comparison

| Feature | Manual Mode | Shared Mode | Recommendation |
|---------|-------------|-------------|----------------|
| Async tests | Yes | No | Manual is better |
| Test performance | Fast | Slow | Manual is better |
| Background processes | Requires `allow/3` | Auto-access | Depends on use case |
| Explicitness | Clear | Implicit | Manual is better |
| Setup complexity | Medium | Low | Shared is simpler |
| **Production use** | **Recommended** | Selective use | **Use manual** |

**Decision Matrix:**
- Simple unit tests? -> `async: true` (manual mode, no allowances needed)
- Background processes but known PIDs? -> `async: true` with explicit `allow/3`
- Complex integration with many processes? -> `async: false` (auto-shared mode)

## Common Pitfalls and Solutions

### Pitfall 1: Using Global Shared Mode

**Problem:**
```elixir
# test/test_helper.exs
Ecto.Adapters.SQL.Sandbox.mode(YourApp.Repo, {:shared, self()})
```

**Why it's bad:**
- Forces ALL tests to use `async: false`
- Significantly slower test suite
- Doesn't scale as project grows

**Solution:**
Use manual mode with selective shared mode via `start_owner!/2` pattern.

### Pitfall 2: Using Mix.env() in Production Code

**Problem:**
```elixir
# lib/your_app/application.ex (WRONG!)
defp children do
  if Mix.env() == :test do
    []
  else
    [YourApp.IPAM]
  end
end
```

**Why it's bad:**
- `Mix` is not available in releases
- Application will crash in production
- Compile-time check, not runtime

**Solution:**
Use application configuration or configure via test setup:
```elixir
# Use start_supervised! in tests that need the process
test "test that needs IPAM" do
  start_supervised!(YourApp.IPAM)
  # Test code
end
```

## Oban Testing Configuration

### Config Setup

**Recommended Configuration:**
```elixir
# config/test.exs
config :your_app, Oban,
  testing: :manual,  # Jobs persist, use perform_job/3 to execute
  plugins: false     # Prevent background plugins from starting
```

**Why this configuration:**
- Prevents Stager, Pruner, and other plugins from causing sandbox errors
- Allows testing job enqueueing separately from execution
- More flexible for integration testing
- Recommended by Oban documentation

## Rules Summary

### Database & Concurrency
1. **USE manual mode** as default (`Ecto.Adapters.SQL.Sandbox.mode(Repo, :manual)`)
2. **USE `start_owner!/2`** instead of `checkout/2` (modern pattern)
3. **USE selective shared mode** via `shared: not tags[:async]`
4. **USE `start_supervised!/1`** for automatic process cleanup
5. **CONFIGURE Oban** with `testing: :manual` and `plugins: false`
6. **ENABLE async: true** wherever possible for performance
7. **USE async: false** for complex integration tests
8. **NEVER use global shared mode** in test_helper.exs
9. **NEVER use `Mix.env()`** in production code (lib/ directory)
10. **NEVER forget cleanup** - use `start_supervised!` or `on_exit`

### Test Output Quality
11. **CAPTURE all expected logs** with `capture_log/1` or `capture_io/1`
12. **ASSERT on captured logs** for error/warning cases
13. **ENSURE zero warnings/errors** in test output
14. **NEVER commit noisy tests** - silence expected logs immediately
15. **NEVER use capture to hide real problems** - investigate unexpected errors

## Test Output Quality

### CRITICAL: Test Output Must Be Clean

**Test runs MUST produce zero warnings and zero error log output.** This is not optional - noisy test output masks real problems and makes debugging significantly harder.

### Silencing Expected Logs

When your code intentionally produces log output (errors, warnings, info), you **MUST** silence it in tests using `ExUnit.CaptureLog` or `ExUnit.CaptureIO`.

**Correct pattern - capture and assert:**

```elixir
import ExUnit.CaptureLog

test "handles failure gracefully" do
  log = capture_log(fn ->
    result = SomeWorker.perform(%{will: "fail"})
    assert {:error, _} = result
  end)

  assert log =~ "Operation failed"
  assert log =~ "Expected error message"
end
```

**Correct pattern - silence expected info logs:**

```elixir
test "successful operation" do
  capture_log(fn ->
    result = SomeWorker.perform(%{will: "succeed"})
    assert {:ok, _} = result
  end)
end
```

**WRONG - letting logs pollute test output:**

```elixir
test "handles failure" do
  # This will spam error logs to console!
  result = SomeWorker.perform(%{will: "fail"})
  assert {:error, _} = result
end
```

### When to Use capture_log vs capture_io

- **`capture_log/1`** - For Logger calls (`Logger.error`, `Logger.info`, etc.)
- **`capture_io/1`** - For IO calls (`IO.puts`, `IO.warn`, etc.)
- **`capture_io/2`** - For capturing specific devices (`:stderr`, `:stdio`)

### Rules for Test Output

1. **ALWAYS** use `capture_log/1` when testing code that intentionally logs
2. **ASSERT** on captured log content for error/warning cases
3. **IMPORT** `ExUnit.CaptureLog` at the top of test modules that need it
4. **INVESTIGATE** any uncaptured warnings or errors - they indicate real problems
5. **NEVER** ignore test output noise - fix it immediately
6. **NEVER** use capture to hide unexpected errors or warnings
7. **NEVER** commit tests that produce console spam

### Example: Testing Worker with Expected Failure

```elixir
defmodule YourApp.WorkerTest do
  use YourApp.DataCase, async: false
  import ExUnit.CaptureLog

  test "reserves resource before operation to prevent races" do
    {:ok, resource} = create_resource()

    expect(MockService, :do_operation, fn ->
      assert get_resource!(resource.id).status == "reserved"
      {:error, "Operation failed"}
    end)

    log = capture_log(fn ->
      result = Worker.perform(%{resource_id: resource.id})
      assert {:error, _} = result
    end)

    assert log =~ "Worker failed"
    assert log =~ "Operation failed"

    assert get_resource!(resource.id).status == "failed"
    assert get_resource!(resource.id).reserved_id != nil
  end
end
```

## References

- [Ecto.Adapters.SQL.Sandbox](https://hexdocs.pm/ecto_sql/Ecto.Adapters.SQL.Sandbox.html)
- [Oban Testing Guide](https://hexdocs.pm/oban/testing.html)
- [Phoenix.Ecto.SQL.Sandbox](https://hexdocs.pm/phoenix_ecto/Phoenix.Ecto.SQL.Sandbox.html)
- [ExUnit.CaptureLog](https://hexdocs.pm/ex_unit/ExUnit.CaptureLog.html)
- [ExUnit.CaptureIO](https://hexdocs.pm/ex_unit/ExUnit.CaptureIO.html)
