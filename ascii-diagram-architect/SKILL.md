---
name: ascii-diagram-architect
description: Generate high-utility, tool-agnostic ASCII diagrams and flowcharts within code comments or documentation, following the formal "Design Space of ASCII Drawings" (Hayatpur et al., 2024). Use when the user asks to "visualize," "diagram," "flowchart," or "map out" a process, data structure, or system architecture using plain text.
---

# ASCII Diagram Architect

A specialized skill for generating high-utility, tool-agnostic ASCII diagrams and flowcharts within code comments or documentation, following the formal *"Design Space of ASCII Drawings"* (Hayatpur et al., 2024).

## Trigger Instructions

Use this skill when the user asks to:
- "visualize,"
- "diagram,"
- "flowchart," or
- "map out"

a process, data structure, or system architecture using plain text.

Activate when providing code explanations where a visual representation of logic or memory layout would improve clarity.

## Core Principles

### Tool-Agnosticism

Ensure the diagram is strictly monospaced and readable in any basic text editor (IDE, Terminal, Git).

### Proximity of Information

The diagram should act as a *"visual map"* that sits directly above or beside the logic it describes.

### Structured Notation

Use consistent character sets (`+`, `-`, `|`, `v`, `>`) to define boundaries and flow.

## The Design Space Workflow

Based on the Hayatpur et al. research, define the following dimensions internally before rendering:

### 1. Content Identification

Determine what is being represented:

- **System Architecture**: High-level components and networking.
- **Data Structures**: Memory layouts, linked lists, or tree nodes.
- **Dynamic Behavior**: Step-by-step logic or state transitions.

### 2. Layout & Connectivity

- **Box-and-Line**: Use `+---+` for entities and `|` or `-` for connections.
- **Flow Direction**: Standardize on top-to-bottom or left-to-right.
- **Alignment**: Ensure all vertical pipes (`|`) align perfectly to maintain the *"box"* shape.

## Implementation Guidelines

### The "Box" Standard

Always use a 3-tier boundary for entities to ensure they are visually distinct:

```
+-----------------------+
|      ENTITY NAME      |
+-----------------------+
```

### The "Connector" Standard

Use explicit arrowheads to show directionality. Do not use ambiguous lines.

- **Vertical**: `|` followed by `v`
- **Horizontal**: `---` followed by `>`

### Annotations

Place short, descriptive text next to connectors to explain the transition or logic (e.g., `"if x > 0"` or `"API Call"`).

## Example Workflow: System Request

**Input:** *"Show me how a Load Balancer distributes traffic to three servers."*

**Output:**

```
          [ CLIENT REQUEST ]
                  |
                  v
       +-----------------------+
       |     LOAD BALANCER     |
       +----------+------------+
                  |
        +---------+---------+
        |         |         |
        v         v         v
    +-------+ +-------+ +-------+
    | SRV 1 | | SRV 2 | | SRV 3 |
    +-------+ +-------+ +-------+
```

## Constraints

1. **Do Not** use non-standard Unicode characters (like `┌` or `┐`) unless explicitly requested, as they may break in certain terminal environments or legacy editors.
2. **Maintain Monospace**: Every line must be checked for character-width consistency.
3. **Avoid Complexity**: If a diagram requires more than **80 characters** of width, break it into *"Sub-Modules"* to ensure it fits standard IDE windows.
