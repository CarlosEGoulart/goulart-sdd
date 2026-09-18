## Purpose

Provides clear product documentation — README rewrite and Getting Started guide for real-world installation and use.

## ADDED Requirements

### Requirement: README serves as product landing page
The root README.md SHALL function as the project's primary entry point with installation, usage, and architecture documentation.

#### Scenario: README sections present
- **WHEN** a developer opens the README
- **THEN** it contains: overview, quick start, installation, workflow, command reference, core principles, supported agents, generated project layout, architecture, OpenSpec relationship, and roadmap sections

#### Scenario: Quick start works
- **WHEN** a developer follows the Quick Start section
- **THEN** they can initialize Goulart SDD in a new repository within 5 minutes

### Requirement: Getting Started works for external repositories
The Getting Started guide SHALL allow a user to initialize Goulart in a repository other than `goulart-sdd`.

#### Scenario: External repo initialization
- **WHEN** a user follows Getting Started in a fresh git repository
- **THEN** they successfully initialize Goulart SDD without referencing the bootstrap repository

### Requirement: Documentation reflects actual implementation
All documentation commands and examples SHALL reflect the actual implemented CLI behavior.

#### Scenario: Commands match implementation
- **WHEN** a user runs a command documented in README
- **THEN** the command exists and produces the documented output
