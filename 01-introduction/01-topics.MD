# Helm Course Syllabus

> Complete roadmap covering Helm CLI, Chart Development, Advanced Concepts, and Production Features.

---

# Part 1 - Helm CLI Commands

## Create & Manage Charts

- `helm create`
- `helm install`
- `helm upgrade`
- `helm rollback`
- `helm package`

---

## Repository & Search

- `helm list`
- `helm repo`
- `helm search`

---

## Chart Development Commands

- `helm lint`
- `helm template`
- `helm get`
- `helm history`
- `helm pull`
- `helm plugin`
- `helm show`
- `helm status`

---

## Validation & Utility Commands

- `helm test`
- `helm verify`
- `helm version`
- `helm env`
- `helm dependency`
- `helm push`
- `helm uninstall`
- `helm help`

---

## Installation & Basic Operations

1. Install Docker Desktop and Helm CLI
2. Install Helm
3. Helm Upgrade using `--set`
4. Helm Upgrade using Chart Versions
5. Helm Uninstall (Keep History)
6. Install with Generated Name
7. Install with `--atomic`
8. Working with Namespaces
9. Override Values

---

# Part 2 - Helm Chart Development

## Chart Structure

### 10. Helm Chart Structure

---

## Built-in Objects

### 11. Built-in Objects

Topics:

- `.Release`
- `.Chart`
- `.Files`
- `.Values`

---

## Helm Development Basics

### 12. Helm Development Basics

Topics:

- Template Actions
- Control White Spaces
- `default`
- `lower`
- `nindent`

---

## Flow Control

### 13. If / Else using `eq`

### 14. If / Else using `and`

### 15. If / Else using `or`

### 16. If / Else using `not`

### 17. `with`

### 18. `with` + `if/else`

### 19. Variables

### 20. `range` with List

### 21. `range` with Dictionary

---

## Template Functions

Functions Covered:

- `eq`
- `and`
- `or`
- `not`
- `toYaml`

---

## Named Templates

### 22. Named Templates (`_helpers.tpl`)

### 23. `printf` Function

### 24. Calling Templates from Templates

---

# Part 3 - Advanced Helm Concepts

## Create & Package Charts

### 25. Create and Package Charts

---

## Chart Dependencies

### 26. Dependencies

### 27. Dependency Alias

### 28. Dependency Condition

### 29. Dependency Condition Alias

### 30. Dependency Tags

---

## Subcharts

### 31. Override Subchart Values

### 32. Global Values

### 33. Import Values (Explicit)

### 34. Import Values (Implicit)

---

## Starters & Plugins

### 35. Starters

### 36. Plugins

### 37. Build Helm Plugins

---

## Hooks

### 38. Hooks

### 39. Hook Delete Policy

### 40. Hook Weights

### 41. Helm Tests

Topics Covered:

- Hook Types
- Delete Policies
- Hook Weights
- Testing Helm Charts

---

# Part 4 - Production Features

### 42. Resource Policy

### 43. Sign and Verify Charts

### 44. Host Helm Repository on GitHub Pages

### 45. Integrate GitHub Helm Repository with Artifact Hub

### 46. Validate `values.yaml` using JSON Schema

---

# Course Summary

## Helm CLI (24 Commands)

- `helm create`
- `helm install`
- `helm upgrade`
- `helm rollback`
- `helm package`
- `helm list`
- `helm repo`
- `helm search`
- `helm lint`
- `helm template`
- `helm get`
- `helm history`
- `helm pull`
- `helm plugin`
- `helm show`
- `helm status`
- `helm test`
- `helm verify`
- `helm version`
- `helm env`
- `helm dependency`
- `helm push`
- `helm uninstall`
- `helm help`

---

## Helm Development Topics

- Helm Chart Structure
- Built-in Objects
- Template Actions
- White Space Control
- Flow Control (`if`, `with`, `range`)
- Variables
- Template Functions
- Named Templates
- Template Reuse

---

## Advanced Helm Topics

- Packaging Charts
- Dependencies
- Dependency Alias
- Dependency Conditions
- Dependency Tags
- Subcharts
- Global Values
- Import Values
- Starters
- Plugins
- Hooks
- Testing

---

## Production Topics

- Resource Policies
- Chart Signing
- Chart Verification
- GitHub Pages Repository
- Artifact Hub Integration
- JSON Schema Validation

---

# Learning Outcome

After completing this course, you will be able to:

- Build production-ready Helm Charts
- Develop reusable templates
- Manage chart dependencies
- Create and publish Helm repositories
- Use hooks effectively
- Validate chart values using JSON Schema
- Sign and verify Helm Charts
- Deploy Kubernetes applications using Helm following industry best practices
