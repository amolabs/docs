# AMO Coding Guide

## General
### Editing style
- Indentation with tab in most cases
- Tab size 4 (i.e. indentation size is 4)
- Unix format (line ending with LF not CR LF)
    - **Be careful when you use Windows PC**
- UTF-8 encoding

### File mode
- No exe perm for source code files (go files)
    - **Be careful when you use Windows PC**

## Go
### Naming style
- See `go fmt`
- Use camelCase and CamelCase for Go

### Misc.
- Use `go fmt` before commit. This will enforce:
    - Indentation with tabs
    - Alignment with blanks (= spaces)
    - Order of imported packages

## JavaScript
### Naming style
- Use camelCase for Javascript

## JSON
### Naming style
- Use snake_case for JSON
    - This also applies to field tags for members of a Go `struct`.
    ```go
    type MyStruct struct {
        MemberName string `json:"member_name"`
    }
    ```

## RPC spec
### Naming style
- Use snake_case for RPC keywords
