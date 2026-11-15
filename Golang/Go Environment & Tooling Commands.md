2026-11-12 00:21

Tags: 

- **`go version`** Verifies the installation and displays the current Go version and architecture (e.g., `linux/amd64`).
    
- **`which go`** Troubleshoots path issues by showing the location of the Go executable if `go version` fails.
    
- **`go mod init <module_name>`** Initializes a new Go project in the current directory and creates a `go.mod` file to track module requirements and dependencies.
    
- **`go fmt ./...`** Automatically formats all Go source files in the current directory and subdirectories to match standard Go whitespace and indentation rules. Run this before compiling or committing code.
    
- **`go vet ./...`** Scans syntactically valid code for common logical bugs or mistakes (e.g., missing placeholder arguments in `fmt.Printf`) that the compiler won't catch.
    
- **`go build`** Compiles the Go code into a single native executable binary in the current directory. The binary name defaults to the module name.
    
- **`go build -o <custom_name>`** Compiles the code and outputs the executable with the specified custom name.

### Makefiles

