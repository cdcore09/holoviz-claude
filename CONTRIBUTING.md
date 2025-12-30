# Contributing to HoloViz Expert Plugin

Thank you for your interest in contributing to the HoloViz Expert Claude Code plugin! This document provides guidelines and instructions for contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Workflow](#development-workflow)
- [Content Guidelines](#content-guidelines)
- [Testing Your Changes](#testing-your-changes)
- [Submitting Changes](#submitting-changes)
- [Community](#community)

## Code of Conduct

This project adheres to a Code of Conduct that all contributors are expected to follow. Please read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) before contributing.

## Getting Started

### Prerequisites

- Git
- Claude Code CLI installed
- HoloViz libraries (Panel, HoloViews, hvPlot, GeoViews, Datashader, Lumen, Param, Colorcet)

```bash
pip install panel holoviews hvplot geoviews datashader lumen param colorcet
```

### Fork and Clone

1. Fork the repository on GitHub
2. Clone your fork locally:

```bash
git clone https://github.com/YOUR-USERNAME/holoviz-plugins.git
cd holoviz-plugins
```

3. Add the upstream repository:

```bash
git remote add upstream https://github.com/cdcore09/holoviz-plugins.git
```

## How to Contribute

We welcome contributions in several areas:

### 1. Skills Content

- Add new code examples
- Improve existing documentation
- Add troubleshooting tips
- Update best practices

### 2. Agent Improvements

- Enhance agent prompts
- Improve agent descriptions
- Add new use cases

### 3. Resources

- Add new best practices
- Contribute code patterns
- Expand troubleshooting guides
- Update library compatibility matrix

### 4. Bug Fixes

- Fix typos and formatting issues
- Correct code examples
- Update outdated information

### 5. New Features

- Propose new skills
- Suggest new agents
- Add integration examples

## Development Workflow

### 1. Create a Branch

Create a descriptive branch name:

```bash
git checkout -b feat/add-new-skill
git checkout -b fix/typo-in-panel-skill
git checkout -b docs/improve-contributing-guide
```

Branch naming convention:
- `feat/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation updates
- `refactor/` - Code refactoring

### 2. Make Your Changes

Follow these guidelines:

#### For Skills (SKILL.md files)

- Maintain the existing structure
- Add working code examples
- Include output/results where applicable
- Follow HoloViz best practices
- Test all code examples

#### For Agents (agent markdown files)

- Keep frontmatter properly formatted
- Ensure clear, actionable descriptions
- Maintain consistent tone
- Reference relevant skills

#### For Resources

- Use clear headings and structure
- Provide practical examples
- Link to official documentation
- Keep content up-to-date

### 3. Code Example Standards

All code examples should:

```python
# ✅ Good: Complete, runnable example
import panel as pn
import holoviews as hv

# Create data
data = [(i, i**2) for i in range(10)]

# Create plot
curve = hv.Curve(data, 'x', 'y')

# Display
curve.opts(width=600, height=400, title='Quadratic Function')
```

```python
# ❌ Bad: Incomplete or unclear example
curve = hv.Curve(data)  # What is data? Where does it come from?
curve.opts(...)  # What options?
```

### 4. Documentation Style

- Use clear, concise language
- Write in active voice
- Provide context for examples
- Include "when to use" guidance
- Add troubleshooting tips

## Content Guidelines

### Skills

Each skill should include:

1. **Overview**: What the skill covers
2. **Fundamentals**: Core concepts
3. **Common Patterns**: Frequently used approaches
4. **Advanced Techniques**: Expert-level content
5. **Best Practices**: Recommendations
6. **Troubleshooting**: Common issues and solutions
7. **Code Examples**: Working, tested examples

### Agents

Each agent should have:

1. **YAML Frontmatter**: Properly formatted with all fields
2. **Profile**: Clear description of expertise
3. **Expertise Areas**: Specific competencies
4. **When to Use**: Example scenarios
5. **Integration**: How it works with other agents

### Code Quality

- All code must be tested with current HoloViz versions
- Follow PEP 8 style guidelines
- Use meaningful variable names
- Add comments for complex logic
- Include error handling where appropriate

## Testing Your Changes

### 1. Validate Markdown

Ensure markdown files are properly formatted:

```bash
# Check for broken links (if you have a markdown linter)
mdl *.md
```

### 2. Test Code Examples

Run all code examples to ensure they work:

```python
# Test in a fresh Python environment
python -m venv test_env
source test_env/bin/activate  # or `test_env\Scripts\activate` on Windows
pip install panel holoviews hvplot geoviews datashader lumen param colorcet

# Run your code examples
python test_example.py
```

### 3. Test Agent Invocation

If you modified agents:

```bash
# Test agent can be invoked (if plugin is installed locally)
# Ask a question that should trigger the agent
```

### 4. Validate YAML

Check YAML frontmatter syntax:

```bash
# You can use a YAML linter or Python
python -c "import yaml; yaml.safe_load(open('agents/your-agent.md').read().split('---')[1])"
```

## Submitting Changes

### 1. Commit Your Changes

Write clear, descriptive commit messages:

```bash
git add .
git commit -m "feat(skill): Add new datashader aggregation examples

- Add mean aggregation example
- Add sum aggregation example
- Include performance comparison"
```

Commit message format:
```
<type>(<scope>): <subject>

<body>
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

### 2. Push to Your Fork

```bash
git push origin feat/your-branch-name
```

### 3. Create a Pull Request

1. Go to the repository on GitHub
2. Click "New Pull Request"
3. Select your fork and branch
4. Fill out the PR template with:
   - Clear description of changes
   - Motivation and context
   - Testing performed
   - Related issues (if applicable)

### 4. Respond to Review

- Address reviewer feedback promptly
- Make requested changes in new commits
- Ask questions if feedback is unclear
- Be respectful and professional

## Content Review Criteria

PRs will be reviewed for:

- **Accuracy**: Is the content technically correct?
- **Clarity**: Is it easy to understand?
- **Completeness**: Are examples complete and runnable?
- **Style**: Does it match existing content style?
- **Value**: Does it add value to users?

## Recognition

All contributors will be recognized in:

- GitHub contributors sidebar
- CHANGELOG.md for significant contributions
- Repository acknowledgments

## Getting Help

If you need help:

- Open an issue with the `question` label
- Join the [HoloViz Discourse](https://discourse.holoviz.org)
- Check existing issues and discussions

## License

By contributing, you agree that your contributions will be licensed under the BSD 3-Clause License.

## Questions?

Don't hesitate to ask! We're here to help. Open an issue or reach out through:

- GitHub Issues
- HoloViz Discourse
- Project discussions

Thank you for contributing to the HoloViz Expert plugin!
