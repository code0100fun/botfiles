---
name: error-handling-validator
description: Validates proper error handling patterns in Zig code including custom error sets, error context, and error propagation. Use when writing error-prone code, reviewing error handling, or debugging error cases.
---

# Error Handling Validator

This skill ensures Zig code follows best practices for error handling, including custom error sets, proper propagation, and meaningful error context.

## Zig Error Handling Fundamentals

### Error Sets

Define custom error sets for each module:

```zig
// GOOD: Module-specific error set
pub const ParseError = error{
    InvalidMagic,
    UnsupportedVersion,
    MalformedChunk,
    UnknownChunkType,
    BufferTooSmall,
};

pub fn parseData(data: []const u8) ParseError!Result {
    if (data.len < 4) return error.BufferTooSmall;
    if (!std.mem.eql(u8, data[0..4], "MAGIC")) return error.InvalidMagic;
    // ...
}
```

**Avoid generic errors:**
```zig
// BAD: No custom error set
pub fn parseData(data: []const u8) !Result {
    if (data.len < 4) return error.TooSmall;  // Unclear error
    // ...
}
```

### Error Unions

Functions that can fail return error unions:

```zig
// GOOD: Clear error union return type
pub fn execute(self: *Server) ServerError!void {
    // ...
}

// GOOD: Combined error sets
pub fn run(self: *Server) (ServerError || AllocatorError)!Result {
    // ...
}

// BAD: Hiding errors with void
pub fn process(self: *Server) void {
    self.execute() catch |err| {
        // Silently ignoring errors!
    };
}
```

### Error Propagation

Use `try` for simple propagation, `catch` for handling:

```zig
// GOOD: Propagate errors up
pub fn loadConfig(self: *Server, path: []const u8) !void {
    const file = try std.fs.cwd().openFile(path, .{});
    defer file.close();

    const data = try file.readToEndAlloc(self.allocator, max_size);
    defer self.allocator.free(data);

    const config = try Config.parse(data);
    try self.applyConfig(config);
}

// BAD: Catching without handling
pub fn loadConfig(self: *Server, path: []const u8) void {
    const file = std.fs.cwd().openFile(path, .{}) catch return;
    // Lost error information!
}
```

## Error Handling Patterns

### 1. Define Module-Level Error Sets

Each module should define its error set at the top:

```zig
// src/parser.zig
pub const ParseError = error{
    InvalidMagic,
    UnsupportedVersion,
    MalformedSection,
    UnknownSectionType,
    BufferTooSmall,
    InvalidNameTable,
    InvalidCodeSection,
};

// src/server.zig
pub const ServerError = error{
    InvalidOpcode,
    StackOverflow,
    StackUnderflow,
    RegisterOutOfBounds,
    UndefinedFunction,
    InvalidArity,
    SpawnFailed,
};

// src/value.zig
pub const ValueError = error{
    InvalidType,
    ListTooLong,
    ContainerTooBig,
    NameTooLong,
    InvalidEncoding,
};
```

### 2. Combine Error Sets When Needed

```zig
// GOOD: Explicit error set combination
pub fn loadAndExecute(self: *Server, path: []const u8) (ParseError || ServerError || std.fs.File.OpenError)!void {
    const config = try Config.parseFile(path);  // ParseError || File.OpenError
    try self.execute(config);  // ServerError
}

// ALSO GOOD: Use anyerror for complex combinations
pub fn complexOperation() anyerror!Result {
    // When combining many error sets
}
```

### 3. Add Error Context

Provide context when catching errors:

```zig
// GOOD: Add context to errors
pub fn loadConfig(self: *Server, path: []const u8) !void {
    const config = Config.parseFile(path) catch |err| {
        std.log.err("Failed to load config from {s}: {}", .{ path, err });
        return err;
    };
}

// BETTER: Use std.log for debugging
pub fn execute(self: *Server) !void {
    const opcode = self.fetchOpcode() catch |err| {
        std.log.err("Fetch failed at IP={d}: {}", .{ self.ip, err });
        return err;
    };
}

// BAD: Silent error swallowing
pub fn execute(self: *Server) void {
    self.fetchOpcode() catch return;  // No context!
}
```

### 4. Use errdefer for Cleanup

Clean up partial state on errors:

```zig
// GOOD: errdefer for error-path cleanup
pub fn init(allocator: Allocator) !Server {
    const stack = try allocator.alloc(Value, 1024);
    errdefer allocator.free(stack);

    const registers = try allocator.alloc(Value, 256);
    errdefer allocator.free(registers);

    const heap = try allocator.alloc(u8, 65536);
    errdefer allocator.free(heap);

    return Server{
        .allocator = allocator,
        .stack = stack,
        .registers = registers,
        .heap = heap,
    };
}
```

### 5. Document Error Conditions

```zig
/// Executes the next instruction.
///
/// Returns:
/// - ServerError.InvalidOpcode if opcode is unknown
/// - ServerError.StackOverflow if stack is full
/// - ServerError.RegisterOutOfBounds if register index invalid
pub fn execute(self: *Server) ServerError!void {
    // Implementation
}
```

## Common Anti-Patterns

### 1. Catching All Errors Without Handling

```zig
// BAD: Losing error information
pub fn process(data: []const u8) void {
    parseData(data) catch return;  // What went wrong?
}

// GOOD: Log or propagate
pub fn process(data: []const u8) !void {
    try parseData(data);  // Propagate up
}

// ALSO GOOD: Handle specific errors
pub fn process(data: []const u8) !void {
    parseData(data) catch |err| {
        std.log.err("Parse failed: {}", .{err});
        return err;
    };
}
```

### 2. Using Generic Errors

```zig
// BAD: Generic errors are unclear
pub fn validate(self: *Server) !void {
    if (self.ip >= self.code.len) return error.Invalid;
    if (self.sp >= self.stack.len) return error.Error;
}

// GOOD: Specific error names
pub fn validate(self: *Server) ServerError!void {
    if (self.ip >= self.code.len) return error.InvalidInstructionPointer;
    if (self.sp >= self.stack.len) return error.StackOverflow;
}
```

### 3. Panic Instead of Error

```zig
// BAD: Panic for recoverable errors
pub fn getRegister(self: *Server, index: u8) Value {
    if (index >= self.registers.len) {
        @panic("Register out of bounds");  // Crashes program!
    }
    return self.registers[index];
}

// GOOD: Return error
pub fn getRegister(self: *Server, index: u8) ServerError!Value {
    if (index >= self.registers.len) {
        return error.RegisterOutOfBounds;
    }
    return self.registers[index];
}
```

**When to use panic:**
- Programmer errors (unreachable states, contract violations)
- Assertions in debug builds
- Truly unrecoverable situations

**When to use errors:**
- Invalid user input
- File I/O failures
- Network errors
- Resource exhaustion
- Malformed data

### 4. Not Using errdefer

```zig
// BAD: Memory leak on error
pub fn init(allocator: Allocator) !Server {
    const stack = try allocator.alloc(Value, 1024);
    const registers = try allocator.alloc(Value, 256);  // If this fails, stack leaks!

    return Server{ .stack = stack, .registers = registers };
}

// GOOD: errdefer prevents leak
pub fn init(allocator: Allocator) !Server {
    const stack = try allocator.alloc(Value, 1024);
    errdefer allocator.free(stack);

    const registers = try allocator.alloc(Value, 256);
    errdefer allocator.free(registers);

    return Server{ .stack = stack, .registers = registers };
}
```

### 5. Ignoring Error Return Values

```zig
// BAD: Silently ignoring errors
pub fn run(self: *Server) void {
    _ = self.execute();  // Ignoring error!
}

// GOOD: Handle or propagate
pub fn run(self: *Server) !void {
    try self.execute();  // Propagate
}

// ALSO GOOD: Explicitly handle
pub fn run(self: *Server) void {
    self.execute() catch |err| {
        std.log.err("Execution failed: {}", .{err});
        self.halt();
    };
}
```

## Error Handling Checklist

### When Writing New Code:

- [ ] Define custom error set for the module
- [ ] Use specific error names (not `Error`, `Invalid`, etc.)
- [ ] Document which errors each function can return
- [ ] Use `try` to propagate errors up
- [ ] Use `errdefer` for cleanup on error paths
- [ ] Log errors with context before returning
- [ ] Prefer errors over panics for recoverable failures

### When Reviewing Code:

- [ ] Are custom error sets defined?
- [ ] Are error names descriptive and specific?
- [ ] Is error context provided (logging, messages)?
- [ ] Are errors properly propagated or handled?
- [ ] Is `errdefer` used for cleanup?
- [ ] Are there any silent error swallowing (`catch return`, `catch {}`)?
- [ ] Are there any inappropriate panics?
- [ ] Is error documentation complete?

## Testing Error Cases

Always test error paths:

```zig
test "parse returns error on invalid magic" {
    const data = "INVALID_MAGIC";
    const result = parseData(data);

    try std.testing.expectError(ParseError.InvalidMagic, result);
}

test "execute returns error on stack overflow" {
    var server = try Server.init(std.testing.allocator);
    defer server.deinit();

    // Fill stack
    while (server.sp < server.stack.len) {
        try server.push(Value.makeInt(0));
    }

    // Next push should overflow
    try std.testing.expectError(ServerError.StackOverflow, server.push(Value.makeInt(1)));
}
```

## Summary

Error handling in Zig should be:
1. **Explicit** - Use custom error sets, not anyerror
2. **Specific** - Use descriptive error names
3. **Documented** - Comment possible errors
4. **Contextual** - Log errors with useful information
5. **Safe** - Use errdefer for cleanup
6. **Testable** - Test error paths explicitly

**Golden Rule:** If an operation can fail, return an error. Never silently ignore failures.
