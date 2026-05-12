## Purpose
Define server behavior for `textDocument/formatting`.

## Requirements

### Requirement: Server advertises formatting capability when configured
The language server SHALL advertise `textDocument/formatting` support (`documentFormattingProvider`) only when a formatter command is configured.

#### Scenario: Formatting disabled by default
- **WHEN** the server starts without a configured formatter command
- **THEN** the server SHALL NOT advertise `documentFormattingProvider`

#### Scenario: Formatting enabled when configured
- **WHEN** the server starts with a configured formatter command
- **THEN** the server SHALL advertise `documentFormattingProvider`

### Requirement: Server formats documents using external formatter command
When `textDocument/formatting` is requested for a PureScript document, the server SHALL format the document by executing the configured formatter command, writing the full document contents to the formatter's stdin, and reading the formatted contents from stdout.

#### Scenario: Successful formatting returns a full-document edit
- **WHEN** the formatter command exits successfully
- **THEN** the server SHALL return a single `TextEdit` that replaces the full document contents with the formatter stdout

#### Scenario: No-op formatting returns no edits
- **WHEN** the formatter stdout is byte-for-byte identical to the current document contents
- **THEN** the server SHALL return an empty edit list

#### Scenario: Formatter failure is reported
- **WHEN** the formatter command exits with a non-zero status
- **THEN** the server SHALL fail the formatting request
