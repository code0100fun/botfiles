---
name: memory-leak-detector
description: Detects potential memory leaks in Zig code by checking allocation/deallocation patterns, defer statements, and test cleanup. Use when reviewing code changes, writing new code with allocations, or debugging memory issues.
---

# Memory Leak Detector

This skill helps identify potential memory leaks in Zig code by analyzing allocation patterns and ensuring proper cleanup.

## What This Skill Checks

### 1. Allocation/Deallocation Pairing

Every allocation must have a corresponding deallocation:

**Common allocation patterns:**
- `allocator.alloc()` -> requires `allocator.free()`
- `allocator.create()` -> requires `allocator.destroy()`
- `Type.init(allocator)` -> requires `instance.deinit()`
- `ArrayList.init()` -> requires `list.deinit()`
- `HashMap.init()` -> requires `map.deinit()`

**Proper cleanup pattern:**
```zig
// GOOD: Immediate defer after allocation
const buffer = try allocator.alloc(u8, 100);
defer allocator.free(buffer);

const instance = try allocator.create(MyType);
defer allocator.destroy(instance);

var list = ArrayList(u8).init(allocator);
defer list.deinit();
```

**Missing cleanup:**
```zig
// BAD: No cleanup
const buffer = try allocator.alloc(u8, 100);
// Missing: defer allocator.free(buffer)

const instance = MyType.init(allocator);
// Missing: defer instance.deinit()
```

### 2. Test Function Cleanup

All test functions must clean up allocated resources:

**Proper test cleanup:**
```zig
test "my feature" {
    const allocator = std.testing.allocator;

    var server = try Server.init(allocator);
    defer server.deinit();  // Cleanup before test ends

    const value = try Value.create(allocator, "test");
    defer value.deinit();  // Cleanup allocated value

    // Test logic here
}
```

**Missing test cleanup:**
```zig
test "my feature" {
    const allocator = std.testing.allocator;

    var server = try Server.init(allocator);
    // Missing: defer server.deinit()

    const value = try Value.create(allocator, data);
    // Missing: defer value.deinit()

    // Test will leak memory!
}
```

### 3. Conditional Allocations

Handle cleanup in error paths:

**Proper error path cleanup:**
```zig
// GOOD: Arena allocator for automatic cleanup
var arena = std.heap.ArenaAllocator.init(parent_allocator);
defer arena.deinit();
const allocator = arena.allocator();

// All allocations automatically freed on arena.deinit()
```

**Or manual error handling:**
```zig
// GOOD: Manual cleanup on error
const a = try allocator.alloc(u8, 10);
errdefer allocator.free(a);

const b = try allocator.alloc(u8, 20);
errdefer allocator.free(b);

// If second allocation fails, first is cleaned up
```

**Missing error cleanup:**
```zig
// BAD: Leak on error
const a = try allocator.alloc(u8, 10);
const b = try allocator.alloc(u8, 20);  // If this fails, 'a' leaks
defer allocator.free(a);
defer allocator.free(b);
```

### 4. Struct/Type Lifecycle

Types with resources must implement proper cleanup:

**Required pattern:**
```zig
pub const MyType = struct {
    allocator: Allocator,
    buffer: []u8,

    pub fn init(allocator: Allocator, size: usize) !MyType {
        const buffer = try allocator.alloc(u8, size);
        return MyType{
            .allocator = allocator,
            .buffer = buffer,
        };
    }

    pub fn deinit(self: *MyType) void {
        self.allocator.free(self.buffer);
        self.* = undefined;  // Safety: prevent use-after-free
    }
};
```

**Usage:**
```zig
var instance = try MyType.init(allocator, 100);
defer instance.deinit();  // Required
```

### 5. Arena Allocator Usage

For temporary allocations, prefer arena allocators:

**Recommended pattern:**
```zig
pub fn processData(parent_allocator: Allocator) !Result {
    var arena = std.heap.ArenaAllocator.init(parent_allocator);
    defer arena.deinit();  // Single cleanup for all allocations
    const allocator = arena.allocator();

    // All temporary allocations use arena
    const temp1 = try allocator.alloc(u8, 100);
    const temp2 = try allocator.alloc(u8, 200);

    // No need for individual defer statements
    // All freed automatically on arena.deinit()

    return result;
}
```

## Red Flags to Watch For

### 1. Missing defer After Allocation

```zig
// RED FLAG
const data = try allocator.alloc(u8, size);
// Next line should be: defer allocator.free(data)
```

### 2. Allocation in Loop Without Cleanup

```zig
// RED FLAG: Allocates on every iteration
for (items) |item| {
    const temp = try allocator.alloc(u8, item.size);
    // Missing cleanup - leaks on each iteration
}

// BETTER: Use arena or defer inside loop
for (items) |item| {
    const temp = try allocator.alloc(u8, item.size);
    defer allocator.free(temp);  // Cleanup each iteration
}
```

### 3. Early Returns Without Cleanup

```zig
// This is OK - defer runs on all exit paths
const data = try allocator.alloc(u8, 100);
defer allocator.free(data);

if (condition) {
    return error.Failed;  // defer will run
}
```

But watch for:

```zig
// ACTUAL RED FLAG
const data = try allocator.alloc(u8, 100);

if (condition) {
    return error.Failed;  // Leaks data
}

defer allocator.free(data);  // Too late for early returns
```

### 4. Storing Allocations in Containers

```zig
// WATCH OUT: Who owns the memory?
var list = ArrayList(*MyType).init(allocator);
defer list.deinit();  // Only frees the list, not items

const item = try allocator.create(MyType);
try list.append(item);  // Must free item separately

// PROPER CLEANUP
defer {
    for (list.items) |item| {
        allocator.destroy(item);
    }
    list.deinit();
}
```

## When Reviewing Code

### Questions to Ask:

1. **For every allocation:**
   - Is there a corresponding `defer` statement?
   - Is the `defer` immediately after the allocation?
   - Does the cleanup happen on all exit paths?

2. **For every test:**
   - Does it use `std.testing.allocator`?
   - Are all allocated resources freed before test ends?
   - Would this test pass under leak detection?

3. **For every struct with resources:**
   - Does it have a `deinit()` method?
   - Does `deinit()` free all owned resources?
   - Is ownership clearly documented?

4. **For complex functions:**
   - Should this use an arena allocator?
   - Are error paths handled correctly?
   - Is `errdefer` used appropriately?

## Action Items When Leak Detected

1. **Identify the allocation** - What function allocates the memory?
2. **Find the owner** - Who is responsible for cleanup?
3. **Add defer** - Place it immediately after allocation
4. **Verify all paths** - Check error returns and early exits
5. **Test** - Run under leak detection to verify fix

## Testing for Leaks

Zig's test allocator detects leaks automatically:

```zig
test "no leaks" {
    const allocator = std.testing.allocator;

    // If any allocation isn't freed, test fails
    const data = try allocator.alloc(u8, 100);
    defer allocator.free(data);
}
```

Run tests to detect leaks:
```bash
zig build test
```

Leaks will show as test failures with allocation tracking info.

## Summary Checklist

When writing new code:
- [ ] Every `alloc/create/init` has corresponding `free/destroy/deinit`
- [ ] `defer` statement immediately follows allocation
- [ ] Test functions clean up all resources
- [ ] Error paths handled with `errdefer`
- [ ] Consider arena allocator for temporary allocations
- [ ] Document ownership in comments
- [ ] Run tests with leak detection

When reviewing code:
- [ ] Check for missing `defer` statements
- [ ] Verify test cleanup
- [ ] Look for allocations in loops
- [ ] Check error path cleanup
- [ ] Verify struct `deinit` implementations
- [ ] Run tests to confirm no leaks
