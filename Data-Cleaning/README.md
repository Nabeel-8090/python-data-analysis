# Data Cleaning

Data cleaning is the process of detecting and fixing (or removing) incorrect, incomplete, duplicated, or irrelevant data. It's usually one of the first — and most time-consuming — steps in any data analysis workflow.

This covers three main areas:
1. **Missing data** (NumPy & Pandas)
2. **Invalid / not-null-but-wrong data**
3. **Duplicates & text cleaning**

---

## 1. Missing Data

### Falsy values in Python

```python
falsy_values = (0, False, None, '', [], {})
```

For Python, all the values above are considered **"falsy"**.

```python
any(falsy_values)   # False
```

> Note: `np.nan` is technically **truthy** — it's not in the falsy list, since it's still a "value" (just an undefined numeric one).

### `NaN` — Not a Number

`NaN` represents missing or undefined numerical data.

```python
3 + np.nan   # nan
```

Any arithmetic operation involving `NaN` returns `NaN` — it "propagates" through calculations, similar to a virus.

```python
a = np.array([1, 2, 3, np.nan, np.nan, 4])
a.sum()    # nan
a.mean()   # nan
```

`None` gets converted to `NaN` when placed inside a numeric NumPy array:

```python
a = np.array([1, 2, 3, np.nan, None, 4], dtype='float')
a
# array([ 1.,  2.,  3., nan, nan,  4.])
```

### Infinite values

NumPy also supports an "infinite" type:

```python
np.inf

3 + np.inf        # inf
np.inf / np.inf   # nan  (undefined result)
```

### Checking for NaN / Infinite

```python
np.isnan(np.nan)   # True
np.isinf(np.inf)   # True
```

### Filtering out missing values (NumPy)

Whenever you perform an operation on a NumPy array that might contain missing values, filter them out first to avoid `NaN` propagation:

```python
a = np.array([1, 2, 3, np.nan, np.nan, 4])

a[~np.isnan(a)]       # remove NaNs (negation of isnan)
a[np.isfinite(a)]     # keep only finite values (excludes NaN and inf)

a[np.isfinite(a)].sum()
a[np.isfinite(a)].mean()
```

---

## 2. Handling Missing Data with Pandas

Pandas provides cleaner, more graceful handling of missing values than raw NumPy.

### Checking for null values

```python
pd.isnull(np.nan)     # True
pd.isna(None)          # True
pd.notnull(None)       # False
```

These functions also work on Series and DataFrames:

```python
pd.isnull(pd.DataFrame({
    'Column A': [1, np.nan, 7],
    'Column B': [np.nan, 2, 3],
    'Column C': [np.nan, 2, np.nan]
}))
```

### Pandas operations ignore NaN (instead of propagating it)

Unlike NumPy, Pandas methods typically **skip** `NaN` values rather than letting them "infect" the whole result:

```python
pd.Series([1, 2, np.nan]).count()   # 2 (NaN excluded automatically)
```

### Dropping null values

```python
s
# 0    1.0
# 1    2.0
# 2    3.0
# 3    NaN
# 4    NaN
# 5    4.0
# dtype: float64

s.dropna()
# 0    1.0
# 1    2.0
# 2    3.0
# 5    4.0
```

For DataFrames:

```python
df = pd.DataFrame({
    'Column A': [1, np.nan, 30, np.nan],
    'Column B': [2, 8, 31, np.nan],
    'Column C': [np.nan, 9, 32, 100],
    'Column D': [5, 8, 34, 110],
})

df.dropna()
```

| `dropna()` option | Behavior |
|---|---|
| `how='any'` (default) | Drop row/column if **any** value is `NaN` |
| `how='all'` | Drop row/column only if **all** values are `NaN` |
| `thresh=N` | Keep rows/columns with at least `N` non-null values |
| `axis='rows'` (default) | Apply the drop logic across rows |
| `axis='columns'` | Apply the drop logic across columns |

```python
df.dropna(how='all')
df.dropna(how='any')                  # default
df.dropna(thresh=3)                    # default axis='rows'
df.dropna(thresh=3, axis='columns')
```

### Filling null values

Instead of dropping missing data, you can **fill** it — with `0`, the column mean, a forward/backward fill, or another sensible default. The right choice depends entirely on your dataset and context.

```python
s.fillna(0)
# 0    1.0
# 1    2.0
# 2    3.0
# 3    0.0
# 4    0.0
# 5    4.0

s.fillna(s.mean())     # fill with the column's average
```

Other useful fill strategies:

```python
s.fillna(method='ffill')   # forward-fill: propagate last valid value
s.fillna(method='bfill')   # backward-fill: use next valid value
```

---

## 3. Cleaning Not-Null but Invalid Values

Missing values (`NaN`) are easy to spot programmatically. But data can be **present and still wrong** — e.g. an age of `290`, or a "Sex" column containing `D` or `?` instead of `M`/`F`.

### Finding unique values

For categorical fields, start by inspecting the variety of values present:

```python
df = pd.DataFrame({
    'Sex': ['M', 'F', 'F', 'D', '?'],
    'Age': [29, 30, 24, 290, 25],
})

df['Sex'].unique()         # array(['M', 'F', 'D', '?'], dtype=object)
df['Sex'].value_counts()   # counts of each category
```

### Replacing invalid values

```python
df['Sex'].replace('D', 'F')
df['Sex'].replace({'D': 'F', 'N': 'M'})
```

Replace values across multiple columns at once:

```python
df.replace({
    'Sex': {
        'D': 'F',
        'N': 'M'
    },
    'Age': {
        290: 29
    }
})
```

---

## 4. Duplicates

### Duplicates in a Series

```python
ambassadors = pd.Series([
    'France',
    'United Kingdom',
    'United Kingdom',
    'Italy',
    'Germany',
    'Germany',
    'Germany',
], index=[
    'Gérard Araud',
    'Kim Darroch',
    'Peter Westmacott',
    'Armando Varricchio',
    'Peter Wittig',
    'Peter Ammon',
    'Klaus Scharioth',
])
```

**Detecting duplicates:**

```python
ambassadors.duplicated()             # default keep='first' -> marks all but the first occurrence
ambassadors.duplicated(keep='last')  # marks all but the last occurrence
ambassadors.duplicated(keep=False)   # marks ALL duplicates (no exceptions)
```

**Removing duplicates:**

```python
ambassadors.drop_duplicates()             # default keep='first'
ambassadors.drop_duplicates(keep='last')  # keeps the last occurrence
ambassadors.drop_duplicates(keep=False)   # drops every duplicated entry entirely
```

> Note: the correct method name is `drop_duplicates()` (not `drop_duplicated()`).

### Duplicates in DataFrames

In a DataFrame, duplicates are evaluated at the **row level** — two rows are duplicates only if *all* their values match.

```python
players = pd.DataFrame({
    'Name': [
        'Kobe Bryant',
        'LeBron James',
        'Kobe Bryant',
        'Carmelo Anthony',
        'Kobe Bryant',
    ],
    'Pos': [
        'SG',
        'SF',
        'SG',
        'SF',
        'SF'
    ]
})

players.duplicated()                       # full-row duplicate check
players.duplicated(subset=['Name'])        # only compare the 'Name' column
players.duplicated(subset=['Name'], keep='last')
```

---

## 5. Text Handling

Cleaning text values is often the hardest part of data cleaning — most invalid text comes from mistyping, which follows no predictable pattern. It's less common today since manual data entry has largely been replaced by automated systems, but it still shows up in surveys, legacy datasets, and free-text fields.

### Splitting columns

Suppose survey data was loaded into a single combined column:

```python
df = pd.DataFrame({
    'Data': [
        '1987_M_US _1',
        '1990?_M_UK_1',
        '1992_F_US_2',
        '1970?_M_   IT_1',
        '1985_F_I  T_2',
    ]
})
```

Split the string on the delimiter:

```python
df['Data'].str.split('_')                   # returns lists
df['Data'].str.split('_', expand=True)       # returns separate columns

df = df['Data'].str.split('_', expand=True)
df.columns = ['Year', 'Sex', 'Country', 'No Children']
```

### Cleaning up the split columns

```python
df['Year'].str.contains('\?')      # flag rows where Year has a '?'
df['Country'].str.strip()           # remove leading/trailing whitespace
df['Country'].str.replace(' ', '')  # remove all internal whitespace
```

Other handy string methods:

```python
df['Country'].str.lower()
df['Country'].str.upper()
df['Country'].str.len()
df['Country'].str.startswith('U')
```

---

## Complete Data Cleaning Flow (Summary)

1. **Load the data** → `pd.read_csv(...)`
2. **Inspect it** → `df.info()`, `df.describe()`, `df.head()`
3. **Check for missing values** → `df.isnull().sum()`
4. **Handle missing values** → `dropna()` or `fillna()`
5. **Check for invalid (not-null) values** → `df['col'].unique()`, `value_counts()`
6. **Fix invalid values** → `replace()`
7. **Check for duplicates** → `duplicated()`
8. **Remove duplicates** → `drop_duplicates()`
9. **Clean text/string columns** → `.str.split()`, `.str.strip()`, `.str.replace()`
10. **Re-verify** → run `df.info()` / `df.describe()` again to confirm the dataset is clean

---

## Quick Reference Cheat Sheet

| Task | Code |
|---|---|
| Check for NaN | `pd.isnull(x)` / `pd.isna(x)` |
| Check for not-null | `pd.notnull(x)` |
| Drop missing rows | `df.dropna()` |
| Drop only fully-empty rows | `df.dropna(how='all')` |
| Fill missing values | `df.fillna(value)` |
| Forward fill | `df.fillna(method='ffill')` |
| Unique values in a column | `df['col'].unique()` |
| Value frequency counts | `df['col'].value_counts()` |
| Replace specific values | `df['col'].replace(old, new)` |
| Detect duplicate rows | `df.duplicated()` |
| Remove duplicate rows | `df.drop_duplicates()` |
| Split a string column | `df['col'].str.split('_', expand=True)` |
| Strip whitespace | `df['col'].str.strip()` |
| Remove all spaces | `df['col'].str.replace(' ', '')` |

---

## Next: More Visualizations

Earlier notes covered the basics of Pandas' `.plot()` method and the core Matplotlib API. The next lesson goes deeper into visualization techniques for exploring cleaned data.