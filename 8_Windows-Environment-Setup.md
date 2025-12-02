# Windows Environment Setup Guide

This guide provides Windows-specific instructions for setting environment variables and running commands throughout the course.

## Quick Reference Table

| Task              | Unix/macOS         | Windows (PowerShell)       |
| ----------------- | ------------------ | -------------------------- |
| Set variable      | `export VAR=value` | `$env:VAR = "value"`       |
| Use variable      | `${VAR}` or `$VAR` | `$env:VAR` or `${env:VAR}` |
| Current directory | `./` or `$(pwd)`   | `.\` or `${PWD}`           |
| Username          | `${USER}`          | `$env:USERNAME`            |
| Line continuation | `\`                | `` ` ``                    |
| Path separator    | `/`                | `\` or `/` (both work)     |

## Environment Variables

### Setting environment variables in PowerShell

Windows PowerShell uses a different syntax than Unix-based shells (bash/zsh). Instead of `export`, use the `$env:` prefix:

#### Unix/macOS (bash/zsh)

```bash
export VARIABLE_NAME="value"
```

#### Windows (PowerShell)

```powershell
$env:VARIABLE_NAME = "value"
```

### Using environment variables

Once set, you can reference environment variables in PowerShell using `$env:VARIABLE_NAME`:

```powershell
# Setting a variable
$env:RESOURCE_GROUP = "cyberphysische-systeme"

# Using the variable
echo $env:RESOURCE_GROUP
```

### Persisting Environment Variables

**Important:** Environment variables set with `$env:` are only available in the current PowerShell session. When you close the terminal, they are lost.

## Command Line Differences

### Line Continuation

#### Unix/macOS (bash/zsh)

Uses backslash `\` for line continuation:

```bash
ibmcloud ce app create --name mosquitto-${USER} \
  --image de.icr.io/${CR_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG} \
  --port 8083
```

#### Windows (PowerShell)

Uses backtick `` ` `` for line continuation:

```powershell
ibmcloud ce app create --name mosquitto-${env:USERNAME} `
  --image de.icr.io/${env:CR_NAMESPACE}/${env:IMAGE_NAME}:${env:IMAGE_TAG} `
  --port 8083
```

## Common Patterns

### Pattern 1: Setting Multiple Variables

**Unix/macOS:**

```bash
export RESOURCE_GROUP=cyberphysische-systeme
export CR_NAMESPACE=hslu-cyberphysische-systeme
export IMAGE_NAME=mosquitto-${USER}
export IMAGE_TAG=1.0
```

**Windows (PowerShell):**

```powershell
$env:RESOURCE_GROUP = "cyberphysische-systeme"
$env:CR_NAMESPACE = "hslu-cyberphysische-systeme"
$env:IMAGE_NAME = "mosquitto-${env:USERNAME}"
$env:IMAGE_TAG = "1.0"
```

### Pattern 2: Using Variables in Commands

**Unix/macOS:**

```bash
ibmcloud target -g ${RESOURCE_GROUP}
podman tag ${IMAGE_NAME}:${IMAGE_TAG} de.icr.io/${CR_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG}
```

**Windows (PowerShell):**

```powershell
ibmcloud target -g ${env:RESOURCE_GROUP}
podman tag ${env:IMAGE_NAME}:${env:IMAGE_TAG} de.icr.io/${env:CR_NAMESPACE}/${env:IMAGE_NAME}:${env:IMAGE_TAG}
```

### Pattern 3: Multi-line commands with volume mounts

**Unix/macOS:**

```bash
podman run -d \
  -v ./:/home/mosquitto/passwords:ro \
  -v ./:/home/mosquitto/acl:ro \
  -v ./:/home/mosquitto/config:ro \
  -p 1883:1883 -p 8083:8083 \
  --name mosquitto \
  mosquitto-${USER}:1.0
```

**Windows (PowerShell):**

```powershell
podman run -d `
  -v "${PWD}:/home/mosquitto/passwords:ro" `
  -v "${PWD}:/home/mosquitto/acl:ro" `
  -v "${PWD}:/home/mosquitto/config:ro" `
  -p 1883:1883 `
  -p 8083:8083 `
  --name mosquitto `
  mosquitto-${env:USERNAME}:1.0
```
