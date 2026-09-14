# Pandas

Pandas is a Python library built on top of **NumPy**, designed for working with labeled, tabular, and time-series data. It's the go-to tool for inspecting, cleaning, and transforming data before modeling or visualization.

---

## Pandas Series

A **Series** is a one-dimensional labeled array, capable of holding any data type (integers, strings, floats, objects, etc.).

```python
import pandas as pd

g7_pop = pd.Series([35.467, 63.951, 80.940, 60.665, 127.061, 64.511, 318.523])
g7_pop
```

```
0     35.467
1     63.951
2     80.940
3     60.665
4    127.061
5     64.511
6    318.523
dtype: float64
```

### Series values are basically NumPy arrays

```python
type(g7_pop.values)
# numpy.ndarray
```

This confirms that under the hood, a Series is just a NumPy array **plus an index** (labels) attached to it.

### Key properties of Series

- **Mutable** — values can be changed after creation
- Has an **index** (labels for each value) and **values** (the actual data, as a NumPy array)
- Can be created from lists, dicts, or NumPy arrays
- Supports custom indexing:

```python
g7_pop.index = [
    'Canada', 'France', 'Germany',
    'Italy', 'Japan', 'United Kingdom', 'United States'
]
```

### Indexing

```python
g7_pop['Canada']        # label-based access
g7_pop.iloc[0]           # position-based access
g7_pop[['Canada', 'Japan']]   # multiple labels
```

### Conditional Selection (Boolean Arrays)

```python
g7_pop[g7_pop > 70]
# Returns only the values greater than 70
```

You can combine conditions using `&` (and) / `|` (or):

```python
g7_pop[(g7_pop > 70) & (g7_pop < 200)]
```

### Operations and Methods

```python
g7_pop.mean()
g7_pop.std()
g7_pop * 1_000_000     # broadcasted operation, just like NumPy
g7_pop.sort_values()
g7_pop.sort_index()
```

---

## Pandas DataFrames

A **DataFrame** is a 2-dimensional labeled data structure — think of it as a table (rows + columns), or a dictionary of Series that share the same index.

```python
df = pd.DataFrame({
    'Population': [35.467, 63.951, 80.940],
    'GDP': [1785387, 2833687, 3874437],
    'Country': ['Canada', 'France', 'Germany']
})
```

### Common attributes

```python
df.shape        # (rows, columns)
df.columns      # column names
df.index        # row labels
df.dtypes       # data type of each column
df.info()       # summary of the DataFrame
df.describe()   # statistical summary (mean, std, min, max, etc.)
```

### Selecting data

```python
df['Population']          # select a single column (returns a Series)
df[['Population', 'GDP']] # select multiple columns (returns a DataFrame)

df.loc['Canada']          # select a row by label
df.iloc[0]                 # select a row by position

df.loc['Canada', 'GDP']   # select a specific cell
```

### Filtering rows (Boolean indexing)

```python
df[df['Population'] > 60]
```

---

## Reading Data: `pd.read_csv`

```python
df = pd.read_csv(
    'data/btc-market-price.csv',
    header=None,             # file has no header row
    names=['Timestamp', 'Price'],   # manually assign column names
    index_col=0,             # use the first column as the index
    parse_dates=True         # parse index as datetime objects
)
```

Useful `read_csv` parameters:

| Parameter | Purpose |
|---|---|
| `header` | Row number to use as column names (`None` if no header) |
| `names` | Manually specify column names |
| `index_col` | Column(s) to set as the DataFrame index |
| `parse_dates` | Automatically parse date columns into `datetime` objects |
| `sep` | Delimiter used in the file (default is `,`) |
| `usecols` | Load only specific columns |
| `nrows` | Limit the number of rows read (useful for large files) |

### Inspecting a loaded DataFrame

```python
df.head()      # first 5 rows
df.tail()      # last 5 rows
df.info()      # column types, non-null counts
df.describe()  # statistical summary
```

### Handling missing data

```python
df.isnull().sum()      # count missing values per column
df.dropna()             # drop rows with missing values
df.fillna(0)             # fill missing values with a default
```

### Grouping data

```python
df.groupby('Country')['Population'].mean()
```

---

## Matplotlib — Basic Plotting

```python
import matplotlib.pyplot as plt
```

### Plotting from a DataFrame directly

```python
df.plot()
```

Pandas DataFrames/Series have a built-in `.plot()` method (a shortcut over Matplotlib) — quick way to visualize without importing matplotlib explicitly.

### Plotting with Matplotlib directly

```python
plt.plot(df.index, df['Price'])
# general form: plt.plot(x, y)
```

### Titles, sizing, and styling

```python
plt.title('My Title')
plt.xlabel('Date')
plt.ylabel('Price (USD)')
plt.figure(figsize=(12, 6))   # width, height in inches
plt.show()
```

### Combining pandas `.plot()` with styling

```python
df.plot(figsize=(16, 9), title='Bitcoin Price 2017-2018')
```

---

## Quick Reference Cheat Sheet

| Task | Code |
|---|---|
| Create Series | `pd.Series([...])` |
| Create DataFrame | `pd.DataFrame({...})` |
| Read CSV | `pd.read_csv('file.csv')` |
| First/last rows | `df.head()` / `df.tail()` |
| Column selection | `df['col']` or `df[['col1', 'col2']]` |
| Row by label | `df.loc['label']` |
| Row by position | `df.iloc[0]` |
| Filter rows | `df[df['col'] > value]` |
| Summary stats | `df.describe()` |
| Missing values | `df.isnull().sum()` |
| Group and aggregate | `df.groupby('col').mean()` |
| Sort | `df.sort_values('col')` |
| Plot | `df.plot()` |