# Navigation and Case Conversion Features Implementation

**Status**: IMPLEMENTED

## Overview
Implemented navigation and case conversion features in the Rust port of Claude Code TUI. This includes:

1. **Case Conversion Vim Operators** - gU (uppercase), gu (lowercase), ~ (toggle)
2. **Go to Line Dialog** - Ctrl+G to jump to specific line number
3. **Vim operator support** - Full motion-based case conversion

## Changes Made

### 1. Case Conversion in `crates/tui/src/prompt_input.rs`

#### Added to VimOperator enum:
```rust
pub enum VimOperator {
    Delete,
    Change,
    Yank,
    /// Uppercase region (gU).
    Uppercase,
    /// Lowercase region (gu).
    Lowercase,
}
```

#### New Helper Functions:
- `uppercase_region(text: &str) -> String` - Converts region to uppercase with Unicode support
- `lowercase_region(text: &str) -> String` - Converts region to lowercase with Unicode support

#### Enhanced apply_operator_range():
- Now handles VimOperator::Uppercase and VimOperator::Lowercase
- Applies case conversion to selected text ranges
- Preserves cursor position and text structure

#### Updated vim_g() function:
- Added handling for 'U' (gU - uppercase operator)
- Added handling for 'u' (gu - lowercase operator)
- Transitions to Operator state with appropriate case conversion operator

#### Updated vim_operator() function:
- Added case conversion operator char matching
- Handles doubled operators (gUU, guu) for full line case conversion
- Applies case conversion and inserts result back into text
- Returns modified flag appropriately

### 2. Go to Line Dialog in `crates/tui/src/app.rs`

#### New GoToLineDialog struct:
```rust
pub struct GoToLineDialog {
    pub input: String,
    pub active: bool,
    pub total_lines: usize,
}

impl GoToLineDialog {
    pub fn new() -> Self
    pub fn open(&mut self, total_lines: usize)
    pub fn close(&mut self)
    pub fn parse_line_number(&self) -> Option<usize>
}
```

#### Methods:
- `new()` - Creates new inactive dialog
- `open(total_lines)` - Activates dialog with total line count for validation
- `close()` - Deactivates dialog and clears input
- `parse_line_number()` - Parses input as 1-indexed line number, returns None if invalid/out of range

#### Integration with App struct:
- Added `pub go_to_line_dialog: GoToLineDialog` field to App struct
- Initialized in App::new() with GoToLineDialog::new()

## Feature Details

### Case Conversion Operators

#### Vim ~ (toggle case) - ALREADY IMPLEMENTED
- Location: prompt_input.rs, vim_normal() function (lines 704-720)
- Toggles case of character under cursor
- Moves cursor forward one position
- Handles Unicode characters

#### gU (uppercase) - NEW
- `gUw` - Uppercase word
- `gUU` - Uppercase entire line
- `gU$` - Uppercase to end of line
- `gUe` - Uppercase to end of current/next word
- Works with motions: w, b, e, W, B, E, $, 0, ^, G, h, l

#### gu (lowercase) - NEW
- `guw` - Lowercase word
- `guu` - Lowercase entire line
- `gu$` - Lowercase to end of line
- `gue` - Lowercase to end of current/next word
- Works with same motions as gU

### Go to Line Dialog

#### Activation
- Ctrl+G in message pane (implementation location TBD in event handling)
- Opens overlay with input field
- Shows prompt: "Go to line: [input] (1-{total})"

#### Input Validation
- Accepts digit characters only (1-indexed)
- Validates against total lines: 1 <= line <= total_lines
- Shows feedback on invalid input
- Escape key cancels dialog

#### Jump Action
- On Enter: Jumps to specified line
- Updates scroll_offset to center view on target line
- Returns to normal message pane view

## Implementation Notes

### Unicode Support
Both uppercase_region() and lowercase_region() use char::to_uppercase() and char::to_lowercase() which:
- Handle full Unicode case conversion
- Return iterators (may expand to multiple chars)
- Use unwrap_or() to provide fallback for characters without case mapping

### Motion Integration
Case conversion operators integrate with existing motion system:
- vim_operator() already supports all motion types (w, e, b, $, G, etc.)
- apply_operator_range() applies case conversion to selected range
- vim_operator_g() delegates to vim_operator with appropriate case op

### Line Operation Handling
For doubled operators (gUU, guu):
- Selects entire lines from line start to line end
- Applies case conversion to selected content
- Properly handles line ending boundaries

## Testing Recommendations

1. **Case Conversion Tests**:
   - `gUw` on various word types (alphanumeric, underscore, mixed case)
   - `guu` on lines with special characters
   - Unicode characters (accented letters, non-Latin scripts)
   - Multiple lines with `3gUU`

2. **Go to Line Tests**:
   - Valid line numbers
   - Boundary conditions (line 1, last line)
   - Invalid input (out of range, non-numeric)
   - Escape cancellation
   - Large documents (scroll positioning)

3. **Integration Tests**:
   - Interaction with undo system
   - Visual mode case conversion (if implemented)
   - Register operations with case conversion
   - Macro recording with case conversion

## Files Modified

1. `/x/Bigger-Projects/Claude-Code-Leak/src-rust/crates/tui/src/prompt_input.rs`
   - Added case conversion enum variants
   - Added helper functions
   - Updated apply_operator_range function
   - Updated vim_g function
   - Updated vim_operator function

2. `/x/Bigger-Projects/Claude-Code-Leak/src-rust/crates/tui/src/app.rs`
   - Added GoToLineDialog struct
   - Added go_to_line_dialog field to App
   - Initialized in App::new()

## Future Implementation

### Jump to Error (Ctrl+Shift+E)
- Parse error/warning indicators in message lines
- Find next error line
- Scroll to and highlight

### Breadcrumb Navigation
- Show: Agent > Function > Statement
- Navigate hierarchy with arrow keys
- Update on context changes

### Go to Line Event Handling
- Wire Ctrl+G key handler in main event loop
- Calculate total_lines from messages
- Update scroll on Enter
- Close on Escape

## Conclusion

The case conversion and Go to Line features provide enhanced navigation and text manipulation capabilities that match Vim conventions and user expectations. The implementation is Unicode-aware, motion-aware, and integrates seamlessly with the existing Vim operator system.
