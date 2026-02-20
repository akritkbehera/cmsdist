# Scram Tool Files Documentation

## Overview

`get_tools` is a helper Bash script used by the **(cmsdist)**. Its primary purpose is to **collect XML tool definition files**, copy them into the appropriate *selected* directory, and perform environment variable substitution inside those XML files.
---

## Usage

```bash
get_tools <tool‑install‑dir> <tool‑version> <directory‑to‑install‑toolfiles> <toolname>
```

| Argument | Description |
|----------|-------------|
| `tool‑install‑dir` | Root directory of the tool source tree (e.g. `$CMSSW_BASE` or a path to a tool checkout). |
| `tool‑version` | Version string of the tool being installed. |
| `directory‑to‑install‑toolfiles` | Destination directory where the processed XML files will be placed. |
| `toolname` | Logical name of the tool (e.g. `gcc`, `python`). |

The script sets a number of environment variables based on these arguments (see *Environment Variables* below) and then copies XML files from two possible locations:

1. The tool‑specific source directory (`$SCRAM_TOOL_SOURCE_DIR`).
2. The generic `etc/scram.d` directory inside the tool root.

---

## Environment Variables

| Variable | Value | Meaning |
|----------|-------|---------|
| `TOOL_ROOT` | `$1` | Root of the tool source tree. |
| `TOOL_VERSION` | `$2` | Version string supplied on the command line. |
| `TOOLFILES_INSTALL_DIR` | `$3` | Destination directory for the processed XML files. |
| `TOOL` | `$4` | Logical name of the tool. |
| `TOOL_FILENAME` | filename.xml | filename with `.xml` stripped off |
| `TOOL_BASE` | Upper‑case version of `TOOL` with hyphens converted to underscores, suffixed with `_BASE`. |
| `SCRAM_TOOLS_BIN_DIR` | Directory containing the `get_tools` script itself. |
| `SCRAM_TOOL_SOURCE_DIR` | Path to the tool‑specific source directory (`../tools/<toolname>` relative to the script). |

These variables are exported so that any *env.sh* script inside the tool source directory can use them.


### Example: OpenBLAS XML

**Original `openblas.xml` (excerpt)**

```xml
<tool name="@TOOL_FILENAME@" version="@TOOL_VERSION@" revision="1">
  <lib name="openblas"/>
  <client>
    <environment name="$@TOOL_BASE@" default="@TOOL_ROOT@"/>
    <environment name="INCLUDE" default="$OPENBLAS_BASE/include"/>
    <environment name="LIBDIR" default="$OPENBLAS_BASE/lib"/>
    <environment name="BINDIR" default="$OPENBLAS_BASE/bin"/>
  </client>
  <runtime name="OPENBLAS_NUM_THREADS" value="1"/>
</tool>
```

**After processing (with `TOOL=OpenBLAS`, `TOOL_VERSION=0.3.21`, `TOOL_ROOT=/opt/tools/OpenBLAS`)**

```xml
<tool name="OpenBLAS" version="0.3.21" revision="1">
  <lib name="openblas"/>
  <client>
    <environment name="OPENBLAS_BASE" default="/opt/tools/OpenBLAS"/>
    <environment name="INCLUDE" default="$OPENBLAS_BASE/include"/>
    <environment name="LIBDIR" default="$OPENBLAS_BASE/lib"/>
    <environment name="BINDIR" default="$OPENBLAS_BASE/bin"/>
  </client>
  <runtime name="OPENBLAS_NUM_THREADS" value="1"/>
</tool>
```

The placeholders `@TOOL_FILENAME@`, `@TOOL_VERSION@`, `@TOOL_BASE@`, and `@TOOL_ROOT@` are replaced by the exported environment variables, yielding a concrete tool definition ready for consumption.
```
