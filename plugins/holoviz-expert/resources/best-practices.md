# HoloViz Best Practices Guide

## Overview

This guide compiles best practices from across the HoloViz ecosystem to help you build high-quality, performant, and maintainable data visualization applications.

---

## General Principles

### Separation of Content and Presentation

**HoloViews Philosophy**: Maintain strict separation between data/declarations and visual styling.

```python
# Good: Separate data declaration from styling
scatter = hv.Scatter(data, 'x', 'y')
scatter.opts(color='blue', size=10, tools=['hover'])

# Avoid: Mixing data logic with presentation
```

**Benefits**:
- Reusable visualizations
- Easier testing and debugging
- Clear code organization
- Flexible styling changes

### Library Selection Strategy

**Choose the right tool for the task**:

- **hvPlot**: Quick exploratory plots (<100k points)
- **HoloViews**: Complex compositions, multi-dimensional data
- **Datashader**: Large datasets (100M+ points)
- **Panel**: Interactive applications and dashboards
- **GeoViews**: Geographic and spatial data
- **Param**: Application state and configuration

**When to combine libraries**:
- Panel + HoloViews: Interactive dashboards with complex visualizations
- hvPlot + Datashader: Quick syntax with big data rendering
- GeoViews + Datashader: Large geographic datasets
- Panel + Param: Reactive applications with clean state management

---

## Performance Best Practices

### Data Storage and Format

**Use Apache Parquet for large datasets** ([Datashader Performance Guide](https://datashader.org/user_guide/Performance.html)):

```python
# Save with categorical optimization
df['category'] = df['category'].astype('category')
df.to_parquet('data.parquet', compression='snappy', engine='fastparquet')
```

**Benefits**:
- Memory-efficient categorical representation
- Fast columnar access
- Excellent compression ratios

### Numeric Precision Optimization

**Use 32-bit floats when possible**:

```python
# Convert to float32 for aggregations
df['value'] = df['value'].astype('float32')
```

**Impact**:
- 50% reduction in memory and storage
- Faster computations due to reduced memory access
- Minimal precision loss for visualization aggregations

### Dask Integration

**Multi-core processing on single machines**:

```python
import dask.dataframe as dd
import multiprocessing as mp

# Optimal partition count
npartitions = mp.cpu_count()
dask_df = dd.from_pandas(df, npartitions=npartitions)

# Persist when data fits in memory
dask_df = dask_df.persist()
```

**Best practices**:
- Use `cpu_count()` partitions for single machines
- Persist data in memory when possible to avoid redundant loading
- Let Dask handle out-of-core operations for datasets larger than RAM

### When to Use Datashader

**Size thresholds** ([Datashader FAQ](https://datashader.org/FAQ.html)):

- **< 1,000 points**: Use standard plotting (Bokeh, Matplotlib)
- **1,000 - 100,000 points**: hvPlot works well
- **100,000+**: Consider Datashader
- **10M+**: Definitely use Datashader

**Trade-offs**:
- **Gain**: Render billions of points interactively
- **Lose**: Individual point inspection and metadata hover
- **Solution**: Use linked selection or aggregated hover

---

## Panel Application Best Practices

### Application Architecture

**Use Param for state management** ([Panel Cheat Sheet](https://panel.holoviz.org/api/cheatsheet.html)):

```python
import panel as pn
import param

class Dashboard(param.Parameterized):
    data_source = param.Selector(default='csv', objects=['csv', 'database'])
    refresh_rate = param.Integer(default=5000, bounds=(1000, 60000))

    @param.depends('data_source', 'refresh_rate')
    def view(self):
        # Automatically updates when parameters change
        return pn.Column(self._load_data(), self._create_plot())
```

**Why Param-based architecture**:
- Declarative parameter definitions
- Automatic validation
- Clean separation of logic and UI
- Reusable in CLI, batch, and server contexts

### Reactive Programming Patterns

**Use `@param.depends` correctly** ([Panel Dependencies Guide](https://panel.holoviz.org/how_to/param/dependencies.html)):

```python
class Analyzer(param.Parameterized):
    num_points = param.Integer(default=100)
    aggregation = param.Selector(default='mean', objects=['mean', 'sum'])

    # Without watch=True: Panel tracks and updates display
    @param.depends('num_points', 'aggregation')
    def summary(self):
        return f"Computing {self.aggregation} over {self.num_points} points"

    # With watch=True: For parameter-to-parameter dependencies
    @param.depends('num_points', watch=True)
    def _validate_range(self):
        if self.num_points > 1000:
            # Update another parameter
            self.aggregation = 'mean'
```

**Critical rule**: Don't use `watch=True` on methods already monitored by Panel (causes double execution).

### Server Deployment Patterns

**Wrap apps in functions for session isolation**:

```python
# Bad: Shared state across users
dashboard = Dashboard()
template.main.append(dashboard.view)

# Good: Separate state per session
def create_app():
    dashboard = Dashboard()
    template = pn.template.MaterialTemplate(title='Dashboard')
    template.main.append(dashboard.view)
    return template

pn.serve(create_app)
```

### Performance Profiling

**Use Panel's built-in profiling** ([Panel Profiling Guide](https://panel.holoviz.org/how_to/profiling/profile.html)):

```bash
# Profile initialization
panel serve app.py --profiler pyinstrument

# Profile specific callbacks
@pn.io.profile('expensive_computation', engine='snakeviz')
def process_data(event):
    # Complex computation
    pass
```

**Profiler selection**:
- **pyinstrument**: Quick visual execution time analysis
- **snakeviz**: Detailed call graph exploration
- **memray**: Memory allocation tracking

---

## HoloViews Customization Patterns

### Options System Hierarchy

**Apply customizations at the right level**:

```python
# Element-level: Single plot
scatter.opts(color='blue', size=10)

# Type-level: All scatters in session
hv.opts.Scatter(color='blue', size=10)

# Global defaults
hv.opts.defaults(
    hv.opts.Scatter(color='blue'),
    hv.opts.Curve(line_width=2)
)
```

### Data Processing Pipelines

**Build declarative, lazy pipelines**:

```python
# Declare transformations without execution
points = hv.Points(data, ['x', 'y'])
histogram = hv.operation.histogram(points, dimension='y', num_bins=20)
smoothed = histogram.apply(lambda h: h.opts(alpha=0.5))

# Computation happens only when displayed
smoothed  # Now it renders
```

**Benefits**:
- Lazy evaluation for performance
- Composable transformations
- Automatic updates with dynamic data

---

## Param Best Practices

### Parameter Definition Patterns

**Choose the right parameter type** ([Param User Guide](https://param.holoviz.org/user_guide/Parameters.html)):

```python
class Config(param.Parameterized):
    # Immutable, shared default
    name = param.String(default='default')

    # Mutable, instance-specific
    tags = param.List(default=[], instantiate=True)

    # Computed at instantiation
    created_at = param.Parameter(default_factory=lambda: datetime.now())

    # Validated range
    count = param.Integer(default=10, bounds=(1, 100))
```

**Key principles**:
- Use `instantiate=True` for mutable defaults (lists, dicts)
- Use `default_factory` for dynamic values
- Leverage built-in validation with bounds, objects, etc.

### Inheritance Best Practices

**Use inheritance for default management**:

```python
class BaseConfig(param.Parameterized):
    debug = param.Boolean(default=False)

class DevConfig(BaseConfig):
    debug = param.Boolean(default=True)  # Override default

# Control across hierarchy
BaseConfig.debug = True  # Affects all instances
```

---

## GeoViews Geographic Best Practices

### Coordinate Reference Systems

**Always be explicit about CRS**:

```python
# Specify CRS for your data
gdf = gpd.read_file('data.shp')
gdf = gdf.to_crs('EPSG:4326')  # WGS84

# GeoViews respects the CRS
polygons = gv.Polygons(gdf, crs=ccrs.PlateCarree())
```

**Common issues**:
- Mixing CRS causes misalignment
- Web tiles use EPSG:3857 (Web Mercator)
- Most GPS data is EPSG:4326 (WGS84)

### Geographic Data Validation

**Validate geometry before plotting**:

```python
# Check for invalid geometries
assert gdf.is_valid.all(), "Invalid geometries found"

# Fix invalid geometries
gdf['geometry'] = gdf.buffer(0)
```

---

## Accessibility Best Practices

### Colormap Selection

**Use perceptually uniform colormaps** from Colorcet:

```python
from colorcet import cm

# Good: Perceptually uniform
plot.opts(cmap=cm['cet_fire'])
plot.opts(cmap=cm['cet_blues'])

# Avoid: Non-uniform colormaps
# plot.opts(cmap='jet')  # Bad for colorblind users
```

**Categories**:
- **Continuous**: `cet_fire`, `cet_blues`, `cet_greens`
- **Diverging**: `cet_coolwarm`, `cet_bkr`
- **Categorical**: `cet_glasbey_*` for distinct categories
- **Colorblind-safe**: `cet_cbd1`, `cet_cbd2`

### Multiple Visual Encodings

**Don't rely solely on color**:

```python
# Good: Multiple encodings
hv.Scatter(data, 'x', 'y').opts(
    color='category',
    size='value',
    marker='type'  # Shape also encodes information
)

# Add labels and legends
plot.opts(show_legend=True, legend_position='right')
```

---

## Code Organization Best Practices

### Module Structure

**Separate concerns clearly**:

```
project/
├── data/
│   ├── loaders.py       # Data loading functions
│   └── transforms.py    # Data transformations
├── viz/
│   ├── plots.py         # Plot generation
│   ├── layouts.py       # Layout composition
│   └── styles.py        # Styling options
├── app/
│   ├── config.py        # Param classes
│   ├── callbacks.py     # Event handlers
│   └── dashboard.py     # Panel application
└── app.py               # Entry point
```

### Reusability Patterns

**Create composable functions**:

```python
def create_scatter(data, x, y, **opts):
    """Reusable scatter plot factory."""
    default_opts = {'size': 10, 'tools': ['hover']}
    default_opts.update(opts)
    return hv.Scatter(data, x, y).opts(**default_opts)

def create_dashboard_layout(*plots):
    """Standard dashboard layout."""
    return pn.Column(
        pn.pane.Markdown('# Dashboard'),
        pn.Row(*plots),
        sizing_mode='stretch_width'
    )
```

---

## Testing Best Practices

### Unit Testing Visualizations

**Test data transformations, not pixels**:

```python
def test_data_aggregation():
    data = pd.DataFrame({'x': [1, 2, 3], 'y': [4, 5, 6]})
    result = aggregate_by_bin(data, bin_size=1)
    assert len(result) == 3
    assert 'mean_y' in result.columns

def test_parameter_validation():
    config = Config()
    with pytest.raises(ValueError):
        config.count = 1000  # Exceeds bounds
```

### Performance Testing

**Benchmark critical paths**:

```python
import pytest

@pytest.mark.benchmark
def test_datashader_rendering(benchmark):
    df = create_large_dataset(10_000_000)
    result = benchmark(lambda: datashade(hv.Points(df)))
    assert result is not None
```

---

## Documentation Best Practices

### Docstring Standards

**Document parameters and returns clearly**:

```python
def create_choropleth(gdf, value_col, cmap='viridis'):
    """
    Create a choropleth map from GeoDataFrame.

    Parameters
    ----------
    gdf : gpd.GeoDataFrame
        Geographic data with geometry column
    value_col : str
        Column name for color encoding
    cmap : str, optional
        Colormap name (default: 'viridis')

    Returns
    -------
    gv.Polygons
        Styled geographic polygons

    Examples
    --------
    >>> choropleth = create_choropleth(states_gdf, 'population')
    """
    return gv.Polygons(gdf, vdims=[value_col]).opts(
        color=value_col, cmap=cmap, colorbar=True
    )
```

---

## Deployment Best Practices

### Environment Management

**Use environment.yml for reproducibility**:

```yaml
name: holoviz-app
channels:
  - conda-forge
dependencies:
  - python=3.11
  - panel>=1.3
  - holoviews>=1.18
  - datashader>=0.15
  - bokeh>=3.3
  - numpy
  - pandas
```

### Production Considerations

**Optimize for production**:

```python
# Enable caching
pn.config.cache = True

# Set appropriate throttle
pn.config.throttled = True

# Use CDN for faster loading
pn.extension(inline=False)

# Enable compression
panel serve app.py --num-threads 4 --websocket-max-message-size=100000000
```

---

## Summary Checklist

### Before Production

- [ ] Profile application performance
- [ ] Use appropriate library for data size
- [ ] Implement session isolation for multi-user apps
- [ ] Use perceptually uniform colormaps
- [ ] Validate geographic data CRS
- [ ] Add multiple visual encodings (not just color)
- [ ] Test on target deployment environment
- [ ] Document parameter meanings and bounds
- [ ] Use Parquet for large datasets
- [ ] Enable caching where appropriate

### Code Quality

- [ ] Separate content from presentation
- [ ] Use Param for configuration
- [ ] Create reusable components
- [ ] Write unit tests for transformations
- [ ] Document public APIs
- [ ] Use type hints where beneficial
- [ ] Handle errors gracefully
- [ ] Log important events

---

## References

- [Datashader Performance Guide](https://datashader.org/user_guide/Performance.html)
- [Panel How-To Guides](https://panel.holoviz.org/how_to/)
- [Panel Cheat Sheet](https://panel.holoviz.org/api/cheatsheet.html)
- [HoloViews User Guide](https://holoviews.org/user_guide/)
- [Param Documentation](https://param.holoviz.org/user_guide/Parameters.html)
- [GeoViews User Guide](https://geoviews.org/user_guide/)
- [hvPlot Tutorials](https://hvplot.holoviz.org/en/docs/latest/tutorials/index.html)
