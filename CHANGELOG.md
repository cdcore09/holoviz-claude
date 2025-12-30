# Changelog

All notable changes to the HoloViz Expert plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased] - 2025-12-30

### Changed

#### Plugin Architecture (BREAKING - Multi-Plugin Support)
- **Restructured into self-contained plugin** at `plugins/holoviz-expert/`
- Relocated all agents to `plugins/holoviz-expert/agents/`
- Relocated all skills to `plugins/holoviz-expert/skills/`
- Relocated all resources to `plugins/holoviz-expert/resources/`
- Moved `.mcp.json` into `plugins/holoviz-expert/` directory
- Updated `marketplace.json` source path to `./plugins/holoviz-expert`
- Updated all agent and skill paths in marketplace.json
- **Benefits**: Enables future multi-plugin development with clean separation

#### Agent Optimization (Token Efficiency)
- **Removed Technical Foundation sections** from all 4 agents
- Eliminated ~950 words of duplicative content between agents and skills
- Achieved **9.9% overall reduction** (63 lines removed, 639 → 576 lines)
- **Agent-specific reductions**:
  - `panel-specialist.md`: 133 → 115 lines (13.5% reduction)
  - `geo-spatial-expert.md`: 192 → 174 lines (9.4% reduction)
  - `data-engineer.md`: 171 → 154 lines (9.9% reduction)
  - `visualization-designer.md`: 143 → 133 lines (7.0% reduction)
- **Zero loss** of unique orchestration logic or workflow guidance
- **Preserved**: Domain context, workflow frameworks, agent personality, example interactions

#### Architecture Principles
- **Clear separation**: Agents orchestrate workflows, skills provide technical knowledge
- **Single source of truth**: Skills contain authoritative documentation
- **Improved maintainability**: Skill updates don't require agent updates
- **Better token efficiency**: No duplication between agents and skills

#### Repository Metadata
- Updated package name: `holoviz-agents` → `holoviz-claude`
- Updated owner: `HoloViz Community` → `Cordero Core`
- Updated repository URLs to `cdcore09/holoviz-claude`
- Updated homepage to match new repository location

### Technical Details

#### Content Metrics
- Agent total: 576 lines (down from 639)
- Skills total: ~19,000 lines (unchanged - remain comprehensive)
- Resources total: ~3,000 lines (unchanged)
- Elimination of 40-50% duplication between agents and skills

## [1.1.0] - 2025-12-20

### Added

#### Plugin Structure (BREAKING)
- Restructured to follow Claude Code marketplace pattern
- **marketplace.json** moved to `.claude-plugin/marketplace.json`
- Removed `plugin.json` (consolidated into marketplace.json)
- Uses multi-plugin repository structure compatible with rse-agents pattern
- Plugin paths now use relative references (`./agents/`, `./skills/`)

### Added

#### Skills (2 New Lumen Skills)
- **Lumen Dashboards**: Declarative, no-code dashboard development
  - YAML-based specifications for rapid development
  - Data sources (files, databases, REST APIs)
  - Transforms and filters for data processing
  - Views (tables, plots, indicators, custom components)
  - Pipelines combining sources, transforms, and views
  - Layout and responsive design patterns
  - Global configuration and theming
  - Complete dashboard examples
  - Python API for programmatic creation
  - Best practices and deployment guidance

- **Lumen AI**: AI-powered natural language data exploration
  - Natural language interface for querying data
  - Multi-LLM support (OpenAI, Anthropic, Google, Mistral, local models)
  - Agent architecture (SQL, hvPlot, VegaLite, Analysis, Chat agents)
  - Custom agent development patterns
  - Custom analyses for domain-specific tasks
  - Custom tools for extending capabilities
  - Document context and RAG integration
  - Memory and context management
  - Complete examples with business analytics use cases
  - LLM provider configuration guide
  - Security and privacy best practices

#### Resources (Enhanced)
- **best-practices.md**: Now includes Lumen-specific guidance (557 lines)
  - No-code dashboard development patterns
  - AI query optimization
  - LLM selection strategies

- **code-patterns.md**: Added Lumen patterns (936 lines)
  - Multi-source dashboard examples
  - Drill-down dashboard patterns
  - Custom component integration

- **troubleshooting.md**: Lumen troubleshooting section (918 lines)
  - Dashboard specification validation
  - LLM integration issues
  - Agent selection debugging

### Changed

- Updated plugin version from 1.0.0 to 1.1.0
- Enhanced README with Lumen sections
- Updated dependencies to include lumen >= 0.10.0
- Expanded skill count from 7 to 9
- Updated marketplace listing with new features

### Technical Details

#### New Dependencies
- lumen >= 0.10.0
- Optional: openai, anthropic, or other LLM providers for AI features

#### Content Metrics
- Added 5,195 lines of Lumen documentation
- Lumen Dashboards skill: 1,500+ lines
- Lumen AI skill: 1,280+ lines
- Total plugin content: ~22,000 lines

## [1.0.0] - 2025-12-20

### Added

#### Plugin Foundation
- Initial release of HoloViz Expert Claude Code plugin
- Comprehensive plugin.json with full metadata
- Marketplace.json for distribution
- MCP server configuration for real-time library access
- BSD 3-Clause license

#### Agents (4 Specialized Experts)
- **Panel Specialist**: Interactive dashboard and web application development
  - Component-based architecture guidance
  - Reactive programming patterns
  - Template and theming strategies
  - Real-time data streaming
  - File handling and validation workflows

- **Visualization Designer**: Multi-library visualization strategy
  - Library selection guidance
  - Visualization design principles
  - Composition and layout strategies
  - Color and accessibility recommendations
  - Performance optimization for visualization

- **Data Engineer**: Large-scale data and performance optimization
  - Datashader implementation guidance
  - Memory optimization techniques
  - Chunked processing for massive files
  - Performance profiling and benchmarking
  - Real-time streaming visualization

- **Geo-Spatial Expert**: Geographic and mapping visualizations
  - GeoViews and GeoPandas expertise
  - Coordinate reference system management
  - Multi-layer map composition
  - Spatial analysis workflows
  - Tile provider integration

#### Skills (7 Comprehensive Knowledge Bases)
- **Panel Dashboards**: Complete guide to Panel and Param
  - Component systems and layouts
  - Reactive patterns and watchers
  - Template systems (Material, Bootstrap, Vanilla, Dark)
  - File upload and data processing
  - Real-time updates and streaming
  - Best practices and common patterns

- **Plotting Fundamentals**: hvPlot and HoloViews basics
  - Quick plotting with hvPlot
  - Common plot types and customization
  - Interactive features and hover information
  - Geographic plotting
  - Performance considerations

- **Data Visualization**: Advanced HoloViews techniques
  - Element composition (overlays, layouts, facets)
  - Interactive streams and selection
  - Dynamic maps for responsive visualization
  - Network and hierarchical data
  - Statistical visualizations
  - Advanced styling and theming

- **Geospatial Visualization**: Professional mapping with GeoViews
  - Basic geographic visualization
  - Point, polygon, and line features
  - Choropleth maps
  - Spatial analysis and joins
  - Multi-layer composition
  - Tile provider integration
  - Optimization for large geographic datasets

- **Advanced Rendering**: Datashader for massive datasets
  - Datashader fundamentals and canvas configuration
  - Aggregation strategies (count, mean, sum, max/min)
  - Memory optimization techniques
  - Transfer functions and color mapping
  - Chunked processing for files larger than RAM
  - Integration with HoloViews and Panel

- **Parameterization**: Declarative parameter systems with Param
  - Parameter basics and type safety
  - Advanced parameter types (Date, Path, Range, Color, Dict)
  - Dynamic dependencies with @param.depends
  - Watchers for side effects
  - Custom validation
  - Hierarchical parameterization
  - Panel UI generation

- **Colormaps & Styling**: Color management and visual design
  - Colorcet colormap selection and best practices
  - Accessibility and colorblind-friendly design
  - Custom color mapping and normalization
  - HoloViews element styling
  - Panel theme customization
  - Dark mode support
  - Multi-element styling consistency

#### Resources (5 Curated Reference Materials)
- **holoviz-ecosystem.md**: Complete overview of the HoloViz ecosystem
  - Core philosophy and principles
  - Library stack overview
  - Relationships and data flow
  - When to use each library
  - Common workflows
  - Integration patterns
  - Learning path recommendations

- **library-matrix.md**: Library selection decision framework
  - Quick reference tables by task, data size, data type
  - Decision trees for common scenarios
  - Trade-offs and considerations
  - Library combinations and stacks
  - Performance comparisons
  - Example scenarios and recommendations

- **best-practices.md**: Proven patterns and techniques
  - Performance optimization
  - Accessibility guidelines
  - Code organization strategies
  - Responsive design patterns
  - Error handling approaches

- **code-patterns.md**: Production-ready code snippets
  - Common patterns for each library
  - Reusable component templates
  - Integration examples
  - Performance optimization examples

- **troubleshooting.md**: Solutions to common issues
  - Common problems and solutions
  - Debugging strategies
  - Performance troubleshooting
  - Platform-specific issues

#### Documentation
- Comprehensive README.md with:
  - Feature overview
  - Quick start guides
  - Library overview with code examples
  - Expert guidance examples
  - Installation instructions
  - Architecture diagram
  - Skill deep dives
  - Best practices highlights
  - MCP server integration details
  - Resources and community links
  - Troubleshooting section

- License information (BSD 3-Clause)
- Changelog (this file)

### Technical Details

#### Dependencies
- panel >= 1.3.0
- holoviews >= 1.18.0
- hvplot >= 0.9.0
- geoviews >= 1.11.0
- datashader >= 0.15.0
- param >= 2.0.0
- colorcet >= 3.1.0

#### Supported Features
- Agent-based guidance with specialized expertise
- Skill-based knowledge organization
- Resource library for reference
- MCP server integration for real-time information
- Comprehensive code examples and patterns
- Best practices and optimization guidance
- Accessibility and performance considerations

### Quality Assurance

- Comprehensive agent profiles with clear expertise areas
- Detailed skill documentation with real-world examples
- Production-ready code patterns and templates
- Accessibility guidelines integrated throughout
- Performance optimization recommendations
- Troubleshooting guides for common issues
- Integration with broader HoloViz ecosystem

## Future Versions

### Planned for 1.2.0
- Advanced styling patterns and theme templates
- Performance benchmarking tools
- Interactive examples and tutorials
- Deployment and scaling guidance
- Integration with popular web frameworks
- Advanced geospatial analysis patterns
- Real-time data streaming best practices

### Long Term
- Community-contributed patterns library
- Video tutorials and walkthroughs
- Interactive skill assessments
- Certification pathway
- Integration with development tools

## Support

For issues, questions, or contributions, visit:
- [HoloViz Discourse](https://discourse.holoviz.org)
- [GitHub Repository](https://github.com/holoviz/)
- [Official Website](https://holoviz.org)

---

## Version Numbering

This plugin uses [Semantic Versioning](https://semver.org/):
- MAJOR version for incompatible API changes
- MINOR version for new functionality in a backward compatible manner
- PATCH version for backward compatible bug fixes

## Release Schedule

- **Patch releases** (1.0.x): As needed for bug fixes
- **Minor releases** (1.x.0): Quarterly with new features
- **Major releases** (x.0.0): As needed for significant changes
