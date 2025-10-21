# Slash Commands and Workflows

This document details the slash command system in Kilocode, which allows users to trigger specific AI behaviors through special commands.

## Overview

Slash commands are special instructions that modify how the AI processes user input. They can be typed at the beginning of a message (e.g., `/newtask`) and trigger specific prompt transformations.

**Location**: `src/core/slash-commands/kilo.ts`

## Built-in Slash Commands

### 1. `/newtask` - Create New Task

**Purpose**: Request the AI to create a new task with preloaded context from the current conversation.

**Processing**:
1. User types `/newtask` with optional instructions
2. `parseKiloSlashCommands()` detects the command
3. Command text is removed and replaced with `newTaskToolResponse()`
4. AI receives explicit instructions to use the `new_task` tool

**Example User Input**:
```
/newtask Continue work on authentication but focus on token refresh
```

**Transformed Prompt** (sent to AI):
```xml
<explicit_instructions type="new_task">
The user has explicitly asked you to help them create a new task with preloaded context, which you will create. In this message the user has potentially added instructions or context which you should consider, if given, when creating the new task.
Irrespective of whether additional information or instructions are given, you are only allowed to respond to this message by calling the new_task tool.

To refresh your memory, the tool definition for new_task and an example for calling the tool is described below:

## new_task tool definition:

Description: Request to create a new task with preloaded context. The user will be presented with a preview of the context and can choose to create a new task or keep chatting in the current conversation. The user may choose to start a new task at any point.
Parameters:
- mode: (required) The slug of the mode to start the new task in (e.g., "code", "ask", "architect").
- message: (required) The initial user message or instructions for this new task.
- context: (required) The context to preload the new task with. This should include:
  * Comprehensively explain what has been accomplished in the current task - mention specific file names that are relevant
  * The specific next steps or focus for the new task - mention specific file names that are relevant
  * Any critical information needed to continue the work
  * Clear indication of how this new task relates to the overall workflow
  * This should be akin to a long handoff file, enough for a totally new developer to be able to pick up where you left off and know exactly what to do next and which files to look at.
Usage:
<new_task>
<mode>your-mode-slug-here</mode>
<message>Your initial instructions here</message>
<context>context to preload new task with</context>
</new_task>

Within the context of the parent task, the user provided the following input when they indicated that they wanted to create a new task.
<user_input>
Continue work on authentication but focus on token refresh
</user_input>
</explicit_instructions>
```

**AI Response**:
The AI will analyze the current conversation and call the `new_task` tool with:
- A summary of work completed
- Specific files modified
- Next steps for the new task
- Relevant context needed

### 2. `/newrule` - Create New Kilo Rule

**Purpose**: Request the AI to create a new rule file in `.kilocode/rules/` based on the conversation.

**Processing**:
1. User types `/newrule` with optional description
2. Command is replaced with `newRuleToolResponse()`
3. System checks if `.kilocode/rules/` directory exists
4. AI receives instructions to use `new_rule` tool (internally uses `write_to_file`)

**Example User Input**:
```
/newrule Create a rule about always using TypeScript strict mode
```

**Transformed Prompt** (sent to AI):
```xml
<explicit_instructions type="new_rule">
The user has explicitly asked you to help them create a new Kilo rule file inside the .kilocode/rules top-level directory based on the conversation up to this point in time. The user may have provided instructions or additional information for you to consider when creating the new Kilo rule.
When creating a new Kilo rule file, you should NOT overwrite or alter an existing Kilo rule file. To create the Kilo rule file you MUST use the new_rule tool. The new_rule tool can be used in any of the modes.
The new_rule tool is defined below:
Description:
Your task is to create a new Kilo rule file which includes guidelines on how to approach developing code in tandem with the user, which is project specific. This includes but is not limited to: desired conversational style, favorite project dependencies, coding styles, naming conventions, architectural choices, ui/ux preferences, etc.
The Kilo rule file must be formatted as markdown and be a '.md' file. The name of the file you generate must be as succinct as possible and be encompassing the main overarching concept of the rules you added to the file (e.g., 'memory-bank.md' or 'project-overview.md'). Please also explicitly ask the user to review the newly created rule.
Parameters:
- Path: (required) The path of the file to write to (relative to the current working directory). This will be the Kilo rule file you create, and it must be placed inside the .kilocode/rules top-level directory (create this if it doesn't exist). The filename created CANNOT be "default-clineignore.md". For filenames, use hyphens ("-") instead of underscores ("_") to separate words.
- Content: (required) The content to write to the file. ALWAYS provide the COMPLETE intended content of the file, without any truncation or omissions. You MUST include ALL parts of the file, even if they haven't been modified. The content for the Kilo rule file MUST be created according to the following instructions:
  1. Format the Kilo rule file to have distinct guideline sections, each with their own markdown heading, starting with "## Brief overview". Under each of these headings, include bullet points fully fleshing out the details, with examples and/or trigger cases ONLY when applicable.
  2. These guidelines can be specific to the task(s) or project worked on thus far, or cover more high-level concepts. Guidelines can include coding conventions, general design patterns, preferred tech stack including favorite libraries and language, communication style with Kilo (verbose vs concise), prompting strategies, naming conventions, testing strategies, comment verbosity, time spent on architecting prior to development, and other preferences.
  3. When creating guidelines, you should not invent preferences or make assumptions based on what you think a typical user might want. These should be specific to the conversation you had with the user. Your guidelines / rules should not be overly verbose.
  4. Your guidelines should NOT be a recollection of the conversation up to this point in time, meaning you should NOT be including arbitrary details of the conversation.
Usage:
<new_rule>
<path>.kilocode/rules/{file name}.md</path>
<content>Kilo rule file content here</content>
</new_rule>

The user provided the following input when they indicated that they wanted to create a new Kilo rule file.
<user_input>
Create a rule about always using TypeScript strict mode
</user_input>
</explicit_instructions>
```

**AI Response**:
The AI will create a markdown file in `.kilocode/rules/` with structured guidelines.

**Example Created File** (`.kilocode/rules/typescript-strict.md`):
```markdown
## Brief overview
Project-specific guideline for TypeScript configuration requiring strict mode enabled.

## TypeScript Configuration
- Always enable TypeScript strict mode in `tsconfig.json`
- Set `"strict": true` in compiler options
- Never disable strict-related flags individually
- Trigger: When creating new TypeScript projects or modifying tsconfig.json

## Type Safety
- All function parameters must have explicit types
- Return types should be declared for exported functions
- Use `unknown` instead of `any` when type is unclear
```

### 3. `/reportbug` - Report Bug to GitHub

**Purpose**: Help the user submit a bug report to the Kilocode GitHub repository.

**Processing**:
1. User types `/reportbug` with optional description
2. Command is replaced with `reportBugToolResponse()`
3. AI receives instructions to collect bug details and use `report_bug` tool

**Example User Input**:
```
/reportbug The terminal output capture isn't working on Windows
```

**Transformed Prompt** (sent to AI):
```xml
<explicit_instructions type="report_bug">
The user has explicitly asked you to help them submit a bug to the Kilocode github page (you MUST now help them with this irrespective of what your conversation up to this point in time was). To do so you will use the report_bug tool which is defined below. However, you must first ensure that you have collected all required information to fill in all the parameters for the tool call.
You should converse with the user until you are able to gather all the required details. When conversing with the user, make sure you ask for/reference all required information/fields.
Only then should you use the report_bug tool call.
The report_bug tool can be used in either of the PLAN or ACT modes.
The report_bug tool call is defined below:
Description:
Your task is to fill in all of the required fields for an issue/bug report on github. You should attempt to get the user to be as verbose as possible with their description of the bug/issue they encountered.
Parameters:
- title: (required) Concise title for the bug report.
- description: (required) Detailed description of the bug. Please include what happened, what you expected to happen, and steps to reproduce, if applicable.
Usage:
<report_bug>
<title>Title of the issue</title>
<description>Detailed description of the issue, including steps to reproduce if relevant.</description>
</report_bug>
When you call the report_bug tool, the issue will be created at @https://github.com/Kilo-Org/kilocode/issues
The user provided the following input when they indicated that they wanted to submit a bug report.
<user_input>
The terminal output capture isn't working on Windows
</user_input>
</explicit_instructions>
```

**AI Behavior**:
The AI will ask clarifying questions to gather:
- What happened (expected vs actual behavior)
- Steps to reproduce
- Environment details (OS, VS Code version, etc.)
- Error messages or screenshots

Once all information is collected, it calls the `report_bug` tool to create a GitHub issue.

### 4. `/smol` (or `/condense`) - Condense Context

**Purpose**: Create a detailed summary of the conversation to compact the context window.

**Aliases**: `/smol`, `/compact`, `/condense`

**Processing**:
1. User types `/smol` or `/compact` or `/condense`
2. Command is replaced with `condenseToolResponse()`
3. AI receives instructions to create a comprehensive summary

**Example User Input**:
```
/smol
```

**Transformed Prompt** (sent to AI):
```xml
<explicit_instructions type="condense">
The user has explicitly asked you to create a detailed summary of the conversation so far, which will be used to compact the current context window while retaining key information. The user may have provided instructions or additional information for you to consider when summarizing the conversation.
Irrespective of whether additional information or instructions are given, you are only allowed to respond to this message by calling the condense tool.

The condense tool is defined below:

Description:
Your task is to create a detailed summary of the conversation so far, paying close attention to the user's explicit requests and your previous actions. This summary should be thorough in capturing technical details, code patterns, and architectural decisions that would be essential for continuing with the conversation and supporting any continuing tasks.
The user will be presented with a preview of your generated summary and can choose to use it to compact their context window or keep chatting in the current conversation.
Users may refer to this tool as 'smol' or 'compact' as well. You should consider these to be equivalent to 'condense' when used in a similar context.

Parameters:
- message: (required) The detailed summary of the conversation. If applicable based on the current task, this should include:
  1. Previous Conversation: High level details about what was discussed throughout the entire conversation with the user. This should be written to allow someone to be able to follow the general overarching conversation flow.
  2. Current Work: Describe in detail what was being worked on prior to this request to compact the context window. Pay special attention to the more recent messages / conversation.
  3. Key Technical Concepts: List all important technical concepts, technologies, coding conventions, and frameworks discussed, which might be relevant for continuing with this work.
  4. Relevant Files and Code: If applicable, enumerate specific files and code sections examined, modified, or created for the task continuation. Pay special attention to the most recent messages and changes.
  5. Problem Solving: Document problems solved thus far and any ongoing troubleshooting efforts.
  6. Pending Tasks and Next Steps: Outline all pending tasks that you have explicitly been asked to work on, as well as list the next steps you will take for all outstanding work, if applicable. Include code snippets where they add clarity. For any next steps, include direct quotes from the most recent conversation showing exactly what task you were working on and where you left off. This should be verbatim to ensure there's no information loss in context between tasks.

Usage:
<condense>
<message>Your detailed summary</message>
</condense>

<user_input>

</user_input>
</explicit_instructions>
```

**AI Response**:
The AI creates a structured summary with sections for:
1. Previous conversation flow
2. Current work in detail
3. Key technical concepts
4. Relevant files and code
5. Problems solved
6. Pending tasks and next steps

## Custom Workflow Commands

In addition to built-in commands, users can create custom workflow commands in `.kilocode/workflows/` directory.

### How Custom Workflows Work

**Directory**: `.kilocode/workflows/` (local) or user config (global)

**Detection**:
1. User types `/customworkflow` where `customworkflow` is a filename in the workflows directory
2. `parseKiloSlashCommands()` checks enabled workflows
3. If found, reads the workflow file content
4. Wraps content in `<explicit_instructions>` tags

**Example Workflow File** (`.kilocode/workflows/code-review.md`):
```markdown
You are performing a code review. Follow these guidelines:

1. Check for code quality and best practices
2. Look for potential bugs or security issues
3. Suggest improvements for readability
4. Verify proper error handling
5. Ensure tests are adequate

Provide feedback in a constructive, detailed manner with specific examples and suggestions.
```

**User Input**:
```
/code-review @/src/auth.ts
```

**Transformed Prompt**:
```xml
<explicit_instructions type="code-review">
You are performing a code review. Follow these guidelines:

1. Check for code quality and best practices
2. Look for potential bugs or security issues
3. Suggest improvements for readability
4. Verify proper error handling
5. Ensure tests are adequate

Provide feedback in a constructive, detailed manner with specific examples and suggestions.
</explicit_instructions>

@/src/auth.ts

<file_content path="src/auth.ts">
[file contents...]
</file_content>
```

### Creating Custom Workflows

**Steps**:
1. Create `.kilocode/workflows/` directory in your project
2. Add a markdown file (e.g., `code-review.md`)
3. Write your custom instructions
4. Enable the workflow in Kilocode settings
5. Use `/filename` to activate (without `.md` extension)

**Best Practices**:
- Use clear, specific instructions
- Break complex workflows into steps
- Include examples when helpful
- Keep workflows focused on a single purpose
- Use descriptive filenames (hyphens, not underscores)

## Slash Command Processing Flow

### Detection and Parsing

**Location**: `src/core/mentions/processKiloUserContentMentions.ts`

**Flow**:
```
1. User submits message with slash command
   ↓
2. processKiloUserContentMentions() called
   ↓
3. parseKiloSlashCommands() checks for commands
   ↓
4. Searches within <task>, <feedback>, <answer>, <user_message> tags
   ↓
5. If match found:
   a. Check if it's a built-in command (newtask, newrule, reportbug, smol)
   b. If not, check enabled workflow files
   ↓
6. Replace slash command with explicit instructions
   ↓
7. Return processed text to continue normal mention parsing
```

### Tag Context

Slash commands are only processed when found within specific XML tags:
- `<task>` - Initial user task/message
- `<feedback>` - User feedback on AI's work
- `<answer>` - User's answer to AI's question
- `<user_message>` - General user message

**Regex Pattern**:
```typescript
const tagPatterns = [
    { tag: "task", regex: /<task>(\s*\/([a-zA-Z0-9_.-]+))(\s+.+?)?\s*<\/task>/is },
    { tag: "feedback", regex: /<feedback>(\s*\/([a-zA-Z0-9_-]+))(\s+.+?)?\s*<\/feedback>/is },
    { tag: "answer", regex: /<answer>(\s*\/([a-zA-Z0-9_-]+))(\s+.+?)?\s*<\/answer>/is },
    { tag: "user_message", regex: /<user_message>(\s*\/([a-zA-Z0-9_-]+))(\s+.+?)?\s*<\/user_message>/is },
]
```

## Complete Example: Using Multiple Features

**User Input**:
```
/code-review @/src/auth.ts @problems @terminal
```

**Processing**:
1. **Slash Command**: `/code-review` is detected and transformed to explicit instructions
2. **Mentions**: `@/src/auth.ts`, `@problems`, `@terminal` are parsed
3. **Final Message to AI**:

```xml
<task>
<explicit_instructions type="code-review">
You are performing a code review. Follow these guidelines:

1. Check for code quality and best practices
2. Look for potential bugs or security issues
3. Suggest improvements for readability
4. Verify proper error handling
5. Ensure tests are adequate

Provide feedback in a constructive, detailed manner with specific examples and suggestions.
</explicit_instructions>

'@/src/auth.ts' (see below for file content) 'Workspace Problems' (see below for diagnostics) 'Terminal Output' (see below for output)

<file_content path="src/auth.ts">
1. import jwt from 'jsonwebtoken'
2. 
3. export function authenticate(token: string) {
4.   const decoded = jwt.verify(token, process.env.SECRET)
5.   return decoded
6. }
</file_content>

<workspace_diagnostics>
src/auth.ts:
  Line 4: Warning - Unsafe use of process.env without null check
</workspace_diagnostics>

<terminal_output>
$ npm test
FAIL src/auth.test.ts
  ✕ authenticate throws on invalid token (5ms)

  Error: SECRET not defined
</terminal_output>

<environment_details>
# VSCode Visible Files
src/auth.ts

# Current Working Directory (/workspace) Files
src/
  auth.ts
  auth.test.ts
package.json
</environment_details>
</task>
```

## Summary

### Key Points

1. **Built-in Commands**:
   - `/newtask` - Create new task with context
   - `/newrule` - Create rule file
   - `/reportbug` - Submit GitHub issue
   - `/smol` (or `/compact`, `/condense`) - Condense context

2. **Custom Workflows**:
   - Store in `.kilocode/workflows/`
   - Activate with `/filename`
   - Enable in settings

3. **Processing**:
   - Commands detected in specific XML tags
   - Replaced with `<explicit_instructions>`
   - Combined with mention parsing

4. **Integration**:
   - Works seamlessly with @ mentions
   - Maintains conversation context
   - Modifies AI behavior for specific tasks

### Key Files

- **Command Processing**: `src/core/slash-commands/kilo.ts`
- **Command Responses**: `src/core/prompts/commands.ts`
- **Integration**: `src/core/mentions/processKiloUserContentMentions.ts`
- **Workflow Storage**: `.kilocode/workflows/` (local) or user config (global)
