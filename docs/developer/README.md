# Developer Documentation

This directory contains technical documentation for Kilocode developers and contributors.

## Available Documentation

### [AI Prompts and Context Handling](./ai-prompts-and-context.md)

Comprehensive guide covering:
- **System Prompt Structure**: How the AI's foundational instructions are assembled
- **Context Handling Mechanisms**: All 8+ types of context mentions (@file, @folder, @problems, @terminal, @git, @url, @commands, @images)
- **User Input Processing**: Step-by-step flow from user input to AI request
- **Practical Examples**: Real-world scenarios showing exact prompt formats for each context type

This document is essential for understanding:
- How Kilocode feeds information to AI models
- What context is available to the AI at each stage
- How different types of mentions are processed
- The complete message format sent to AI providers

### [Slash Commands and Workflows](./slash-commands.md)

Detailed documentation covering:
- **Built-in Commands**: `/newtask`, `/newrule`, `/reportbug`, `/smol` (condense)
- **Custom Workflows**: How to create and use custom workflow commands
- **Command Processing**: How slash commands are detected and transformed
- **Integration**: How commands work with @ mentions and context

This document explains:
- How slash commands modify AI behavior
- The prompt transformations for each command
- Creating custom workflows in `.kilocode/workflows/`
- Complete examples showing commands + mentions together

## Purpose

These documents serve multiple purposes:
1. **Onboarding**: Help new developers understand how Kilocode works internally
2. **Reference**: Quick lookup for implementation details
3. **Debugging**: Understand what the AI sees to diagnose issues
4. **Extension**: Learn how to add new context types or modify existing behavior

## Contributing

When adding new features related to AI prompts or context handling, please update the relevant documentation to keep it current.
