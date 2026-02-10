---
name: assessment-diagram
description: Generate architecture diagram for the application
---

# Architecture Diagram Generation Skill

This skill generates an architecture diagram for the application based on its structure and dependencies.

## Purpose

This skill creates a visual representation of the application architecture, showing:
- Application components and their relationships
- External dependencies (databases, caches, APIs)
- Technology stack
- Communication patterns
- Deployment architecture

## Inputs

- `workspace-path` (Mandatory): The root path of the project to analyze
- `output-path` (Optional): The directory where the diagram will be saved. Defaults to `.github/modernize/`
- `format` (Optional): The output format (mermaid, plantuml, or both). Defaults to mermaid.

## Workflow

1. **Analyze Application Structure**:
   - Scan source code to identify main components
   - Detect application entry points
   - Identify controllers, services, repositories, etc.

2. **Identify Dependencies**:
   - Parse configuration files (application.properties, application.yml)
   - Detect external services (databases, caches, message queues)
   - Identify API integrations
   - Find configuration for external systems

3. **Map Architecture**:
   - Build a component model of the application
   - Map dependencies between components
   - Identify communication patterns
   - Determine deployment structure

4. **Generate Diagram**:
   - Create a Mermaid or PlantUML diagram
   - Include all identified components and dependencies
   - Show relationships and data flow
   - Add appropriate labels and annotations

5. **Save Output**:
   - Save the diagram source code to the output path
   - Optionally render the diagram to an image

## Output

The skill generates:
- `architecture-diagram.mmd` - Mermaid diagram source
- `architecture-diagram.md` - Markdown file with the diagram embedded
- Optionally: `architecture-diagram.puml` - PlantUML source

## Diagram Elements

The diagram includes:
- **Application Components**: Main application, controllers, services
- **Data Stores**: Databases, caches (Redis), file storage
- **External APIs**: Third-party API integrations
- **Infrastructure**: Web servers, containers
- **Connections**: HTTP, database connections, cache connections

## Completion Criteria

1. The architecture diagram is generated and saved
2. The diagram accurately represents the application structure
3. All major components and dependencies are included
4. The diagram is readable and well-formatted
5. The output is in the requested format(s)
