# HoloViz Troubleshooting Guide

## Overview

Common issues, error messages, and solutions for the HoloViz ecosystem. Organized by library and problem type.

---

## Panel Issues

### Display and Rendering

#### Issue: Plot Not Displaying

**Symptoms**: Empty output, no visualization shown

**Solutions**:

1. **Missing extension call**
```python
# Wrong
import panel as pn
app = pn.Column(...)

# Correct
import panel as pn
pn.extension('bokeh')  # or 'plotly', 'vega', etc.
app = pn.Column(...)
```

2. **Missing `.servable()` in deployment**
```python
# For panel serve
app = pn.Column(...)
app.servable()  # Required for deployment
```

3. **Jupyter notebook display issues**
```python
# Ensure extension is loaded in first cell
import panel as pn
pn.extension()

# For inline display
app = pn.Column(...)
app  # Display directly
```

#### Issue: Updates Not Reflecting

**Symptoms**: Changes to parameters don't update the UI

**Solutions**:

1. **Missing `@param.depends` decorator**
```python
# Wrong
def view(self):
    return pn.pane.Markdown(f"Value: {self.param_value}")

# Correct
@param.depends('param_value')
def view(self):
    return pn.pane.Markdown(f"Value: {self.param_value}")
```

2. **Incorrect use of `watch=True`**
```python
# Wrong: Double execution when method is monitored by Panel
@param.depends('value', watch=True)
def update_display(self):
    return pn.pane.Markdown(str(self.value))

# Correct: Only use watch for parameter-to-parameter dependencies
@param.depends('value')  # No watch=True
def update_display(self):
    return pn.pane.Markdown(str(self.value))
```

3. **Mutable default without instantiate**
```python
# Wrong: Shared state across instances
tags = param.List(default=[])

# Correct
tags = param.List(default=[], instantiate=True)
```

### Performance Issues

#### Issue: Dashboard Running Slowly

**Symptoms**: Laggy interface, slow updates

**Diagnostic approach**:

1. **Profile the application** ([Panel Profiling Guide](https://panel.holoviz.org/how_to/profiling/profile.html))
```bash
panel serve app.py --profiler pyinstrument
```

2. **Check for common bottlenecks**:
   - Large data transfers to browser
   - Expensive computations in callbacks
   - Missing caching
   - Too frequent updates

**Solutions**:

1. **Enable caching**
```python
import panel as pn

pn.config.cache = True

@pn.cache
def load_data():
    return pd.read_csv('large_file.csv')
```

2. **Throttle updates**
```python
# Reduce update frequency
slider = pn.widgets.FloatSlider(throttled=True)

# Or globally
pn.config.throttled = True
```

3. **Use background tasks for expensive operations**
```python
import asyncio

async def expensive_operation():
    await asyncio.sleep(2)  # Simulate work
    return result

@param.depends('trigger')
async def update(self):
    result = await expensive_operation()
    return pn.pane.Markdown(str(result))
```

4. **Reduce data sent to browser**
```python
# Wrong: Send millions of points to browser
plot = df.hvplot.scatter(x='x', y='y')  # df has 10M rows

# Correct: Aggregate or use Datashader
from holoviews.operation.datashader import datashade
plot = datashade(hv.Points(df, ['x', 'y']))
```

#### Issue: Memory Usage Growing Over Time

**Symptoms**: Application memory increases with each interaction

**Solutions**:

1. **Clear cached data periodically**
```python
def clear_old_cache():
    pn.state.cache.clear()
```

2. **Use session-specific state**
```python
# Wrong: Global state shared across users
data_cache = {}

# Correct: Session-specific state
def create_app():
    state = pn.state
    if 'data_cache' not in state:
        state.data_cache = {}
    # Use state.data_cache
```

3. **Profile memory usage**
```bash
panel serve app.py --profiler memray
```

### Deployment Issues

#### Issue: Works in Notebook, Fails in Deployment

**Symptoms**: Application works locally but fails with `panel serve`

**Solutions**:

1. **Wrap application in function for session isolation**
```python
# Wrong: Shared state
dashboard = Dashboard()
dashboard.view().servable()

# Correct: Per-session instances
def create_dashboard():
    dashboard = Dashboard()
    return dashboard.view()

create_dashboard().servable()
```

2. **Use relative paths**
```python
# Wrong: Absolute path
df = pd.read_csv('/Users/me/data.csv')

# Correct: Relative to script
import os
script_dir = os.path.dirname(__file__)
data_path = os.path.join(script_dir, 'data.csv')
df = pd.read_csv(data_path)
```

3. **Check environment variables**
```python
# Set appropriate defaults
DEBUG = os.getenv('DEBUG', 'False') == 'True'
```

#### Issue: CORS Errors in Deployment

**Symptoms**: Cross-origin errors in browser console

**Solutions**:

1. **Configure allowed origins**
```bash
panel serve app.py --allow-websocket-origin=myapp.com
```

2. **In code**
```python
pn.config.allow_websocket_origin = ['localhost:5006', 'myapp.com']
```

---

## HoloViews Issues

### Display Problems

#### Issue: Plot Displays but is Empty

**Symptoms**: Axes visible but no data shown

**Solutions**:

1. **Check data structure**
```python
# Verify data is not empty
print(len(df))
print(df.head())

# Check column names match exactly
hv.Scatter(df, 'x_col', 'y_col')  # Column names must be exact
```

2. **Verify dimension order**
```python
# HoloViews expects (kdims, vdims)
# Wrong
hv.Scatter(df, 'value', 'category')  # Probably backwards

# Correct
hv.Scatter(df, 'category', 'value')  # kdims, then vdims
```

3. **Check for NaN values**
```python
# Remove NaN values
df_clean = df.dropna(subset=['x', 'y'])
scatter = hv.Scatter(df_clean, 'x', 'y')
```

#### Issue: Options Not Applied

**Symptoms**: `.opts()` calls have no effect

**Solutions**:

1. **Use correct backend**
```python
import holoviews as hv
hv.extension('bokeh')  # Specify backend

# Some options are backend-specific
scatter.opts(size=10)  # Works with Bokeh
```

2. **Check option names**
```python
# Wrong
scatter.opts(colour='red')  # British spelling not accepted

# Correct
scatter.opts(color='red')  # American spelling
```

3. **Apply to correct element**
```python
# Overlay options vs element options
overlay = scatter * curve

# Wrong: Options on overlay
overlay.opts(color='red')  # Won't work as expected

# Correct: Options on individual elements
scatter.opts(color='blue')
curve.opts(color='red')
overlay = scatter * curve
```

### Composition Issues

#### Issue: Overlay Not Displaying Properly

**Symptoms**: Only one element shows, or elements don't align

**Solutions**:

1. **Ensure matching dimensions**
```python
# Elements must share key dimensions
scatter = hv.Scatter(df, 'x', 'y')
curve = hv.Curve(df2, 'x', 'y')  # Same kdims
overlay = scatter * curve
```

2. **Check coordinate ranges**
```python
# Elements might be outside visible range
scatter.opts(xlim=(-10, 10), ylim=(-10, 10))
curve.opts(xlim=(-10, 10), ylim=(-10, 10))
```

3. **Verify element types**
```python
# Can't overlay incompatible types
# Check what you're overlaying
print(type(element1), type(element2))
```

---

## Datashader Issues

### Performance Problems

#### Issue: Datashader Rendering is Slow

**Symptoms**: Long wait times for plot updates

**Solutions**:

1. **Optimize data format** ([Datashader Performance](https://datashader.org/user_guide/Performance.html))
```python
# Use Parquet with categorical optimization
df['category'] = df['category'].astype('category')
df.to_parquet('data.parquet', compression='snappy')

# Load efficiently
df = pd.read_parquet('data.parquet')
```

2. **Use Dask for large datasets**
```python
import dask.dataframe as dd

# Convert to Dask
dask_df = dd.from_pandas(df, npartitions=8)
dask_df = dask_df.persist()  # Keep in memory if possible

# Use with Datashader
points = hv.Points(dask_df, ['x', 'y'])
datashade(points)
```

3. **Reduce precision**
```python
# Use float32 instead of float64
df['x'] = df['x'].astype('float32')
df['y'] = df['y'].astype('float32')
```

#### Issue: Datashader Output Looks Wrong

**Symptoms**: Unexpected colors, missing categories, artifacts

**Solutions**:

1. **Check aggregation type**
```python
# For continuous values
datashade(points, aggregator='mean')

# For categories
datashade(points, aggregator='count_cat', color_key=['red', 'blue'])
```

2. **Verify color mapping**
```python
# Ensure colormap is appropriate
import colorcet as cc

# For continuous data
datashade(points, cmap=cc.fire)

# For categories, specify colors
datashade(points, color_key=['#FF0000', '#00FF00', '#0000FF'])
```

3. **Check data range**
```python
# Outliers can distort visualization
# Filter or transform data
df_filtered = df[(df.x > -3) & (df.x < 3)]
```

### Integration Issues

#### Issue: No Interactivity with Datashaded Plots

**Symptoms**: Hover, selection don't work

**Explanation**: This is expected behavior ([Datashader FAQ](https://datashader.org/FAQ.html))

**Solutions**:

1. **Use rasterize for aggregated hover**
```python
from holoviews.operation.datashader import rasterize

# Rasterize preserves values for hover
rasterized = rasterize(points, aggregator='mean')
rasterized.opts(tools=['hover'], colorbar=True)
```

2. **Implement linked selection**
```python
from holoviews import streams

# Create selection stream
selection = streams.Selection1D()
# Use with linked plots
```

3. **Downsample for interaction**
```python
# Show datashaded overview + downsampled detail
overview = datashade(all_points)
detail = hv.Scatter(sampled_points).opts(tools=['hover'])
layout = overview + detail
```

---

## GeoViews Issues

### Projection Problems

#### Issue: Map Not Displaying or Misaligned

**Symptoms**: Empty map, coordinates in wrong location

**Solutions**:

1. **Verify CRS is set correctly**
```python
import geopandas as gpd
import geoviews as gv

# Check current CRS
print(gdf.crs)

# Convert to WGS84 (EPSG:4326)
gdf = gdf.to_crs('EPSG:4326')

# Specify CRS in GeoViews
polygons = gv.Polygons(gdf, crs=ccrs.PlateCarree())
```

2. **Match tile CRS**
```python
# Tiles are typically Web Mercator (EPSG:3857)
from geoviews import tile_sources as gvts

# Your data in WGS84
polygons = gv.Polygons(gdf, crs=ccrs.PlateCarree())

# Overlay on tiles (GeoViews handles reprojection)
map_viz = gvts.OSM * polygons
```

3. **Check coordinate order**
```python
# GeoDataFrame: (lon, lat) for WGS84
# Check if coordinates are swapped
print(gdf.geometry.iloc[0])

# If swapped, fix with:
gdf['geometry'] = gdf.geometry.apply(lambda geom: shapely.ops.transform(lambda x, y: (y, x), geom))
```

#### Issue: Invalid Geometries Error

**Symptoms**: `ValueError` or rendering artifacts

**Solutions**:

1. **Check validity**
```python
# Find invalid geometries
invalid = gdf[~gdf.is_valid]
print(f"Invalid geometries: {len(invalid)}")

# Identify issues
for idx, row in invalid.iterrows():
    print(f"{idx}: {shapely.validation.explain_validity(row.geometry)}")
```

2. **Fix invalid geometries**
```python
# Simple fix: buffer by 0
gdf['geometry'] = gdf.geometry.buffer(0)

# More robust
gdf['geometry'] = gdf.geometry.make_valid()
```

3. **Simplify complex geometries**
```python
# Reduce complexity if needed
gdf['geometry'] = gdf.geometry.simplify(tolerance=0.01)
```

### Performance Issues with Geographic Data

#### Issue: Slow Rendering of Many Features

**Symptoms**: Lag when displaying many polygons/points

**Solutions**:

1. **Simplify geometries**
```python
# Reduce polygon complexity
gdf['geometry'] = gdf.geometry.simplify(tolerance=0.01, preserve_topology=True)
```

2. **Use Datashader for large point sets**
```python
from holoviews.operation.datashader import datashade

# Convert to Points
points = gv.Points(gdf, vdims=['value'])

# Datashade
shaded = datashade(points)
map_viz = gvts.OSM * shaded
```

3. **Filter features by zoom level**
```python
# Only show detailed features when zoomed in
# (requires dynamic map implementation)
```

---

## Param Issues

### Parameter Definition Problems

#### Issue: Parameter Value Not Updating

**Symptoms**: Setting parameter has no effect

**Solutions**:

1. **Check bounds and validation**
```python
# Value might be rejected silently
count = param.Integer(default=10, bounds=(1, 100))

# This won't work (out of bounds)
obj.count = 1000  # Silently ignored or raises error

# Check current value
print(obj.count)  # Still 10
```

2. **Verify parameter is not readonly**
```python
# Check parameter definition
value = param.Number(default=1.0, readonly=True)

# Can't set readonly parameters
obj.value = 2.0  # Error or ignored
```

3. **Use correct syntax**
```python
# Wrong
obj.param.value = 10  # This sets the Parameter object

# Correct
obj.value = 10  # This sets the parameter value
```

#### Issue: Watchers Not Firing

**Symptoms**: `@param.depends` or `.watch()` callbacks not executing

**Solutions**:

1. **Verify parameter name is correct**
```python
# Wrong
@param.depends('valu')  # Typo
def callback(self):
    pass

# Correct
@param.depends('value')
def callback(self):
    pass
```

2. **Check if value actually changed**
```python
# Watchers only fire on change
obj.value = 10
obj.value = 10  # No change, watcher won't fire
```

3. **Use correct watch syntax**
```python
# Wrong
obj.param.watch(callback)  # Missing parameter name

# Correct
obj.param.watch(callback, 'value')

# Or with decorator
@param.depends('value', watch=True)
def callback(self):
    pass
```

---

## hvPlot Issues

### Import and Setup

#### Issue: `hvplot` Accessor Not Available

**Symptoms**: `AttributeError: 'DataFrame' object has no attribute 'hvplot'`

**Solutions**:

1. **Import hvplot extension**
```python
# Wrong
import hvplot

# Correct - import for your data type
import hvplot.pandas  # For pandas DataFrames
import hvplot.xarray  # For xarray
import hvplot.dask    # For Dask
```

2. **Verify pandas/data library is installed**
```python
import pandas as pd
import hvplot.pandas

df = pd.DataFrame({'x': [1, 2, 3], 'y': [4, 5, 6]})
df.hvplot()  # Now works
```

### Plotting Issues

#### Issue: Geographic Plot Not Working

**Symptoms**: Error with `geo=True` or tiles not showing

**Solutions**:

1. **Install geoviews**
```bash
conda install geoviews
# or
pip install geoviews
```

2. **Ensure data has geographic coordinates**
```python
# Data must have lon/lat columns
df.hvplot.points('lon', 'lat', geo=True, tiles='OSM')
```

3. **Check coordinate ranges**
```python
# Coordinates should be valid ranges
# Latitude: -90 to 90
# Longitude: -180 to 180
print(df[['lon', 'lat']].describe())
```

---

## Common Error Messages

### `BokehUserWarning: ColumnDataSource's columns must be of the same length`

**Cause**: Mismatched array lengths in data

**Solution**:
```python
# Check data lengths
print({col: len(df[col]) for col in df.columns})

# Ensure all columns have same length
df = df.dropna()  # Remove rows with missing values
```

### `ValueError: data does not match declared dimensions`

**Cause**: Column names don't match HoloViews dimensions

**Solution**:
```python
# Check column names
print(df.columns.tolist())

# Use exact column names
scatter = hv.Scatter(df, 'x_column', 'y_column')  # Must match exactly
```

### `ImportError: Panel requires Bokeh version >=3.0`

**Cause**: Version mismatch

**Solution**:
```bash
# Update dependencies
conda update panel bokeh holoviews
# or
pip install -U panel bokeh holoviews
```

### `KeyError: 'column_name'`

**Cause**: Column doesn't exist in DataFrame

**Solution**:
```python
# Verify column exists
print(df.columns)

# Check for spacing/casing issues
print([col for col in df.columns if 'value' in col.lower()])
```

---

## Debugging Strategies

### Enable Debug Mode

```python
import panel as pn
import holoviews as hv

# Panel debug mode
pn.config.console_output = 'accumulate'

# HoloViews debug
hv.extension('bokeh', logo=False)
```

### Check Versions

```python
import panel as pn
import holoviews as hv
import datashader as ds
import param

print(f"Panel: {pn.__version__}")
print(f"HoloViews: {hv.__version__}")
print(f"Datashader: {ds.__version__}")
print(f"Param: {param.__version__}")
```

### Inspect Objects

```python
# Check element type
print(type(plot))

# View data
print(plot.data.head() if hasattr(plot, 'data') else 'No data attribute')

# Check dimensions
print(plot.kdims, plot.vdims)

# View options
print(hv.opts.Scatter.allowed())
```

### Browser Console

For Panel apps, check browser console (F12) for JavaScript errors:
- WebSocket connection issues
- JavaScript errors
- Network problems

---

## Getting Help

### Documentation

- [Panel Documentation](https://panel.holoviz.org)
- [HoloViews Documentation](https://holoviews.org)
- [Datashader Documentation](https://datashader.org)
- [GeoViews Documentation](https://geoviews.org)
- [Param Documentation](https://param.holoviz.org)

### Community Support

- [HoloViz Discourse](https://discourse.holoviz.org) - Q&A forum
- [GitHub Issues](https://github.com/holoviz/) - Bug reports
- [Stack Overflow](https://stackoverflow.com/questions/tagged/holoviz) - Tagged questions

### Debugging Checklist

Before asking for help:

1. [ ] Check version compatibility
2. [ ] Verify data format and contents
3. [ ] Review error messages carefully
4. [ ] Test with minimal reproducible example
5. [ ] Check browser console for JavaScript errors
6. [ ] Search documentation and issues
7. [ ] Prepare code snippet that reproduces issue

---

## Performance Profiling Tools

### Panel Profiling

```bash
# Time profiling
panel serve app.py --profiler pyinstrument

# Memory profiling
panel serve app.py --profiler memray

# Interactive profiling
panel serve app.py --profiler snakeviz
```

### Python Profiling

```python
# Time profiling
import cProfile
cProfile.run('expensive_function()')

# Memory profiling
from memory_profiler import profile

@profile
def expensive_function():
    # Your code
    pass
```

### Browser Performance

Use browser DevTools:
- Network tab: Check data transfer sizes
- Performance tab: Profile rendering
- Memory tab: Check for memory leaks

---

## Summary

Most common issues:
1. Missing extension calls
2. Incorrect parameter dependencies
3. Data format mismatches
4. CRS/projection problems
5. Performance bottlenecks
6. Version incompatibilities

Always start by:
1. Checking error messages carefully
2. Verifying data structure and contents
3. Ensuring correct library versions
4. Testing with minimal examples

## References

- [Datashader FAQ](https://datashader.org/FAQ.html)
- [Datashader Performance Guide](https://datashader.org/user_guide/Performance.html)
- [Datashader Plotting Pitfalls](https://datashader.org/user_guide/Plotting_Pitfalls.html)
- [Panel Profiling Guide](https://panel.holoviz.org/how_to/profiling/profile.html)
- [Panel How-To Guides](https://panel.holoviz.org/how_to/)
