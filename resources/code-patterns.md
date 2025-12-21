# HoloViz Code Patterns & Snippets

## Overview

Production-ready code patterns and snippets for common HoloViz tasks. These patterns follow best practices and have been tested in real-world applications.

---

## Panel Application Patterns

### Pattern 1: Param-Based Dashboard

**Use case**: Reactive dashboard with automatic updates

```python
import panel as pn
import param
import pandas as pd
import hvplot.pandas

pn.extension('tabulator')


class DataDashboard(param.Parameterized):
    """Dashboard with reactive parameter-based updates."""

    # Parameters
    data_source = param.Selector(
        default='sample',
        objects=['sample', 'file'],
        doc="Data source selection"
    )
    date_range = param.DateRange(
        doc="Filter data by date range"
    )
    aggregation = param.Selector(
        default='mean',
        objects=['mean', 'sum', 'count', 'std'],
        doc="Aggregation method"
    )

    def __init__(self, **params):
        super().__init__(**params)
        self._data = None

    @param.depends('data_source', watch=True)
    def _load_data(self):
        """Load data when source changes."""
        if self.data_source == 'sample':
            self._data = pd.DataFrame({
                'date': pd.date_range('2024-01-01', periods=100),
                'value': np.random.randn(100).cumsum(),
                'category': np.random.choice(['A', 'B', 'C'], 100)
            })

    @param.depends('data_source', 'date_range', 'aggregation')
    def summary_stats(self):
        """Compute summary statistics."""
        if self._data is None:
            self._load_data()

        df = self._data.copy()
        if self.date_range:
            df = df[(df.date >= self.date_range[0]) &
                    (df.date <= self.date_range[1])]

        agg_func = getattr(df.value, self.aggregation)
        return pn.pane.Markdown(f"**{self.aggregation.title()}**: {agg_func():.2f}")

    @param.depends('data_source', 'date_range')
    def time_series_plot(self):
        """Generate time series visualization."""
        if self._data is None:
            self._load_data()

        df = self._data.copy()
        if self.date_range:
            df = df[(df.date >= self.date_range[0]) &
                    (df.date <= self.date_range[1])]

        return df.hvplot.line(
            x='date', y='value', by='category',
            title='Time Series Data',
            responsive=True,
            height=400
        )

    @param.depends('data_source', 'date_range')
    def data_table(self):
        """Display data table."""
        if self._data is None:
            self._load_data()

        df = self._data.copy()
        if self.date_range:
            df = df[(df.date >= self.date_range[0]) &
                    (df.date <= self.date_range[1])]

        return pn.widgets.Tabulator(df, page_size=10)

    def view(self):
        """Create dashboard layout."""
        return pn.template.MaterialTemplate(
            title='Data Dashboard',
            sidebar=[
                pn.pane.Markdown('## Controls'),
                pn.Param(
                    self.param,
                    parameters=['data_source', 'date_range', 'aggregation'],
                    widgets={
                        'date_range': pn.widgets.DateRangePicker
                    }
                )
            ],
            main=[
                pn.Row(
                    pn.Card(self.summary_stats, title='Summary'),
                    pn.Card(self.time_series_plot, title='Visualization')
                ),
                pn.Card(self.data_table, title='Data Table')
            ]
        )


# Deployment
dashboard = DataDashboard()
dashboard.view().servable()
```

### Pattern 2: File Upload and Processing

**Use case**: Accept user files and process them

```python
import panel as pn
import param
import pandas as pd

pn.extension('tabulator')


class FileProcessor(param.Parameterized):
    """Process uploaded data files."""

    file_input = param.Parameter(doc="Uploaded file")
    column_selector = param.ListSelector(default=[], doc="Columns to analyze")

    def __init__(self, **params):
        super().__init__(**params)
        self._df = None

    @param.depends('file_input', watch=True)
    def _process_file(self):
        """Process uploaded file."""
        if self.file_input is None:
            self._df = None
            self.param.column_selector.objects = []
            return

        # Read file based on extension
        filename = self.file_input.filename
        if filename.endswith('.csv'):
            self._df = pd.read_csv(self.file_input.value)
        elif filename.endswith(('.xls', '.xlsx')):
            self._df = pd.read_excel(self.file_input.value)
        else:
            self._df = None
            return

        # Update available columns
        self.param.column_selector.objects = list(self._df.columns)

    @param.depends('file_input')
    def file_info(self):
        """Display file information."""
        if self._df is None:
            return pn.pane.Markdown("**No file uploaded**")

        info = f"""
        **File**: {self.file_input.filename}
        **Rows**: {len(self._df):,}
        **Columns**: {len(self._df.columns)}
        **Memory**: {self._df.memory_usage(deep=True).sum() / 1024**2:.2f} MB
        """
        return pn.pane.Markdown(info)

    @param.depends('file_input', 'column_selector')
    def preview_plot(self):
        """Create preview visualization."""
        if self._df is None or not self.column_selector:
            return pn.pane.Markdown("*Upload file and select columns*")

        return self._df[self.column_selector].hvplot.hist(
            bins=30, subplots=True, width=300, height=200
        )

    def panel(self):
        """Create UI."""
        file_widget = pn.widgets.FileInput(accept='.csv,.xlsx')
        self.param.watch(lambda e: setattr(self, 'file_input', e.new), 'file_input')

        return pn.Column(
            pn.pane.Markdown('# File Processor'),
            file_widget,
            self.file_info,
            pn.Param(
                self.param,
                parameters=['column_selector'],
                widgets={'column_selector': pn.widgets.MultiSelect}
            ),
            self.preview_plot
        )


FileProcessor().panel().servable()
```

### Pattern 3: Real-Time Data Streaming

**Use case**: Live updating dashboard

```python
import panel as pn
import param
import pandas as pd
import numpy as np
from datetime import datetime

pn.extension()


class LiveDashboard(param.Parameterized):
    """Dashboard with real-time data updates."""

    update_interval = param.Integer(default=1000, bounds=(100, 10000))
    max_points = param.Integer(default=100, bounds=(10, 1000))

    def __init__(self, **params):
        super().__init__(**params)
        self._data = pd.DataFrame({
            'time': pd.date_range(datetime.now(), periods=self.max_points, freq='1s'),
            'value': np.random.randn(self.max_points).cumsum()
        })
        self._callback = None

    def _update_data(self):
        """Append new data point."""
        new_row = pd.DataFrame({
            'time': [datetime.now()],
            'value': [self._data['value'].iloc[-1] + np.random.randn()]
        })
        self._data = pd.concat([self._data, new_row]).iloc[-self.max_points:]

    @param.depends('max_points')
    def plot(self):
        """Create streaming plot."""
        return self._data.hvplot.line(
            x='time', y='value',
            title='Real-Time Data Stream',
            responsive=True,
            height=400
        )

    @param.depends('update_interval', watch=True)
    def _setup_periodic_callback(self):
        """Configure periodic updates."""
        if self._callback:
            self._callback.stop()

        self._callback = pn.state.add_periodic_callback(
            self._update_data,
            period=self.update_interval
        )

    def view(self):
        """Create dashboard."""
        self._setup_periodic_callback()

        return pn.Column(
            pn.pane.Markdown('# Live Data Stream'),
            pn.Param(self.param, parameters=['update_interval', 'max_points']),
            self.plot
        )


LiveDashboard().view().servable()
```

---

## HoloViews Composition Patterns

### Pattern 4: Multi-Element Overlay

**Use case**: Combine multiple plot types

```python
import holoviews as hv
import pandas as pd
import numpy as np

hv.extension('bokeh')

# Generate data
df = pd.DataFrame({
    'x': np.linspace(0, 10, 100),
    'y': np.sin(np.linspace(0, 10, 100)) + np.random.randn(100) * 0.1
})

# Create individual elements
points = hv.Scatter(df, 'x', 'y').opts(
    color='blue', size=5, alpha=0.6, tools=['hover']
)

# Fit curve
from scipy.signal import savgol_filter
df['smoothed'] = savgol_filter(df['y'], 15, 3)
curve = hv.Curve(df, 'x', 'smoothed').opts(
    color='red', line_width=2, line_dash='dashed'
)

# Error band
df['upper'] = df['smoothed'] + 0.2
df['lower'] = df['smoothed'] - 0.2
area = hv.Area(df, 'x', vdims=['lower', 'upper']).opts(
    alpha=0.2, color='red'
)

# Combine with overlay operator
overlay = area * curve * points
overlay.opts(
    title='Data with Smoothed Fit',
    width=600, height=400,
    legend_position='top_right',
    show_legend=True
)
```

### Pattern 5: Faceted Visualization

**Use case**: Small multiples for comparison

```python
import holoviews as hv
import pandas as pd
import numpy as np

hv.extension('bokeh')

# Generate multi-category data
categories = ['A', 'B', 'C', 'D']
data = []
for cat in categories:
    for i in range(100):
        data.append({
            'category': cat,
            'x': i,
            'y': np.sin(i * 0.1 + hash(cat) % 10) + np.random.randn() * 0.1
        })

df = pd.DataFrame(data)

# Create faceted plot
dataset = hv.Dataset(df, ['category', 'x'], 'y')
layout = dataset.to(hv.Curve, 'x', 'y').layout('category').opts(
    hv.opts.Curve(width=300, height=200, tools=['hover']),
    hv.opts.Layout(shared_axes=True)
)

layout.cols(2)
```

### Pattern 6: Dynamic Map with Streams

**Use case**: Interactive parameter exploration

```python
import holoviews as hv
from holoviews import streams
import numpy as np

hv.extension('bokeh')


def generate_wave(frequency=1.0, amplitude=1.0, phase=0.0):
    """Generate parametric wave."""
    x = np.linspace(0, 4*np.pi, 1000)
    y = amplitude * np.sin(frequency * x + phase)
    return hv.Curve((x, y)).opts(width=600, height=400)


# Create parameter streams
frequency_stream = streams.Stream.define('Frequency', frequency=1.0)()
amplitude_stream = streams.Stream.define('Amplitude', amplitude=1.0)()
phase_stream = streams.Stream.define('Phase', phase=0.0)()

# Create dynamic map
dmap = hv.DynamicMap(
    generate_wave,
    streams=[frequency_stream, amplitude_stream, phase_stream]
)

# Add controls
freq_slider = pn.widgets.FloatSlider(
    name='Frequency', start=0.1, end=5, step=0.1, value=1.0
)
amp_slider = pn.widgets.FloatSlider(
    name='Amplitude', start=0.1, end=2, step=0.1, value=1.0
)
phase_slider = pn.widgets.FloatSlider(
    name='Phase', start=0, end=2*np.pi, step=0.1, value=0.0
)

# Link widgets to streams
freq_slider.link(frequency_stream, value='frequency')
amp_slider.link(amplitude_stream, value='amplitude')
phase_slider.link(phase_stream, value='phase')

# Layout
app = pn.Column(
    pn.pane.Markdown('# Interactive Wave Generator'),
    freq_slider, amp_slider, phase_slider,
    dmap
)
```

---

## Datashader Patterns

### Pattern 7: Large Dataset Visualization

**Use case**: Render millions of points efficiently

```python
import pandas as pd
import numpy as np
import holoviews as hv
from holoviews.operation.datashader import datashade, rasterize
import colorcet as cc

hv.extension('bokeh')

# Generate large dataset
n_points = 10_000_000
df = pd.DataFrame({
    'x': np.random.randn(n_points),
    'y': np.random.randn(n_points),
    'category': np.random.choice(['A', 'B', 'C'], n_points)
})

# Create points element
points = hv.Points(df, ['x', 'y'])

# Apply datashader
shaded = datashade(
    points,
    cmap=cc.fire,
    width=800,
    height=600
)

shaded.opts(
    title=f'Datashader: {n_points:,} Points',
    tools=['hover'],
    active_tools=['wheel_zoom']
)
```

### Pattern 8: Category-Aware Datashading

**Use case**: Preserve categories in large datasets

```python
import pandas as pd
import numpy as np
import holoviews as hv
from holoviews.operation.datashader import datashade
import colorcet as cc

hv.extension('bokeh')

# Large categorical data
n_points = 5_000_000
df = pd.DataFrame({
    'x': np.random.randn(n_points),
    'y': np.random.randn(n_points),
    'category': np.random.choice(['A', 'B', 'C'], n_points)
})

# Create points with category dimension
points = hv.Points(df, ['x', 'y'], 'category')

# Datashade by category
shaded = datashade(
    points,
    aggregator='count_cat',
    color_key=['red', 'blue', 'green'],
    width=800,
    height=600
)

shaded.opts(
    title='Categorical Datashader',
    tools=['hover']
)
```

### Pattern 9: Rasterize with Colorbar

**Use case**: Continuous value aggregation with legend

```python
import pandas as pd
import numpy as np
import holoviews as hv
from holoviews.operation.datashader import rasterize
import colorcet as cc

hv.extension('bokeh')

# Generate data with values
n_points = 1_000_000
df = pd.DataFrame({
    'x': np.random.randn(n_points),
    'y': np.random.randn(n_points),
    'value': np.random.randn(n_points)
})

# Create points with value dimension
points = hv.Points(df, ['x', 'y'], 'value')

# Rasterize to preserve values
rasterized = rasterize(points, aggregator='mean').opts(
    cmap=cc.coolwarm,
    colorbar=True,
    width=800,
    height=600,
    title='Mean Value Aggregation',
    tools=['hover']
)

rasterized
```

---

## GeoViews Geographic Patterns

### Pattern 10: Basic Choropleth Map

**Use case**: Geographic data with color encoding

```python
import geoviews as gv
import geopandas as gpd
from geoviews import tile_sources as gvts

gv.extension('bokeh')

# Load geographic data
gdf = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))
gdf = gdf[gdf.continent == 'Europe']

# Create choropleth
polygons = gv.Polygons(gdf, vdims=['name', 'pop_est']).opts(
    color='pop_est',
    cmap='viridis',
    colorbar=True,
    tools=['hover'],
    width=800,
    height=600,
    title='Population by Country'
)

# Add basemap
map_viz = gvts.CartoLight * polygons
map_viz
```

### Pattern 11: Point Data on Map

**Use case**: GPS coordinates with interactive basemap

```python
import geoviews as gv
import pandas as pd
from geoviews import tile_sources as gvts

gv.extension('bokeh')

# City locations
cities = pd.DataFrame({
    'city': ['London', 'Paris', 'Berlin', 'Madrid', 'Rome'],
    'lon': [-0.1276, 2.3522, 13.4050, -3.7038, 12.4964],
    'lat': [51.5074, 48.8566, 52.5200, 40.4168, 41.9028],
    'population': [9_000_000, 2_200_000, 3_600_000, 3_200_000, 2_800_000]
})

# Create points
points = gv.Points(cities, ['lon', 'lat'], ['city', 'population']).opts(
    color='population',
    size='population',
    cmap='plasma',
    scale=0.00001,
    colorbar=True,
    tools=['hover'],
    width=800,
    height=600
)

# Overlay on tile source
map_viz = gvts.OSM * points
map_viz.opts(title='European Cities by Population')
```

### Pattern 12: Multi-Layer Geographic Viz

**Use case**: Combine multiple geographic layers

```python
import geoviews as gv
import geopandas as gpd
from geoviews import tile_sources as gvts

gv.extension('bokeh')

# Load data
countries = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))
cities = gpd.read_file(gpd.datasets.get_path('naturalearth_cities'))

# Filter to region
countries = countries[countries.continent == 'Africa']
cities = cities[cities.geometry.within(countries.unary_union)]

# Create layers
country_layer = gv.Polygons(countries, vdims=['name']).opts(
    color='lightgray',
    line_color='black',
    alpha=0.5
)

city_layer = gv.Points(cities, vdims=['name']).opts(
    color='red',
    size=8,
    tools=['hover']
)

# Combine layers
map_viz = gvts.CartoLight * country_layer * city_layer
map_viz.opts(
    width=800,
    height=600,
    title='African Countries and Cities'
)
```

---

## Param Configuration Patterns

### Pattern 13: Hierarchical Configuration

**Use case**: Nested configuration with inheritance

```python
import param


class BaseConfig(param.Parameterized):
    """Base configuration shared across environments."""

    app_name = param.String(default='MyApp')
    debug = param.Boolean(default=False)
    log_level = param.Selector(
        default='INFO',
        objects=['DEBUG', 'INFO', 'WARNING', 'ERROR']
    )


class DatabaseConfig(param.Parameterized):
    """Database connection configuration."""

    host = param.String(default='localhost')
    port = param.Integer(default=5432, bounds=(1, 65535))
    database = param.String(default='mydb')
    pool_size = param.Integer(default=10, bounds=(1, 100))


class AppConfig(BaseConfig):
    """Application-specific configuration."""

    db = param.Parameter(default=DatabaseConfig(), instantiate=True)
    max_workers = param.Integer(default=4, bounds=(1, 32))
    cache_ttl = param.Integer(default=300, bounds=(0, 3600))

    @param.depends('debug', watch=True)
    def _update_log_level(self):
        """Auto-adjust log level based on debug mode."""
        if self.debug:
            self.log_level = 'DEBUG'
        else:
            self.log_level = 'INFO'


# Usage
config = AppConfig()
config.debug = True  # Automatically sets log_level to DEBUG
```

### Pattern 14: Dynamic Parameter Generation

**Use case**: Parameters that depend on runtime data

```python
import param


class DataSelector(param.Parameterized):
    """Dynamic column selection from DataFrame."""

    dataset_name = param.Selector(default='iris', objects=['iris', 'titanic'])
    x_column = param.Selector()
    y_column = param.Selector()

    def __init__(self, **params):
        super().__init__(**params)
        self._datasets = {
            'iris': pd.DataFrame({...}),  # Your data
            'titanic': pd.DataFrame({...})
        }
        self._update_columns()

    @param.depends('dataset_name', watch=True)
    def _update_columns(self):
        """Update available columns when dataset changes."""
        df = self._datasets[self.dataset_name]
        columns = list(df.columns)

        self.param.x_column.objects = columns
        self.param.y_column.objects = columns

        # Set defaults
        if columns:
            self.x_column = columns[0]
            self.y_column = columns[1] if len(columns) > 1 else columns[0]


selector = DataSelector()
```

---

## Integration Patterns

### Pattern 15: Panel + HoloViews + Datashader

**Use case**: Complete interactive big data application

```python
import panel as pn
import param
import pandas as pd
import numpy as np
import holoviews as hv
from holoviews.operation.datashader import datashade
import colorcet as cc

pn.extension()
hv.extension('bokeh')


class BigDataExplorer(param.Parameterized):
    """Explore large datasets interactively."""

    n_points = param.Integer(default=1_000_000, bounds=(10_000, 10_000_000))
    colormap = param.Selector(default='fire', objects=list(cc.palette.keys())[:10])
    alpha = param.Number(default=0.8, bounds=(0.1, 1.0))

    def __init__(self, **params):
        super().__init__(**params)
        self._generate_data()

    @param.depends('n_points', watch=True)
    def _generate_data(self):
        """Generate synthetic data."""
        self._df = pd.DataFrame({
            'x': np.random.randn(self.n_points),
            'y': np.random.randn(self.n_points)
        })

    @param.depends('n_points', 'colormap', 'alpha')
    def plot(self):
        """Create datashaded plot."""
        points = hv.Points(self._df, ['x', 'y'])
        shaded = datashade(
            points,
            cmap=getattr(cc, self.colormap),
            width=800,
            height=600
        ).opts(alpha=self.alpha)

        return shaded.opts(
            title=f'{self.n_points:,} Points',
            tools=['hover'],
            active_tools=['wheel_zoom', 'pan']
        )

    def stats(self):
        """Display statistics."""
        return pn.pane.Markdown(f"""
        **Data Statistics**
        - Points: {self.n_points:,}
        - X range: [{self._df.x.min():.2f}, {self._df.x.max():.2f}]
        - Y range: [{self._df.y.min():.2f}, {self._df.y.max():.2f}]
        - Memory: {self._df.memory_usage(deep=True).sum() / 1024**2:.2f} MB
        """)

    def panel(self):
        """Create application."""
        return pn.template.FastListTemplate(
            title='Big Data Explorer',
            sidebar=[
                pn.Param(
                    self.param,
                    parameters=['n_points', 'colormap', 'alpha']
                ),
                self.stats
            ],
            main=[self.plot]
        )


explorer = BigDataExplorer()
explorer.panel().servable()
```

---

## Performance Optimization Patterns

### Pattern 16: Memoization and Caching

**Use case**: Cache expensive computations

```python
import panel as pn
import param
from functools import lru_cache


class CachedDashboard(param.Parameterized):
    """Dashboard with cached computations."""

    param1 = param.Integer(default=10)
    param2 = param.String(default='A')

    @lru_cache(maxsize=128)
    def _expensive_computation(self, param1, param2):
        """Expensive function with caching."""
        # Simulate expensive operation
        import time
        time.sleep(1)
        return f"Result for {param1}, {param2}"

    @pn.cache  # Panel's caching decorator
    def load_data(self, filename):
        """Cache data loading."""
        return pd.read_csv(filename)

    @param.depends('param1', 'param2')
    def result(self):
        """Use cached computation."""
        return self._expensive_computation(self.param1, self.param2)
```

### Pattern 17: Progressive Disclosure

**Use case**: Load data incrementally

```python
import panel as pn
import param


class ProgressiveLoader(param.Parameterized):
    """Load and display data progressively."""

    load_more = param.Action(lambda self: self._load_next_batch())

    def __init__(self, **params):
        super().__init__(**params)
        self._batch_size = 1000
        self._current_index = 0
        self._all_data = range(10000)  # Large dataset
        self._loaded_data = []

    def _load_next_batch(self):
        """Load next batch of data."""
        end_index = min(
            self._current_index + self._batch_size,
            len(self._all_data)
        )
        batch = list(self._all_data[self._current_index:end_index])
        self._loaded_data.extend(batch)
        self._current_index = end_index

    @param.depends('load_more')
    def display(self):
        """Display loaded data."""
        return pn.pane.Markdown(
            f"Loaded {len(self._loaded_data)} / {len(self._all_data)} items"
        )
```

---

## Summary

These patterns cover:
- Panel application architecture
- HoloViews composition and interactivity
- Datashader for big data
- GeoViews geographic visualization
- Param configuration management
- Performance optimization
- Integration across libraries

Use these as starting points and adapt to your specific needs.

## References

- [Panel Documentation](https://panel.holoviz.org)
- [HoloViews Gallery](https://holoviews.org/gallery/index.html)
- [Datashader Examples](https://datashader.org/getting_started/index.html)
- [GeoViews User Guide](https://geoviews.org/user_guide/)
