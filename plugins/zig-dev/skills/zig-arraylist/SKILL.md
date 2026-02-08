---
name: zig-arraylist
description: Reference for Zig 0.15+ ArrayList API which passes the allocator to each method call instead of at initialization. Use when working with ArrayLists in Zig 0.15+.
---

# Zig 0.15 ArrayList API

This project uses Zig 0.15 which has a different ArrayList API than earlier versions.

## Key Differences from Older Zig Versions

### Initialization

**Old (pre-0.15):**
```zig
var list = std.ArrayList(T).init(allocator);
```

**New (0.15+):**
```zig
var list = std.ArrayList(T){};
```

The allocator is NOT passed at initialization. Instead, it's passed to each method call.

### Common Operations

**Append:**
```zig
// Old
try list.append(item);

// New (0.15)
try list.append(allocator, item);
```

**Deinit:**
```zig
// Old
list.deinit();

// New (0.15)
list.deinit(allocator);
```

**AppendSlice:**
```zig
// Old
try list.appendSlice(items);

// New (0.15)
try list.appendSlice(allocator, items);
```

**ToOwnedSlice:**
```zig
// Old
const slice = try list.toOwnedSlice();

// New (0.15)
const slice = try list.toOwnedSlice(allocator);
```

## Complete Example

```zig
const std = @import("std");

fn example(allocator: std.mem.Allocator) ![]u32 {
    // Initialize without allocator
    var list = std.ArrayList(u32){};
    defer list.deinit(allocator);

    // All operations take allocator as first arg
    try list.append(allocator, 1);
    try list.append(allocator, 2);
    try list.append(allocator, 3);

    // Convert to owned slice
    return try list.toOwnedSlice(allocator);
}
```

## Why This Change?

Zig 0.15 moved to a more explicit allocator passing style. This makes it clearer where allocations happen and allows using different allocators for different operations on the same list.

## Common Error Messages

If you see errors like:
- `struct 'array_list.Aligned(T,null)' has no member named 'init'`
- `expected 2 argument(s), found 1`

You're likely using the old API. Update to the new pattern shown above.
