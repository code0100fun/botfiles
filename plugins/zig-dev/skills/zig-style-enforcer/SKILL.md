---
name: zig-style-enforcer
description: Enforces Zig coding style conventions including naming, formatting, idioms, and best practices. Use when writing new code, reviewing changes, or refactoring for consistency.
---

# Zig Style Enforcer

This skill ensures code follows Zig conventions and idioms for consistency and readability.

## Naming Conventions

### Functions and Variables: snake_case

```zig
// GOOD
pub fn parseConfigFile(path: []const u8) !Config { }
const max_stack_size = 1024;
var instruction_pointer: usize = 0;

// BAD
pub fn ParseConfigFile(path: []const u8) !Config { }  // PascalCase
const MaxStackSize = 1024;  // PascalCase
var instructionPointer: usize = 0;  // camelCase
```

### Types and Namespaces: PascalCase

```zig
// GOOD
pub const Server = struct { };
pub const ConfigFile = struct { };
pub const ValueType = enum { };

// BAD
pub const server = struct { };  // lowercase
pub const config_file = struct { };  // snake_case
pub const value_type = enum { };  // snake_case
```

### Constants: Depends on Context

```zig
// Module-level constants use snake_case
const default_stack_size = 1024;
const max_name_length = 255;

// Enum values use snake_case
pub const Opcode = enum {
    label,
    func_info,
    call,
    call_last,
};

// Compile-time parameters use PascalCase
pub fn List(comptime T: type) type {
    return struct { };
}
```

### Private vs Public

```zig
pub const Server = struct {
    // Public fields
    pub allocator: Allocator,

    // Private fields (no pub)
    stack: []Value,
    registers: []Value,

    // Public methods
    pub fn init(allocator: Allocator) !Server { }
    pub fn deinit(self: *Server) void { }

    // Private methods (no pub)
    fn executeInstruction(self: *Server) !void { }
};
```

## Code Organization

### Module Structure

```zig
// 1. Standard library imports
const std = @import("std");
const Allocator = std.mem.Allocator;

// 2. Local imports
const Value = @import("value.zig").Value;
const Config = @import("config.zig").Config;

// 3. Module-level constants
const MAX_STACK_SIZE = 1024;
const DEFAULT_REGISTERS = 256;

// 4. Error sets
pub const ServerError = error{
    StackOverflow,
    InvalidOpcode,
};

// 5. Type definitions
pub const Server = struct {
    // ...
};

// 6. Public functions
pub fn execute(server: *Server) !void {
    // ...
}

// 7. Private helper functions
fn validateOpcode(op: u8) bool {
    // ...
}

// 8. Tests
test "Server initialization" {
    // ...
}
```

### File Naming

```zig
// GOOD: snake_case for files
server.zig
config_parser.zig
value_list.zig

// BAD
Server.zig  // PascalCase
configParser.zig  // camelCase
```

## Zig Idioms

### Use `const` by Default

```zig
// GOOD: const for immutable
const allocator = std.heap.page_allocator;
const file_path = "config.json";

// BAD: var when const would work
var allocator = std.heap.page_allocator;

// GOOD: var only when needed
var counter: usize = 0;
counter += 1;  // Mutated, so var is correct
```

### Use `defer` for Cleanup

```zig
// GOOD: defer immediately after allocation
const buffer = try allocator.alloc(u8, 1024);
defer allocator.free(buffer);

var file = try std.fs.cwd().openFile(path, .{});
defer file.close();

// BAD: defer far from allocation
const buffer = try allocator.alloc(u8, 1024);
// ... lots of code ...
defer allocator.free(buffer);  // Easy to miss
```

### Use `errdefer` for Error Cleanup

```zig
pub fn init(allocator: Allocator) !Server {
    const stack = try allocator.alloc(Value, 1024);
    errdefer allocator.free(stack);

    const registers = try allocator.alloc(Value, 256);
    errdefer allocator.free(registers);

    return Server{
        .stack = stack,
        .registers = registers,
    };
}
```

### Prefer Slices Over Pointers

```zig
// GOOD: Slices include length
pub fn process(data: []const u8) !void {
    if (data.len < 4) return error.BufferTooSmall;
    // ...
}

// BAD: Raw pointers lose size info
pub fn process(data: [*]const u8, len: usize) !void {
    if (len < 4) return error.BufferTooSmall;
    // ...
}
```

### Use Tagged Unions

```zig
pub const Value = union(enum) {
    small_int: i32,
    big_int: *BigInt,
    float: f64,
    string: []const u8,
    list: *ListValue,
    map: *MapValue,

    pub fn deinit(self: Value) void {
        switch (self) {
            .small_int, .float => {},  // No cleanup
            .big_int => |bi| bi.deinit(),
            .string => |s| s.deinit(),
            .list => |l| l.deinit(),
            .map => |m| m.deinit(),
        }
    }
};
```

### Use Comptime for Generic Code

```zig
pub fn ArrayList(comptime T: type) type {
    return struct {
        items: []T,
        allocator: Allocator,

        pub fn init(allocator: Allocator) ArrayList(T) {
            return .{
                .items = &[_]T{},
                .allocator = allocator,
            };
        }
    };
}
```

### Prefer Error Unions Over Optionals for Failures

```zig
// GOOD: Error union provides context
pub fn divide(a: i32, b: i32) !i32 {
    if (b == 0) return error.DivisionByZero;
    return @divTrunc(a, b);
}

// LESS GOOD: Optional loses error info
pub fn divide(a: i32, b: i32) ?i32 {
    if (b == 0) return null;  // Why did it fail?
    return @divTrunc(a, b);
}
```

### Use Explicit Field Initialization

```zig
// GOOD: Explicit field names
return Server{
    .allocator = allocator,
    .stack = stack,
    .registers = registers,
    .ip = 0,
    .sp = 0,
};
```

## Code Style

### Indentation and Spacing

- 4 spaces for indentation
- Blank line between functions
- No trailing whitespace

### Line Length

```zig
// GOOD: Keep lines under 100 characters
const result = try processConfigFile(
    allocator,
    file_path,
    .{ .validate = true, .debug = false },
);
```

### Comments

```zig
// GOOD: Doc comments for public API
/// Executes the next instruction.
///
/// Returns ServerError if execution fails.
pub fn execute(self: *Server) ServerError!void {
    // TODO: Add support for tail call optimization
}

// BAD: Useless comments
// This function executes
pub fn execute(self: *Server) !void { }
```

### Switch Statements

```zig
// GOOD: Exhaustive switches
switch (value) {
    .small_int => |i| std.debug.print("{d}", .{i}),
    .float => |f| std.debug.print("{d}", .{f}),
    .string => |s| std.debug.print("{s}", .{s}),
    .list => |l| printList(l),
    .map => |m| printMap(m),
}

// GOOD: Use else for catch-all if needed
switch (opcode) {
    .call, .call_last => try self.executeCall(),
    .ret => try self.executeReturn(),
    else => return error.InvalidOpcode,
}
```

### Error Handling

```zig
// GOOD: Propagate errors
pub fn run(self: *Server) !void {
    try self.execute();
    try self.validate();
}

// GOOD: Handle specific errors
pub fn run(self: *Server) void {
    self.execute() catch |err| switch (err) {
        error.StackOverflow => self.handleOverflow(),
        error.InvalidOpcode => self.handleInvalidOp(),
        else => {
            std.log.err("Unexpected error: {}", .{err});
            return;
        },
    };
}

// BAD: Silent error swallowing
pub fn run(self: *Server) void {
    self.execute() catch {};  // Errors ignored!
}
```

## Anti-Patterns to Avoid

### Don't Use Hungarian Notation

```zig
// BAD
const pBuffer: *u8 = undefined;
const nCount: usize = 0;

// GOOD
const buffer: *u8 = undefined;
const count: usize = 0;
```

### Don't Mix Styles

```zig
// BAD: Inconsistent naming
pub const Server_State = struct {
    currentIP: usize,
    stack_pointer: usize,
};

// GOOD: Consistent naming
pub const ServerState = struct {
    current_ip: usize,
    stack_pointer: usize,
};
```

### Don't Over-Comment

```zig
// BAD: Comments state the obvious
// Increment the counter
counter += 1;

// GOOD: Comments explain why
// Skip null terminators in name table
if (byte == 0) continue;
```

### Don't Use Global State

```zig
// BAD: Global mutable state
var global_server: Server = undefined;

pub fn execute() void {
    global_server.run();
}

// GOOD: Pass state explicitly
pub fn execute(server: *Server) void {
    server.run();
}
```

## Style Checklist

When writing code:
- [ ] Functions and variables use snake_case
- [ ] Types use PascalCase
- [ ] Use `const` by default, `var` only when mutating
- [ ] `defer` immediately after allocation
- [ ] Prefer slices over raw pointers
- [ ] Use tagged unions for variants
- [ ] Explicit field initialization
- [ ] Doc comments on public API
- [ ] Exhaustive switch statements
- [ ] Lines under 100 characters
- [ ] No trailing whitespace

When reviewing code:
- [ ] Naming consistent with conventions
- [ ] Cleanup properly structured with defer
- [ ] Error handling explicit, not silent
- [ ] Comments add value, not noise
- [ ] Code formatted consistently
- [ ] Tests descriptive and comprehensive

## Formatting

Use `zig fmt` to automatically format code:

```bash
# Format a single file
zig fmt src/server.zig

# Format entire project
zig fmt src/
```

**Always run `zig fmt` before committing code.**
