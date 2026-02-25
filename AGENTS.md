# AGENTS.md - Bubble Datepicker Development Guide

This document provides essential information for AI agents and developers working on the bubble-datepicker project.

## Build, Lint, and Test Commands

### Building
```bash
go build .
```
Builds the project to verify compilation.

### Testing
```bash
go test -v
```
Runs all tests with verbose output.

```bash
go test -run TestName
```
Runs a specific test function (e.g., `go test -run TestNew`).

### Code Quality
```bash
go vet .
```
Checks for common Go programming errors.

```bash
gofmt -w .
```
Formats Go code according to standard conventions (run this before commits).

```bash
go mod tidy
```
Cleans up go.mod and go.sum files, adding missing dependencies and removing unused ones.

## Code Style Guidelines

### Go Version
- Minimum Go version: 1.21.1 (as specified in go.mod)
- Target Go version: 1.21.1

### File Structure
- Main package file: `bubble-datepicker.go`
- Test file: `bubble-datepicker_test.go`
- Generated code: `focus_string.go` (from `go:generate stringer -type=Focus`)
- Examples in `examples/` directory with subdirectories for different use cases

### Naming Conventions
- **Packages**: Use descriptive, lowercase names (this package uses `datepicker`)
- **Exported types/functions**: PascalCase (e.g., `Model`, `New`, `SetFocus`, `FocusCalendar`)
- **Unexported types/functions**: camelCase (e.g., `updateUp`, `updateRight`)
- **Constants**: PascalCase for exported, camelCase for unexported (e.g., `FocusNone`, `FocusCalendar`)
- **Variables**: camelCase (e.g., `halloween`, `thanksgiving`)

### Import Organization
```go
import (
    "fmt"
    "strconv"
    "strings"
    "time"

    "github.com/charmbracelet/bubbles/key"
    tea "github.com/charmbracelet/bubbletea"
    "github.com/charmbracelet/lipgloss"
)
```
- Standard library imports first (alphabetically sorted)
- Blank line separating standard library from external packages
- External packages alphabetically sorted
- Use aliases for commonly used packages (e.g., `tea` for bubbletea)

### Code Formatting
- **Indentation**: Tabs (not spaces), 4 space width
- **Line endings**: LF (Unix-style)
- **Max line length**: No strict limit, but keep lines readable
- Use `gofmt` for automatic formatting

### Type Definitions
- Use custom types for enums and constants:
```go
type Focus int

const (
    FocusNone Focus = iota
    FocusHeaderMonth
    FocusHeaderYear
    FocusCalendar
)
```
- Implement `String()` method for enums using `go:generate stringer`

### Struct Organization
- Group related fields together
- Document struct fields with comments
- Use meaningful field names
```go
type Model struct {
    // Time is the time.Time struct that represents the selected date month and year
    Time time.Time

    // KeyMap encodes the keybindings recognized by the model
    KeyMap KeyMap

    // Styles represent the Styles struct used to render the datepicker
    Styles Styles

    // Focused indicates the component which the end user is focused on
    Focused Focus

    // Selected indicates whether a date is Selected in the datepicker
    Selected bool
}
```

### Function Structure
- Exported functions start with capital letters and have documentation comments
- Functions follow single responsibility principle
- Use receiver methods for model operations
- Bubbletea pattern: Implement `Init()`, `Update(msg tea.Msg)`, and `View()` methods

### Error Handling
- Follow standard Go error handling patterns
- Return errors from functions that can fail
- Use `fmt.Errorf` for error wrapping
- Handle errors appropriately in calling code

### Testing Patterns
- Use table-driven tests for multiple test cases:
```go
func TestSetFocus(t *testing.T) {
    tests := []struct {
        input Focus
        want  Focus
    }{
        {input: FocusNone, want: FocusNone},
        {input: FocusCalendar, want: FocusCalendar},
    }

    model := New(halloween)
    for i, test := range tests {
        model.SetFocus(test.input)
        if got := model.Focused; test.want != got {
            t.Errorf("TestSetFocus failure - index: %d - want: '%s' got: '%s'", i, test.want, got)
        }
    }
}
```
- Use descriptive test names starting with `Test`
- Test both exported and internal functionality
- Use UTC time for consistent test results
- Test edge cases and normal operation

### Comments and Documentation
- Package comment at top of files: `// Package datepicker provides...`
- Exported functions/types get documentation comments
- Use `// TODO:` for future improvements
- Inline comments for complex logic

### Bubbletea Integration
- Follow Bubbletea component patterns
- Implement the `tea.Model` interface
- Use proper message handling in `Update()` method
- Return appropriate commands from update functions

### Styling
- Use Charm Bracelet lipgloss for styling
- Define style structs with default functions
- Use adaptive colors where appropriate
- Follow consistent color schemes

### Key Bindings
- Define key maps with reasonable defaults
- Support both arrow keys and vim-style navigation (h,j,k,l)
- Include quit keys (ctrl+c, q)
- Use Charm Bracelet key package for key handling

### Dependencies
- Primary dependencies:
  - `github.com/charmbracelet/bubbles`
  - `github.com/charmbracelet/bubbletea`
  - `github.com/charmbracelet/lipgloss`
- Keep dependencies minimal and focused
- Use `go mod tidy` to manage dependencies

### Examples
- Provide clear, runnable examples in `examples/` directory
- Each example should be self-contained
- Include README files for each example
- Demonstrate different use cases (basic, list, textinput integration)

### Version Control
- Use conventional commit messages
- Keep generated files in sync with source (run `go generate` when Focus enum changes)
- Format code before committing
- Run tests before pushing changes

## Development Workflow

1. Make changes to code
2. Run `gofmt -w .` to format code
3. Run `go vet .` to check for issues
4. Run `go test -v` to ensure tests pass
5. Run `go build .` to verify compilation
6. Run `go mod tidy` to clean dependencies
7. Commit with descriptive message

## Common Patterns

### Time Handling
- Use `time.Time` for all date operations
- UTC timezone for consistency in tests
- Proper date arithmetic with `AddDate()` method

### Rendering
- Build strings using `strings.Builder` for efficiency
- Use lipgloss for styled output
- Handle different focus states in rendering

### State Management
- Use struct methods to modify model state
- Keep state changes predictable and testable
- Separate concerns between different focus areas