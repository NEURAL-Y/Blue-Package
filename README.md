# BluePackage

> A zero-configuration package and build workflow for C++.

BluePackage aims to make using C++ libraries feel simple.

```text
+--------------------------------------------------+
|                   BluePackage                    |
|       C++ Package Management Made Simpler        |
+--------------------------------------------------+
```

## The Problem

Using external libraries in C++ often requires manual configuration:

```text
+------------------------------------------+
|              C++ Development             |
+------------------------------------------+
|                                          |
|  CMakeLists.txt                          |
|  find_package()                          |
|  target_link_libraries()                 |
|  Include directories                     |
|  Library paths                           |
|  Compiler flags                          |
|  Platform configuration                  |
|                                          |
+------------------------------------------+
```

Installing a library is often only the beginning.

The project still needs to know how to find, compile, and link it.

## The Vision

Install a package:

```text
blue install qt
```

Use it directly:

```cpp
#include <QApplication>
#include <QWidget>
```

Build and run:

```text
blue run
```

BluePackage handles dependency integration and build configuration automatically.

## How It Works

```text
                  +-----------+
                  | Developer |
                  +-----+-----+
                        |
                        v
              +-------------------+
              | blue install qt   |
              +---------+---------+
                        |
                        v
              +-------------------+
              |   BluePackage     |
              +---------+---------+
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
    +----------+  +----------+  +-----------+
    | Download |  | Resolve  |  | Detect    |
    | Package  |  | Depends  |  | Platform  |
    +----------+  +----------+  +-----------+
          |             |             |
          +-------------+-------------+
                        |
                        v
              +-------------------+
              | Configure Build   |
              | Automatically     |
              +---------+---------+
                        |
                        v
              +-------------------+
              | Write C++ Code    |
              +---------+---------+
                        |
                        v
              +-------------------+
              |    blue run       |
              +---------+---------+
                        |
                        v
              +-------------------+
              |    Executable     |
              +-------------------+
```

## Example

```text
blue init my-game
cd my-game

blue install glfw
blue install glad

blue run
```

The goal is to avoid manually managing:

```text
+----------------------------+
| Manual Configuration       |
+----------------------------+
|                            |
| CMakeLists.txt             |
| Include paths              |
| Library paths              |
| Linker configuration       |
| Compiler flags             |
| Platform-specific setup    |
|                            |
+----------------------------+
```

## Architecture

```text
BluePackage
|
+-- CLI
|   |
|   +-- init
|   +-- search
|   +-- install
|   +-- remove
|   +-- build
|   +-- run
|
+-- Package Manager
|   |
|   +-- Package Discovery
|   +-- Version Management
|   +-- Dependency Resolution
|
+-- Build Engine
|   |
|   +-- Compiler Detection
|   +-- Include Management
|   +-- Library Linking
|   +-- Platform Configuration
|
+-- Package Registry
    |
    +-- Package Metadata
    +-- Package Versions
    +-- Dependencies
    +-- Documentation
```

## Goals

```text
[+] Simple package installation
[+] Automatic dependency integration
[+] Automatic compiler detection
[+] Automatic platform configuration
[+] Minimal project configuration
[+] Simple build workflow
[+] Simple project execution
[+] Modern C++ support
```

## Workflow

```text
                  +-------------+
                  |    Search   |
                  | blue search |
                  +------+------+
                         |
                         v
                  +-------------+
                  |   Install   |
                  | blue install|
                  +------+------+
                         |
                         v
                  +-------------+
                  |    Write    |
                  |  C++ Code   |
                  +------+------+
                         |
                         v
                  +-------------+
                  |    Build    |
                  | blue build  |
                  +------+------+
                         |
                         v
                  +-------------+
                  |     Run     |
                  |  blue run   |
                  +-------------+
```

## Future Vision

```text
                     Developer
                         |
                         v
              +---------------------+
              |   BluePackage CLI   |
              +----------+----------+
                         |
                         v
              +---------------------+
              | BluePackage Registry|
              +----------+----------+
                         |
              +----------+----------+
              |          |          |
              v          v          v
           Library    Library    Library
              A          B          C
```

The future registry aims to allow developers to:

```text
[+] Search packages
[+] Install packages
[+] Publish packages
[+] Manage versions
[+] Resolve dependencies
[+] Access documentation
```

## Important Note

BluePackage does not remove the fundamental requirements of C++ compilation.

Libraries still need to be:

```text
Downloaded
    |
    v
Compiled
    |
    v
Linked
    |
    v
Executed
```

The difference is:

```text
Traditional C++
       |
       v
Developer configures everything manually


BluePackage
       |
       v
BluePackage manages configuration automatically
```

## Roadmap

```text
PHASE 1
|
+-- [ ] Project initialization
+-- [ ] Package installation
+-- [ ] Local package management
+-- [ ] Package metadata


PHASE 2
|
+-- [ ] Compiler detection
+-- [ ] Automatic include paths
+-- [ ] Automatic library linking
+-- [ ] Build command
+-- [ ] Run command


PHASE 3
|
+-- [ ] Package registry
+-- [ ] Package search
+-- [ ] Package publishing
+-- [ ] Version management
+-- [ ] Dependency resolution


PHASE 4
|
+-- [ ] Windows support
+-- [ ] Linux support
+-- [ ] macOS support
+-- [ ] Community registry
+-- [ ] Extended build integration
```

## Status

```text
+-----------------------------------+
|        EARLY DEVELOPMENT          |
|                                   |
|  BluePackage is currently a       |
|  proposed C++ infrastructure      |
|  project.                         |
+-----------------------------------+
```

## Core Philosophy

```text
+----------------------------------------------+
|                                              |
|   Install a library.                         |
|                                              |
|   Use it immediately.                        |
|                                              |
|   Let the tooling handle the complexity.     |
|                                              |
+----------------------------------------------+
```

**BluePackage**

`C++ package management without unnecessary configuration friction.`
