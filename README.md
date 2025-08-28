# GitHub Actions Workflow Explanation

## Overview

This GitHub Actions workflow is a Continuous Integration (CI) pipeline that automatically builds and tests a Go application whenever code is pushed to the repository. The workflow creates cross-platform builds (Linux and Windows) and runs platform-specific tests.

## Workflow Structure

### Basic Configuration

```yaml
name: artifact
on: [push]
```

- **name**: Sets the workflow name to "artifact"
- **on: [push]**: Triggers the workflow on every push to any branch

### Environment Variables

```yaml
env:
  FILE_NAME: hello-server
```

- Defines a global environment variable `FILE_NAME` set to "hello-server"
- This variable can be referenced throughout the workflow using `${{ env.FILE_NAME }}`

## Jobs Breakdown

The workflow consists of three jobs that run in a specific order:

### 1. Build Job

**Purpose**: Compiles the Go application for both Linux and Windows platforms

```yaml
build:
  name: Build
  runs-on: ubuntu-latest
```

**Steps Explained**:

1. **Check out code**
   ```yaml
   - name: Check out code
     uses: actions/checkout@v4
   ```
   - Downloads the repository source code to the runner

2. **Build for Linux**
   ```yaml
   - name: Build ${{ env.FILE_NAME }} for ubuntu-latest
     run: go build ${{ env.FILE_NAME }}.go
   ```
   - Compiles `hello-server.go` for Linux (default GOOS/GOARCH)
   - Creates executable named `hello-server`

3. **Build for Windows**
   ```yaml
   - name: Build ${{ env.FILE_NAME }} for windows-latest
     run: GOOS=windows GOARCH=amd64 go build ${{ env.FILE_NAME }}.go
   ```
   - Cross-compiles for Windows using environment variables
   - `GOOS=windows`: Target operating system
   - `GOARCH=amd64`: Target architecture (64-bit)
   - Creates executable named `hello-server.exe`

4. **Upload Linux Artifact**
   ```yaml
   - name: Upload artifact for linux
     uses: actions/upload-artifact@v4
     with:
       name: linux
       path: ./${{ env.FILE_NAME }}
   ```
   - Uploads the Linux executable as an artifact named "linux"

5. **Upload Windows Artifact**
   ```yaml
   - name: Upload artifact for windows
     uses: actions/upload-artifact@v4
     with:
       name: windows
       path: ./${{ env.FILE_NAME }}.exe
   ```
   - Uploads the Windows executable as an artifact named "windows"

### 2. Test Linux Job

**Purpose**: Tests the Linux build on an Ubuntu runner

```yaml
test-linux:
  name: Test Linux
  runs-on: [ubuntu-latest]
  needs: [build]
```

- **needs: [build]**: Ensures this job only runs after the build job completes successfully

**Steps**:

1. **Check out code**: Gets the repository code (needed for test scripts)
2. **Download artifact**: Downloads the Linux executable from the build job
3. **Run tests**: Executes `./test.sh` script to test the application

### 3. Test Windows Job

**Purpose**: Tests the Windows build on a Windows runner

```yaml
test-windows:
  name: Test Windows
  runs-on: [windows-latest]
  needs: [build]
```

**Steps**:

1. **Check out code**: Gets the repository code
2. **Download artifact**: Downloads the Windows executable from the build job
3. **Run tests**: Directly executes the Windows executable

## Workflow Execution Flow

```
Push Event
    ↓
Build Job (Ubuntu)
    ├── Compile for Linux
    ├── Compile for Windows
    ├── Upload Linux artifact
    └── Upload Windows artifact
    ↓
Test Jobs (Parallel)
    ├── Test Linux (Ubuntu)
    └── Test Windows (Windows)
```

## Key GitHub Actions Concepts

### Actions Used

- **actions/checkout@v4**: Official action to download repository code
- **actions/upload-artifact@v4**: Stores files to be shared between jobs
- **actions/download-artifact@v4**: Retrieves previously uploaded artifacts

### Runners

- **ubuntu-latest**: Linux-based virtual machine
- **windows-latest**: Windows-based virtual machine

### Cross-Compilation

The workflow demonstrates Go's cross-compilation capabilities:
- Uses `GOOS` and `GOARCH` environment variables
- Builds Windows executable from Linux runner
- No need for separate Windows build environment
