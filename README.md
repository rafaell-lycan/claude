# Claude Code Plugin

A collection of reusable skills, commands, and hooks for Claude Code to enhance your development workflow.

## Installation

To use this plugin with Claude Code, clone this repository and reference it in your project's Claude Code settings.

## Contents

### Commands

- **[/why](commands/why.md)** - Five Whys root cause analysis for debugging and problem-solving
- **[/cleanup-branch](commands/cleanup-branch.md)** - Comprehensive branch cleanup using code review tools

### Skills

#### General Development
- **[troubleshooting](skills/troubleshooting/SKILL.md)** - Systematic debugging strategies for tests, builds, MCPs, and TypeScript errors
- **[testing-standards](skills/testing-standards/SKILL.md)** - Unit and integration testing patterns and implementation checklist

#### Backend Development (.NET/C#)
- **[dotnet8-coding-standards](skills/dotnet8-coding-standards/SKILL.md)** - Modern .NET 8 features, primary constructors, and C# 12 patterns
- **[api-design](skills/api-design/SKILL.md)** - API endpoint design, Result patterns, and REST conventions
- **[validation-strategy](skills/validation-strategy/SKILL.md)** - FluentValidation, business rules, and validation adapter patterns
- **[logging-standards](skills/logging-standards/SKILL.md)** - Structured logging with Serilog and proper log level usage
- **[error-handling](skills/error-handling/SKILL.md)** - RFC 7807 Problem Details implementation and HTTP status codes

#### Frontend Development
- **[performance-optimization](skills/performance-optimization/SKILL.md)** - React Query patterns, database optimization, debouncing, and bundle optimization

## Usage

### Commands

Commands can be invoked using the `/` prefix in Claude Code:

```
/why
/cleanup-branch
```

### Skills

Skills are automatically activated based on context or can be explicitly invoked. They provide guidance and best practices for specific development tasks.

#### Example Use Cases

**Troubleshooting:**
- When tests fail, use the troubleshooting skill for systematic debugging
- When builds break, get TypeScript error explanations
- When MCPs disconnect, get recovery strategies

**API Development:**
- Implementing new endpoints with proper Result patterns
- Setting up FluentValidation for input validation
- Designing RESTful routes with proper conventions

**Performance:**
- Optimizing React Query configurations
- Adding database indexes
- Implementing debouncing for validation

**Code Quality:**
- Structured logging throughout your application
- Proper error handling with Problem Details
- Modern .NET 8 coding standards

## Requirements

- Claude Code CLI
- For cleanup-branch: Zen MCP server with codereview tool
- For .NET skills: .NET 8+ SDK

## Contributing

Feel free to submit issues and enhancement requests!

## License

MIT License
