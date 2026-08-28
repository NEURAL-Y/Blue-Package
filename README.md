<div align="center">

<img src="https://raw.githubusercontent.com/NEURAL-Y/Blue-Package/main/public/Blue_package.png" alt="BluePackage" width="220"/>

# BluePackage

**A zero-configuration package and build workflow for C++.**

![Language](https://img.shields.io/badge/C%2B%2B-modern-blue?logo=cplusplus)
![Status](https://img.shields.io/badge/status-early%20development-yellow)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

[The Problem](#the-problem) • [The Vision](#the-vision) • [How It Works](#how-it-works) • [Architecture](#architecture) • [Roadmap](#roadmap)

</div>

---

BluePackage aims to make using C++ libraries feel simple: install a package, `#include` it, build and run — without hand-writing `CMakeLists.txt` entries, `find_package()` calls, include/library paths, linker flags, or platform-specific setup for every dependency.

## The Problem

Using an external library in C++ is often only the beginning. The project still has to know how to **find**, **compile**, and **link** it — every time, for every platform:

```text
CMakeLists.txt · find_package() · target_link_libraries()
include directories · library paths · compiler flags · platform configuration
```

## The Vision

```bash
blue install qt
```

```cpp
#include <QApplication>
#include <QWidget>
```

```bash
blue run
```

BluePackage handles dependency integration and build configuration automatically — so a real project looks like this:

```bash
blue init my-game
cd my-game

blue install glfw
blue install glad

blue run
```

No manually managed `CMakeLists.txt`, include/library paths, linker configuration, compiler flags, or platform-specific setup.

## How It Works

```text
Developer
    │
    ▼
blue install qt
    │
    ▼
BluePackage
    │
    ├── Download package
    ├── Resolve dependencies
    └── Detect platform
    │
    ▼
Configure build automatically
    │
    ▼
Write C++ code → blue run → Executable
```

**Everyday workflow:** `blue search` → `blue install` → write code → `blue build` → `blue run`

## Architecture

```text
BluePackage
├── CLI                  init · search · install · remove · build · run
├── Package Manager      discovery · version management · dependency resolution
├── Build Engine         compiler detection · include management · linking · platform config
└── Package Registry     metadata · versions · dependencies · documentation
```

## Goals

- Simple package installation
- Automatic dependency integration
- Automatic compiler & platform detection
- Minimal project configuration
- Simple build and run workflow
- Modern C++ support

## Future: The Registry

```text
Developer → BluePackage CLI → BluePackage Registry → Library A / B / C ...
```

The planned registry aims to let developers search, install, publish, and version packages, resolve dependencies, and access documentation — all through the CLI.

## Important Note

BluePackage doesn't remove the fundamental steps of C++ compilation — libraries still need to be downloaded, compiled, linked, and executed. The difference is **who does the configuring**:

| | Traditional C++ | BluePackage |
|---|---|---|
| Configuration | Developer configures everything manually | BluePackage manages configuration automatically |

## Roadmap

**Phase 1 — Foundations**
Project initialization · package installation · local package management · package metadata

**Phase 2 — Build Automation**
Compiler detection · automatic include paths & linking · `build` command · `run` command

**Phase 3 — Registry**
Package registry · search · publishing · version management · dependency resolution

**Phase 4 — Platform & Community**
Windows / Linux / macOS support · community registry · extended build integration

## Status

**Early development.** BluePackage is currently a proposed C++ infrastructure project — design and architecture are still taking shape, and interfaces described here are subject to change.

## Core Philosophy

> Install a library. Use it immediately. Let the tooling handle the complexity.

## Contributing

Design discussion, prototyping, and early feedback are all welcome at this stage.

1. Fork the repo and branch from `main`.
2. Open an issue first for larger changes — the CLI and build-engine design are still settling.
3. Open a PR describing the change and motivation.

---

<div align="center">

**BluePackage** — *C++ package management without unnecessary configuration friction.*

</div>
