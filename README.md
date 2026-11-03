DataProcessor

`DataProcessor` is a small, standard-library-only utility for ingesting numerical data, calculating common descriptive statistics, filtering and transforming values, and persisting datasets as CSV files. It is intended for lightweight workflows where a full data-analysis library would be unnecessary.

> **Source note:** The supplied `data_processor.py` source was not present in the workspace. The pasted attachment contains a request-log summarizer, rather than a `DataProcessor` implementation. This README documents the API and behaviors specified in the task brief; confirm details such as constructor validation and CSV row format against the actual source before relying on them as implementation guarantees.

## Requirements and setup

- Python 3.7 or newer
- Standard library modules: `math`, `csv`, and `typing` (no third-party packages required)

Place `data_processor.py` in the `level1` directory. From the parent directory, import it as:

```python
from level1.data_processor import DataProcessor
```

Or, from inside `level1`:

```python
from data_processor import DataProcessor
```

There is no separate installation step. The examples below can be saved as a `.py` file and run with `python example.py` from the parent directory.

## API reference

The signatures below reflect the API named in the task brief. Values are expected to be real numeric values; booleans are technically integers in Python, so whether the implementation accepts them depends on its validation logic.

### `DataProcessor(initial_data=None)`

Create a processor, optionally initialized with an iterable of numeric values.

| Parameter | Type | Description |
| --- | --- | --- |
| `initial_data` | `Iterable[float] \| None` | Initial values; omitted or `None` starts with an empty dataset. |

Returns a `DataProcessor` instance. Invalid initial values may raise `TypeError` or `ValueError`, depending on the implementation's validation policy.

### `add_data(item)`

Add one numeric value to the dataset.

- `item: int | float` — value to append.
- Returns: typically `None`.
- Invalid values may raise `TypeError` for an unsupported type or `ValueError` for a numeric value rejected by validation.

### `batch_add(items)`

Add multiple numeric values from an iterable.

- `items: Iterable[int | float]` — values to add.
- Returns: typically `None`.
- Invalid iterables or elements may raise `TypeError` or `ValueError`. Whether the operation is atomic (no values added if one element is invalid) is implementation-specific.

### `calculate_mean()`

Return the arithmetic mean as a `float`. An empty dataset has no defined mean and should be treated as an error; implementations commonly raise `ValueError`.

### `calculate_median()`

Return the median as an `int` or `float`, depending on the middle value(s). For an even number of values, the median is normally the average of the two middle sorted values. An empty dataset commonly raises `ValueError`.

### `calculate_variance()`

Return the variance as a `float`. Whether this is population variance (divide by `n`) or sample variance (divide by `n - 1`) must be checked in the implementation. Empty or undersized input may raise `ValueError`.

### `calculate_std_dev()`

Return the standard deviation as a `float`, normally the square root of the variance selected by `calculate_variance()`. Empty or undersized input may raise `ValueError`.

### `get_summary_statistics()`

Return a `dict` containing the processor's summary statistics, normally including count, mean, median, variance, and standard deviation. Exact keys and empty-data behavior are defined by the implementation; the example below shows a typical shape.

### `filter_data(min_val, max_val)`

Select values in the inclusive interval `[min_val, max_val]` and return them as a list (or the implementation's documented collection type). It is not specified whether this changes the processor's stored data; check the implementation. Reversed bounds or non-numeric bounds may raise `ValueError` or `TypeError`.

| Parameter | Type | Description |
| --- | --- | --- |
| `min_val` | `int | float` | Lower bound. |
| `max_val` | `int | float` | Upper bound. |

### `transform_data(factor, offset)`

Apply the linear transformation `value * factor + offset` to the dataset. The transformed values may be returned, stored in place, or both; verify the source's mutation behavior before depending on it.

| Parameter | Type | Description |
| --- | --- | --- |
| `factor` | `int | float` | Multiplier. |
| `offset` | `int | float` | Value added after multiplication. |

Invalid parameters may raise `TypeError` or `ValueError`.

### `export_to_csv(filepath)`

Write the dataset to a CSV file at `filepath`.

- `filepath: str | os.PathLike` — destination path.
- Returns: typically `None`.
- File-system failures (for example, an inaccessible directory) may raise `OSError` or a subclass such as `PermissionError`.

### `load_from_csv(filepath)`

Load numeric values from a CSV file at `filepath` into the processor.

- `filepath: str | os.PathLike` — source path.
- Returns: typically `None`, with data loaded into the instance.
- Missing/unreadable files may raise `OSError` or `FileNotFoundError`; malformed CSV or non-numeric cells may raise `csv.Error`, `ValueError`, or `TypeError`.
- Whether loading replaces existing values or appends them, and the expected CSV header/column layout, are implementation-specific.

## Examples

The examples use the methods named in the requested API. As noted above, mutation details, summary keys, and CSV layout should be checked against the source implementation.

### Ingest values and calculate statistics

```python
from level1.data_processor import DataProcessor

processor = DataProcessor([10, 20, 30])
processor.add_data(40)
processor.batch_add([50, 60])

print("mean:", processor.calculate_mean())
print("median:", processor.calculate_median())
print("variance:", processor.calculate_variance())
print("standard deviation:", processor.calculate_std_dev())
print("summary:", processor.get_summary_statistics())
```

### Filter and transform

```python
from level1.data_processor import DataProcessor

processor = DataProcessor([5, 10, 15, 20, 25])
print("from 10 through 20:", processor.filter_data(10, 20))

# A linear transform maps x to 2x + 3.
transformed = processor.transform_data(factor=2, offset=3)
print("transformed:", transformed)
```

If `transform_data` mutates in place and returns `None`, inspect the processor's documented data accessor or call its summary methods after transformation instead of assigning its return value.

### Export and import CSV

```python
from pathlib import Path
from level1.data_processor import DataProcessor

csv_path = Path("numbers.csv")
original = DataProcessor([1.5, 2.0, 3.25])
original.export_to_csv(csv_path)

loaded = DataProcessor()
loaded.load_from_csv(csv_path)
print(loaded.get_summary_statistics())
```

The exact file structure (header, delimiter, and supported number formats) depends on the implementation. Use files produced by `export_to_csv` as the safest input for `load_from_csv`.

## Error handling and edge cases

- **Empty dataset:** Statistics such as mean and median are mathematically undefined. Expect an exception, commonly `ValueError`, unless the implementation explicitly returns a sentinel such as `None`. Confirm how `get_summary_statistics()` handles an empty dataset.
- **Invalid types:** Add and batch operations are intended for numeric values. Strings, `None`, and other non-numeric objects should be rejected, commonly with `TypeError`; malformed numeric text may instead raise `ValueError` if conversion is attempted.
- **Non-finite numbers:** `NaN` and infinity are numeric floating-point values but can make statistics and filtering surprising. The task brief does not say whether they are accepted or rejected.
- **Filtering and transformation:** Bounds and transformation arguments should be numeric. Whether filtering returns a copy and whether transformation updates the stored dataset are source-defined behaviors.
- **CSV input:** Missing files and permission issues are file-system errors. Malformed rows, unexpected headers, or invalid numeric cells can produce parsing or conversion errors. The task brief does not specify partial-load behavior after a bad row.
- **CSV output:** Ensure the destination directory exists and is writable. The module may or may not create parent directories automatically.

Catch the specific errors relevant to the operation when calling the API. For example:

```python
try:
    mean = processor.calculate_mean()
except ValueError as exc:
    print("Cannot calculate a mean:", exc)
```
