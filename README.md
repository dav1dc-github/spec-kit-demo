# spec-kit-demo

A Playground for Demonstrating Spec-Driven Development with GitHub Spec Kit

## What is Spec-Driven Development?

Spec-Driven Development (SDD) is a methodology where you first capture clear specifications of what you want to build and why, then create technical plans, break requirements into actionable tasks, and finally implement these using AI coding agents. This approach ensures alignment between intent and implementation while leveraging AI tools effectively.

## What is GitHub Spec Kit?

[GitHub Spec Kit](https://github.com/github/spec-kit) is an open-source toolkit designed to bring clarity, structure, and AI assistance to spec-driven development. It helps teams:

- Define what to build with clear specifications
- Create detailed technical implementation plans
- Break work into manageable tasks
- Maintain living documentation that evolves with the project
- Collaborate effectively with AI coding agents

## Getting Started

### Installation

Install the Specify CLI tool using one of the following methods:

**Using uv (recommended):**
```sh
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

**Verify installation:**
```sh
specify --version
```

### Initialize a New Project

Create a new spec-driven project:

```sh
specify init <PROJECT_NAME>
```

Or initialize with a specific AI agent:

```sh
specify init <PROJECT_NAME> --ai copilot
```
OR
```sh
cd <PROJECT_NAME> && specify init --ai copilot --here
```

This sets up `.github` and `.specify` directories for your specification files and plans.

## The Spec-Driven Development Workflow

### 1. **Define Your Constitution**

Define what our over archaring project principles that the AI should adhere to when making decisions about the spec, plan and tasks for your development project

```
/speckit.constitution
Create principles focused on code quality, testing standards, user experience consistency, and performance requirements
Most important are concepts of secure coding and utilising test driven development (TDD)
```

### 2. **Create Your Specification**

Define what you want to build and why, focusing on goals, user scenarios, and desired outcomes. Use the `/speckit.specify` slash command:

```
/speckit.specify
Build an app to manage tasks with Kanban boards, multiple users, and real-time commenting.
```

**Key Points:**
- Focus on the "what" and "why", not the "how"
- Describe functional requirements and user scenarios
- Avoid technical implementation details at this stage

### 3. **Plan the Implementation**

Once the specification is ready, generate the technical plan using the `/speckit.plan` command:

```
/speckit.plan
Use React with TypeScript; backend with Node.js & PostgreSQL.
```

This step details:
- Architectural choices
- Technology stack
- Technical approach
- Integration points

### 4. **Optional Commands**

Consider using the following optional commands to further refine your spoecification:

```
│  Optional commands that you can use for your specs (improve quality & confidence)                                 │
│                                                                                                                   │
│  ○ /speckit.clarify (optional) - Ask structured questions to de-risk ambiguous areas before planning (run before  │
│  /speckit.plan if used)                                                                                           │
│  ○ /speckit.analyze (optional) - Cross-artifact consistency & alignment report (after /speckit.tasks, before      │
│  /speckit.implement)                                                                                              │
│  ○ /speckit.checklist (optional) - Generate quality checklists to validate requirements completeness, clarity,    │
│  and consistency (after /speckit.plan)    
```

### 5. **Break Down Into Tasks**

Divide the spec and technical plan into concrete, actionable tasks using the `/tasks` command:

```
/speckit.tasks
# Outputs a list of development tasks, requirements, and milestones.
```

Each task aligns with the guiding specification and can be assigned to developers or AI agents.

### 6. **Iterative Implementation**

Use AI coding agents (like GitHub Copilot, Claude, Cursor, or Gemini) to implement each task. The living specification and technical plan guide the implementation process.

### 7. **Review and Refactor**

Specifications in Spec Kit are living documents. Update them as new insights or requirements arise to ensure specifications and actual code remain aligned.

## Key Features

- **Specify CLI:** Easy project setup and scaffolding
- **Constitution Files:** Define non-negotiable team and project principles
- **Markdown Artifacts:** All specs, plans, and tasks are generated as markdown for easy sharing and version control
- **Cross-Platform:** Works on Windows, macOS, and Linux
- **AI Agent Integration:** Supports multiple AI frameworks and agents (Copilot, Claude, Cursor, Gemini, etc.)
- **Slash Commands:** Fast interaction for specifying, planning, and breaking down work

## Advantages of Spec-Driven Development

- **Reduces Ambiguity:** Clear separation of what/why vs. how
- **Improves Collaboration:** Everyone refers to the same living source of truth
- **Better AI Results:** AI agents generate code closely aligned to verified requirements and architecture
- **Flexible and Scalable:** Adapt to project changes without losing alignment or context
- **Documentation by Default:** Specs serve as up-to-date project documentation

## Best Practices

1. **Start with Why:** Always begin by documenting the purpose and goals
2. **Keep Specs Living:** Update specifications as requirements evolve
3. **Separate Concerns:** Keep the specification (what/why) distinct from the technical plan (how)
4. **Iterate Often:** Review and refine specs based on implementation learnings
5. **Use Constitution Files:** Define team principles and non-negotiables upfront
6. **Leverage AI Effectively:** Use AI agents for implementation while humans focus on specification and review

## Resources

- [Official Spec Kit Documentation](https://github.github.com/spec-kit/)
- [Spec Kit Quick Start Guide](https://github.github.com/spec-kit/quickstart.html)
- [Spec Kit Installation Guide](https://github.github.com/spec-kit/installation.html)
- [GitHub Spec Kit Repository](https://github.com/github/spec-kit)
- [Spec Kit Community Site](https://speckit.org/)
- [GitHub Blog: Spec-driven development with AI](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- [LogRocket: Exploring spec-driven development](https://blog.logrocket.com/github-spec-kit/)

## Example Workflow

Here's a complete example of using Spec Kit:

```sh
# 1. Initialize project
specify init my-task-manager --ai copilot

# 2. Navigate to project
cd my-task-manager

# 3. Create specification
/speckit.specify
Build a task management system with:
- User authentication
- Project workspaces
- Kanban board view
- Real-time collaboration
- Task comments and attachments

# 4. Create technical plan
/speckit.plan
Frontend: React + TypeScript + Tailwind CSS
Backend: Node.js + Express + PostgreSQL
Authentication: JWT tokens
Real-time: Socket.io
Deployment: Docker containers on AWS

# 5. Generate tasks
/speckit.tasks
# This will create a breakdown of implementation tasks

# 6. Implement tasks iteratively
# Work through each task with your AI coding agent
# Update specs and plans as you learn and requirements evolve
/speckit.implement
```

## Contributing

This repository serves as a demonstration and playground for spec-driven development practices. Feel free to experiment with different specifications, plans, and implementation approaches.

## License

This is a demonstration repository. See the main [GitHub Spec Kit repository](https://github.com/github/spec-kit) for licensing information.
