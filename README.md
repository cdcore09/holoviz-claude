# HoloViz Expert - Claude Code Plugin

**Expert-level guidance for interactive data visualization and dashboards with HoloViz**

The definitive Claude Code plugin for mastering the HoloViz ecosystem. Get strategic guidance, production-ready code patterns, and deep expertise in Panel, HoloViews, hvPlot, GeoViews, Datashader, Param, and Colorcet.

## Features

### 4 Specialized Agents
- **Panel Specialist**: Interactive dashboards and web applications
- **Visualization Designer**: Multi-library visualization strategy
- **Data Engineer**: Large-scale data and performance optimization
- **Geo-Spatial Expert**: Geographic and mapping visualizations

### 9 Comprehensive Skills
- **Panel Dashboards**: Interactive apps with Panel and Param
- **Plotting Fundamentals**: Quick visualization with hvPlot
- **Data Visualization**: Advanced HoloViews composition
- **Geospatial Visualization**: Maps and geographic data
- **Advanced Rendering**: Datashader for massive datasets
- **Parameterization**: Declarative parameter systems
- **Colormaps & Styling**: Color management and visual design
- **Lumen Dashboards**: No-code dashboards with YAML specifications
- **Lumen AI**: AI-powered natural language data exploration

### Rich Resource Library
- **HoloViz Ecosystem Overview**: Understanding all the libraries
- **Library Selection Matrix**: Choosing the right tools
- **Best Practices Guide**: Proven patterns and techniques
- **Code Patterns**: Production-ready snippets
- **Troubleshooting Guide**: Solutions to common issues

## Quick Start

### For Exploratory Visualization
Ask the **Visualization Designer**:
> "What's the best way to visualize this dataset?"

### For Building Dashboards
Ask the **Panel Specialist**:
> "Build an interactive dashboard for monitoring real-time metrics"

### For Large Datasets
Ask the **Data Engineer**:
> "How do I visualize 100 million data points efficiently?"

### For Geographic Data
Ask the **Geo-Spatial Expert**:
> "Create an interactive map of my geospatial data"

## Use Cases

### Interactive Dashboards
- Real-time monitoring applications
- Business intelligence dashboards
- Data exploration tools
- Scientific analysis interfaces

### Data Visualization
- Publication-quality figures
- Multi-dimensional data exploration
- Comparative analysis visualizations
- Report generation

### Large-Scale Data
- 100M+ point cloud visualization
- Geospatial analysis of massive datasets
- Time-series data exploration
- High-frequency trading analytics

### Geographic Applications
- Maps and spatial analysis
- Weather data visualization
- Real estate and market analysis
- Infrastructure planning tools

## Library Guide

### Param
Declarative, type-safe parameter system with automatic validation
```python
class Config(param.Parameterized):
    count = param.Integer(default=10, bounds=(1, 100))
    name = param.String(default='Data')
```

### HoloViews
Declarative data visualization with advanced composition
```python
scatter = hv.Scatter(data, 'x', 'y')
curve = hv.Curve(data, 'x', 'y')
overlay = scatter * curve
```

### hvPlot
Pandas-like plotting interface for quick visualization
```python
df.hvplot.scatter(x='x', y='y', by='category')
```

### GeoViews
Geographic data visualization with tile providers
```python
gv.Polygons(geodataframe).opts(cmap='viridis')
```

### Datashader
Efficient rendering of 100M+ point datasets
```python
from holoviews.operation.datashader import datashade
datashade(scatter, cmap='viridis')
```

### Panel
Interactive web applications in pure Python
```python
pn.Column(
    pn.pane.Markdown('# Dashboard'),
    plot,
    controls
).servable()
```

### Colorcet
Perceptually uniform colormaps for scientific visualization
```python
from colorcet import cm
plot.opts(cmap=cm['cet_fire'])
```

### Lumen
No-code dashboards with YAML specs or AI-powered data exploration
```python
# Lumen Dashboards: YAML configuration
lumen serve dashboard.yaml

# Lumen AI: Natural language queries
lumen-ai serve data.csv
# Ask: "Show me total sales by region"
```

## Expert Guidance Examples

### Example 1: Performance Optimization
**User**: "My dashboard with 10M points is too slow"

**Data Engineer recommends**:
1. Use Datashader for rasterization
2. Aggregate data by region
3. Implement progressive disclosure with zooming
4. Profile with memory_profiler to find bottlenecks
5. Caching strategy with reduced update frequency

### Example 2: Visualization Selection
**User**: "50M GPS points, value gradient, need to find patterns"

**Visualization Designer suggests**:
1. Datashader for density heatmap
2. Perceptually uniform colormap (Colorcet)
3. Multi-resolution exploration (zoom-based)
4. Panel application for interactive exploration
5. Alternative: Hexbin aggregation

### Example 3: Application Architecture
**User**: "Build a multi-page app for data analysis"

**Panel Specialist designs**:
1. Param class for application state
2. Panel tabs for different views
3. Reactive dependencies for auto-updates
4. Template for consistent styling
5. File upload for data ingestion

### Example 4: Geographic Application
**User**: "Create a map showing store locations colored by revenue"

**Geo-Spatial Expert implements**:
1. GeoDataFrame from lat/lon coordinates
2. GeoViews Points layer with color encoding
3. Tile provider background (OpenStreetMap)
4. Interactive hover with store details
5. Panel integration for controls

## Installation

This plugin requires HoloViz libraries:

```bash
pip install panel holoviews hvplot geoviews datashader lumen param colorcet
```

Optional: For Lumen AI features
```bash
pip install lumen[ai]
# Plus LLM provider (choose one):
pip install openai        # OpenAI
pip install anthropic     # Anthropic Claude
```

Optional: For MCP server integration
```bash
pip install holoviz-mcp
```

## Architecture

```
holoviz-expert/
├── .claude-plugin/
│   └── plugin.json                      # Plugin metadata
├── agents/                              # 4 specialized agents
│   ├── panel-specialist.md
│   ├── visualization-designer.md
│   ├── data-engineer.md
│   └── geo-spatial-expert.md
├── skills/                              # 9 comprehensive skills
│   ├── panel-dashboards/
│   ├── plotting-fundamentals/
│   ├── data-visualization/
│   ├── geospatial-visualization/
│   ├── advanced-rendering/
│   ├── lumen-dashboards/
│   ├── lumen-ai/
│   ├── parameterization/
│   └── colormaps-styling/
├── resources/                           # Curated reference materials
│   ├── holoviz-ecosystem.md
│   ├── library-matrix.md
│   ├── best-practices.md
│   ├── code-patterns.md
│   └── troubleshooting.md
├── .mcp.json                           # MCP server configuration
├── marketplace.json                    # Marketplace listing
├── README.md                           # This file
├── LICENSE                             # BSD-3-Clause
└── CHANGELOG.md                        # Version history

```

## Skill Deep Dives

### Panel Dashboards Skill
Complete guide to building interactive applications:
- Component-based architecture
- Reactive programming patterns
- Template systems and theming
- Real-time data streaming
- File handling and validation

### Plotting Fundamentals Skill
Quick visualization with hvPlot and HoloViews basics:
- Common plot types
- Customization options
- Interactive features
- Geographic plotting
- Performance considerations

### Data Visualization Skill
Advanced HoloViews composition and interactivity:
- Element composition (overlays, layouts, facets)
- Interactive streams and selection
- Dynamic maps for responsive visualization
- Network and hierarchical data
- Statistical visualizations

### Geospatial Visualization Skill
Professional mapping with GeoViews:
- Basic geographic visualization
- Point data on maps
- Choropleth maps
- Spatial analysis workflows
- Multi-layer compositions
- Optimization for large geographic datasets

### Advanced Rendering Skill
Efficient handling of massive datasets:
- Datashader fundamentals
- Aggregation strategies
- Memory optimization
- Chunked processing
- Performance profiling
- Integration with Panel and HoloViews

### Parameterization Skill
Declarative parameter systems with Param:
- Parameter basics and validation
- Dynamic dependencies
- Watchers for side effects
- Hierarchical parameterization
- Integration with Panel UI generation

### Colormaps & Styling Skill
Professional color and visual design:
- Colorcet colormap selection
- Accessibility and colorblindness
- Custom color mapping
- HoloViews and Panel styling
- Theme customization
- Dark mode support

## Best Practices Highlights

### Performance
- Use hvPlot for < 100k points
- Use Datashader for 100M+ points
- Implement aggregation and sampling
- Cache expensive computations
- Profile with profilers before optimizing

### Accessibility
- Use perceptually uniform colormaps
- Provide multiple visual encodings (color, size, shape)
- Test with colorblind vision simulators
- Include clear labels and legends
- Support keyboard navigation

### Code Organization
- Separate UI concerns from business logic
- Use Param classes for configuration
- Create reusable component functions
- Organize related plots into modules
- Document with clear docstrings

### Responsive Design
- Always use `responsive=True`
- Test on multiple screen sizes
- Use appropriate layout strategies
- Implement lazy loading for large content
- Monitor performance on slower devices

## Integration with MCP Server

The plugin includes MCP server configuration for real-time library access:

```json
{
  "servers": {
    "holoviz-mcp": {
      "command": "uvx",
      "args": ["holoviz-mcp"]
    }
  }
}
```

This enables:
- Real-time library documentation
- Latest API reference access
- Example gallery integration
- Version information lookup

## Resources

### Official Documentation
- [HoloViz Homepage](https://holoviz.org)
- [Panel Documentation](https://panel.holoviz.org)
- [HoloViews Documentation](https://holoviews.org)
- [hvPlot Documentation](https://hvplot.holoviz.org)
- [GeoViews Documentation](https://geoviews.org)
- [Datashader Documentation](https://datashader.org)
- [Param Documentation](https://param.holoviz.org)
- [Colorcet Documentation](https://colorcet.holoviz.org)

### Community
- [HoloViz Discourse](https://discourse.holoviz.org)
- [GitHub Discussions](https://github.com/holoviz/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/holoviz)

### Learning Resources
- [HoloViz Gallery](https://holoviz.org/gallery/index.html)
- [Panel Examples](https://panel.holoviz.org/gallery/index.html)
- [Datashader Examples](https://datashader.org/getting_started/index.html)
- [Tutorial Notebooks](https://holoviz.org/tutorial/index.html)

## Troubleshooting

### Common Issues

**Q: My plot won't display**
- Ensure you've imported the plotting library (hvplot.pandas, geoviews, etc.)
- Check that your data isn't empty
- Verify column names match exactly

**Q: Dashboard is running too slow**
- Profile to find the bottleneck
- Use Datashader for > 100k points
- Implement aggregation or sampling
- Enable caching for expensive computations

**Q: Visualization looks unclear**
- Use perceptually uniform colormaps (Colorcet)
- Add legends and labels
- Increase figure size
- Consider faceting for categorical data

**Q: Map isn't displaying**
- Verify coordinate reference system (CRS)
- Check geometry validity with `gdf.is_valid.all()`
- Ensure coordinates are in correct order (lon, lat for WGS84)

See the **Troubleshooting Guide** in resources for detailed solutions.

## Contributing

This plugin is part of the HoloViz ecosystem. To contribute:

1. Visit [HoloViz on GitHub](https://github.com/holoviz)
2. Check existing issues and discussions
3. Submit improvements and updates
4. Follow HoloViz community guidelines

## License

BSD 3-Clause License - See LICENSE file for details

## Citation

If you use this plugin in your research, please cite HoloViz:

```bibtex
@software{holoviz2024,
  author = {HoloViz Contributors},
  title = {HoloViz: Flexible Scientific Visualization in Python},
  url = {https://holoviz.org},
  year = {2024}
}
```

## Support

- **Questions**: Ask in [HoloViz Discourse](https://discourse.holoviz.org)
- **Issues**: Report on respective GitHub repositories
- **Plugin Issues**: Report in plugin repository
- **Professional Support**: Visit [holoviz.org](https://holoviz.org)

## Changelog

### Version 1.0.0 (2025-12-20)
- Initial release with full HoloViz ecosystem support
- 4 specialized agents
- 7 comprehensive skills
- Complete resource library
- MCP server integration

## About HoloViz

HoloViz (formerly PyViz) is a comprehensive Python ecosystem for building data visualization applications. Created and maintained by a dedicated community of data scientists and engineers, it powers visualization solutions across academia, government, and industry.

Learn more at [holoviz.org](https://holoviz.org)

---

**Ready to become a HoloViz expert?** Start by choosing an agent that matches your current task!
