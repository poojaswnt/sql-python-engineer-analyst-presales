# Python 101 — A Complete Tutorial for Data Analysts

> **How to use this document**
> Read end to end. Every section builds on the previous one.
> All code runs with Python 3.8+ and standard library or common data-stack installs (flagged clearly).
> No API keys needed for any core example.
> Concept examples use 🎬 **Movies** throughout.
> Practice examples tie to the Kaggle datasets in `python_projects.md`.
> SQL equivalents are called out wherever they apply — building dual-language fluency.
>
> **Chat #2 · June 2026 (v2 — Expanded Edition)**

---

## Table of Contents

### Part I — Python Fundamentals
1. [Python Basics — Variables, Types, Operators](#1-python-basics)
2. [Strings — Manipulation, Formatting, Regex](#2-strings)
3. [Collections — Lists, Tuples, Dicts, Sets](#3-collections)
4. [Control Flow — If, For, While, Comprehensions](#4-control-flow)
5. [Functions — Defining, Args, Scope, Lambdas, Decorators](#5-functions)
6. [File I/O — Reading and Writing CSV, JSON, Text](#6-file-io)

### Part II — Data with pandas
7. [pandas Fundamentals — Series & DataFrame](#7-pandas-fundamentals)
8. [Loading Data — CSV, Excel, JSON, SQL, Parquet](#8-loading-data)
9. [Data Cleaning — Nulls, Types, Duplicates, Outliers, Strings](#9-data-cleaning)
10. [Filtering, Sorting, Selecting](#10-filtering-sorting-selecting)
11. [GroupBy, Aggregation, Pivot Tables, crosstab](#11-groupby-and-aggregation)
12. [Merging, Joining, Concatenating DataFrames](#12-merging-and-joining)
13. [Reshaping — melt, pivot, stack, unstack, explode](#13-reshaping)
14. [Window Functions & Time Series in pandas](#14-window-functions-and-time-series)
15. [Performance & Memory Optimisation](#15-performance-and-memory)

### Part III — NumPy
16. [NumPy — Arrays, Vectorised Ops, Broadcasting, Statistics](#16-numpy)

### Part IV — Visualisation
17. [Chart Selection Guide & Design Principles](#17-chart-selection-and-design)
18. [Matplotlib — Foundation & Full Reference](#18-matplotlib)
19. [Seaborn — Statistical Visualisation](#19-seaborn)

### Part V — EDA: Exploratory Data Analysis
20. [EDA — Concepts, Framework, and Full Workflow](#20-eda)

### Part VI — Python ↔ SQL
21. [sqlite3 — Running SQL from Python](#21-sqlite3)
22. [SQLAlchemy — Database Abstraction](#22-sqlalchemy)

### Part VII — Real-World Patterns
23. [Datetime Handling — Complete Guide](#23-datetime-handling)
24. [String Cleaning & Regex for Messy Data](#24-string-cleaning-and-regex)
25. [Error Handling & Writing Clean Code](#25-error-handling)

### Appendix
- [SQL-to-Python Mapping Cheatsheet](#appendix-a-sql-to-python-mapping)
- [pip Install Guide](#appendix-b-pip-install-guide)
- [Common Errors & Fixes](#appendix-c-common-errors-and-fixes)

---
---

## 1. Python Basics

### Why Python for data work?

Python dominates data analysis, data engineering, and data science for three reasons:
1. **Readable syntax** — code reads almost like English; `if score > 8: print("Great film")` needs no translation
2. **Ecosystem** — pandas, NumPy, Seaborn, scikit-learn, PySpark, and thousands more
3. **Interactive** — run one expression at a time in a notebook; see results immediately

**Running Python — three ways:**
```
python          →  REPL (interactive, great for exploration)
python file.py  →  run a script
Jupyter/Cursor  →  notebook cells (best for data work — mix code + output + notes)
```

---

### Variables and assignment

A variable is a named container for a value. Python figures out the type from the value — you don't declare it.

```python
# Assign values
movie_title  = "Inception"        # str
release_year = 2010               # int
rating       = 8.8                # float
is_released  = True               # bool
box_office   = None               # None  (no value yet — like SQL NULL)

# Multiple assignment in one line
x = y = z = 0          # all three set to 0

# Swap two variables (clean Python idiom)
a, b = 10, 20
a, b = b, a             # a=20, b=10 — no temp variable needed

# Unpack a sequence into variables
title, year, rating = ("Inception", 2010, 8.8)

# Check type
type(rating)            # <class 'float'>
type(movie_title)       # <class 'str'>
isinstance(rating, float)    # True
isinstance(rating, (int, float))  # True  — check against multiple types
```

---

### Data types — complete reference

| Type | Example | Key behaviour |
|------|---------|--------------|
| `int` | `42`, `-7`, `0`, `1_000_000` | Exact; no size limit in Python |
| `float` | `8.8`, `3.14`, `1e6`, `float('inf')` | Approximate (IEEE 754); avoid for money |
| `str` | `"hello"`, `'world'`, `"""multi\nline"""` | Immutable; indexed from 0 |
| `bool` | `True`, `False` | Subclass of int; `True == 1`, `False == 0` |
| `None` | `None` | Singleton; `x is None` to check |
| `complex` | `3+4j` | Rarely needed in data work |

```python
# Type conversion
int("42")            # → 42
int(8.9)             # → 8  (truncates — does NOT round)
round(8.9)           # → 9  (proper rounding)
round(8.567, 2)      # → 8.57  (round to 2 decimal places)
float("3.14")        # → 3.14
str(2010)            # → "2010"
bool(0)              # → False
bool("")             # → False
bool([])             # → False  — empty container is falsy
bool(None)           # → False
bool(1)              # → True
bool("hello")        # → True  — non-empty string is truthy
bool([1, 2])         # → True  — non-empty list is truthy

# ⚠️ Never use float for currency — floating point rounding errors
0.1 + 0.2            # 0.30000000000000004  ← not 0.3!
from decimal import Decimal
Decimal("0.1") + Decimal("0.2")   # Decimal('0.3')  ← exact

# Integer underscores for readability
population = 1_400_000_000    # one billion four hundred million
```

---

### Arithmetic operators

```python
# Basic
10 + 3    # 13     addition
10 - 3    # 7      subtraction
10 * 3    # 30     multiplication
10 / 3    # 3.333  true division — always returns float
10 // 3   # 3      floor division — rounds DOWN to integer
10 % 3    # 1      modulo — remainder after division
10 ** 3   # 1000   exponentiation

# Operator precedence: ** > unary - > * / // % > + -
# Use parentheses to be explicit
2 + 3 * 4       # 14  (not 20 — * before +)
(2 + 3) * 4     # 20

# Augmented assignment (modify in place)
count  = 0
count += 1       # 1
count -= 1       # 0
total  = 100
total *= 1.18    # 118.0  (add 18% tax)
total //= 1      # 118.0 → 118  (floor divide to drop decimal)

# Useful patterns
total      = 1500
tax        = total * 0.18       # 270.0
net        = total + tax         # 1770.0
pages      = 105
per_page   = 10
num_pages  = -(-pages // per_page)   # ceiling division trick: 11 (not 10)
# Or: import math; math.ceil(pages / per_page)
```

---

### Comparison and logical operators

```python
# Comparison — always returns bool
5 == 5       # True    equal
5 != 4       # True    not equal
5 > 3        # True    greater than
5 >= 5       # True    greater than or equal
5 < 3        # False   less than
5 <= 6       # True    less than or equal

# Python allows chained comparisons (unlike most languages)
0 < rating <= 10       # True if rating > 0 AND <= 10
2000 <= year <= 2024   # True if year is in range

# Logical operators
True and False    # False — both must be True
True or  False    # True  — at least one must be True
not True          # False

# Short-circuit evaluation — Python stops evaluating as soon as the result is known
False and expensive_function()   # expensive_function() is NEVER called
True  or  expensive_function()   # same — never called

# is vs == — crucial difference
a = [1, 2, 3]
b = [1, 2, 3]
a == b        # True  — same values
a is b        # False — different objects in memory

c = a
c is a        # True  — same object (c is just another name for a)

# None check: ALWAYS use 'is', never '=='
x = None
x is None     # True  ← correct
x == None     # True  but fragile — custom __eq__ could break it

# Truthy/falsy — Python evaluates these as False:
# False, 0, 0.0, 0j, "", b"", [], (), {}, set(), None
# Everything else is True

if df.empty:              # idiomatic check for empty DataFrame
    print("No data")
if not results:           # idiomatic check for empty list/dict
    print("No results")
```

---

## 2. Strings

### Indexing and slicing

```python
title = "The Dark Knight"
#        0123456789...

# Indexing — 0-based; negative counts from end
title[0]       # 'T'
title[4]       # 'D'
title[-1]      # 't'   (last character)
title[-6]      # 'K'

# Slicing: [start:stop:step]  — stop is EXCLUSIVE
title[0:3]     # 'The'
title[4:8]     # 'Dark'
title[4:]      # 'Dark Knight'     (omit stop = to end)
title[:3]      # 'The'             (omit start = from beginning)
title[:]       # 'The Dark Knight' (full copy)
title[::2]     # 'h akKih'         (every 2nd character)
title[::-1]    # 'thginK kraD ehT' (reversed)
title[-6:]     # 'Knight'          (last 6 characters)

# Strings are immutable — you can't change a character
# title[0] = 'X'   → TypeError
# Create a new string instead:
"X" + title[1:]    # 'Xhe Dark Knight'
```

---

### String methods — full toolkit

```python
s = "  Christopher Nolan  "

# ── CASE ──────────────────────────────────────────────────
s.upper()          # "  CHRISTOPHER NOLAN  "
s.lower()          # "  christopher nolan  "
s.title()          # "  Christopher Nolan  "
s.capitalize()     # "  christopher nolan  " → "  Christopher nolan  " (only first char)
s.swapcase()       # "  cHRISTOPHER nOLAN  "

# ── WHITESPACE ────────────────────────────────────────────
s.strip()          # "Christopher Nolan"       (both ends)
s.lstrip()         # "Christopher Nolan  "     (left only)
s.rstrip()         # "  Christopher Nolan"     (right only)
"***data***".strip("*")    # "data"            (strip specific chars)

# ── SEARCH ────────────────────────────────────────────────
s.strip().find("Nolan")         # 13   (-1 if not found)
s.strip().index("Nolan")        # 13   (ValueError if not found)
s.strip().count("o")            # 2
s.strip().startswith("Chris")   # True
s.strip().endswith("lan")       # True
"Nolan" in s                    # True  (membership — fast)

# ── REPLACE ───────────────────────────────────────────────
s.strip().replace("Christopher", "Chris")    # "Chris Nolan"
s.strip().replace("o", "0")                  # "Chr1st0pher N0lan"
# Replace only first 1 occurrence:
s.strip().replace("o", "0", 1)              # "Chr0stopher Nolan"

# ── SPLIT AND JOIN ────────────────────────────────────────
"Inception,2010,8.8".split(",")      # ['Inception', '2010', '8.8']
"hello world".split()                 # ['hello', 'world']  (any whitespace)
"a::b::c".split("::", 1)             # ['a', 'b::c']        (limit splits)
" | ".join(["Inception", "2010"])    # 'Inception | 2010'
",".join(str(x) for x in [1, 2, 3]) # '1,2,3'

# ── VALIDATION ────────────────────────────────────────────
"42".isdigit()           # True   — all characters are digits
"hello".isalpha()        # True   — all characters are letters
"hello42".isalnum()      # True   — letters and digits
"  ".isspace()           # True   — all whitespace
"42.5".replace(".","",1).isdigit()   # True — check if float string

# ── PADDING AND ALIGNMENT ─────────────────────────────────
"42".zfill(6)            # '000042'     zero-pad on left
"hi".ljust(10)           # 'hi        '  left-align, pad right
"hi".rjust(10)           # '        hi'  right-align, pad left
"hi".center(10, "-")     # '----hi----'  center with fill character

# ── MISC ──────────────────────────────────────────────────
"hello world".title()          # 'Hello World'
" ".join("hello".split())      # 'hello'  (normalise whitespace)
s.strip().partition("Nolan")   # ('Christopher ', 'Nolan', '')  — split at first match
```

---

### f-strings — full formatting reference

```python
title  = "Inception"
year   = 2010
rating = 8.8
revenue = 836_800_000

# Basic f-string
f"{title} ({year})"               # 'Inception (2010)'

# Numeric formatting
f"{rating:.2f}"                   # '8.80'    2 decimal places
f"{rating:.0f}"                   # '9'       0 decimal places (rounds)
f"{rating:05.1f}"                 # '008.8'   pad with zeros to width 5
f"{revenue:,}"                    # '836,800,000'   comma separator
f"{revenue:,.0f}"                 # '836,800,000'   comma, no decimals
f"${revenue/1e6:.1f}M"            # '$836.8M'

# Alignment
f"{'Title':<20}"                  # 'Title               '  left-align in 20 chars
f"{'Title':>20}"                  # '               Title'  right-align
f"{'Title':^20}"                  # '       Title        '  centre
f"{'Title':-^20}"                 # '-------Title-------'   centre with fill

# Expressions inside f-strings
f"Score: {rating * 10:.0f}/100"   # 'Score: 88/100'
f"High rated: {rating > 8}"       # 'High rated: True'
f"{title.upper()}"                # 'INCEPTION'
f"{len(title)} chars"             # '9 chars'

# Multi-line f-string
summary = (
    f"Title:     {title}\n"
    f"Year:      {year}\n"
    f"Rating:    {rating}/10\n"
    f"Revenue:   ${revenue/1e6:.1f}M"
)
print(summary)

# Debugging shortcut (Python 3.8+): = inside f-string prints name=value
f"{rating=}"    # 'rating=8.8'
f"{year=}"      # 'year=2010'
```

---

### Raw strings, multiline, and escape sequences

```python
# Escape sequences
"\n"    # newline
"\t"    # tab
"\\"    # literal backslash
"\""    # literal double quote inside double-quoted string
"\'"    # literal single quote

# Raw string — r prefix makes backslashes literal (no escape interpretation)
path = r"C:\Users\Pooja\data\file.csv"   # correct Windows path
print(path)   # C:\Users\Pooja\data\file.csv  (no escape interpretation)

# Without r prefix you'd need:
path = "C:\\Users\\Pooja\\data\\file.csv"   # cumbersome

# Triple-quoted strings — multiline
bio = """
Christopher Nolan is a British-American director.
He is known for Inception, The Dark Knight, and Oppenheimer.
Born: 1970.
"""

# String multiplication
separator = "-" * 40     # '----------------------------------------'
print(separator)
```

---

## 3. Collections

### Lists — ordered, mutable, allows duplicates

```python
# ── CREATE ────────────────────────────────────────────────
movies  = ["Inception", "Parasite", "Pulp Fiction", "Barbie"]
ratings = [8.8, 8.5, 8.9, 7.0]
mixed   = [1, "hello", True, None, 3.14]    # any types
nested  = [[1, 2], [3, 4], [5, 6]]          # list of lists (2D)
empty   = []

# ── ACCESS AND SLICE ──────────────────────────────────────
movies[0]          # "Inception"
movies[-1]         # "Barbie"
movies[1:3]        # ["Parasite", "Pulp Fiction"]
movies[::-1]       # reversed list

# ── MODIFY ────────────────────────────────────────────────
movies[0] = "Oppenheimer"          # replace by index
movies.append("Dune Part Two")     # add to end   ← O(1)
movies.insert(1, "The Matrix")     # insert at index ← O(n) — slower
movies.extend(["Alien", "Jaws"])   # add multiple items (from any iterable)
movies += ["1917", "Parasite"]     # same as extend

# ── REMOVE ────────────────────────────────────────────────
movies.remove("Barbie")      # remove first occurrence by VALUE (ValueError if not found)
del movies[0]                # remove by INDEX
popped = movies.pop()        # remove and return LAST item
popped = movies.pop(2)       # remove and return item at index 2
movies.clear()               # remove all items (faster than del movies[:])

# ── SEARCH ────────────────────────────────────────────────
"Parasite" in movies               # True/False  — O(n) scan
movies.index("Parasite")           # index of first match (ValueError if not found)
movies.count("Inception")          # count occurrences
any(m.startswith("The") for m in movies)   # True if any match
all(len(m) > 3 for m in movies)            # True if all match

# ── SORT ──────────────────────────────────────────────────
ratings.sort()                          # in-place ascending
ratings.sort(reverse=True)              # in-place descending
sorted_ratings = sorted(ratings)        # returns NEW list, original unchanged
movies.sort(key=len)                    # sort by string length
movies.sort(key=str.lower)              # case-insensitive sort
movies.sort(key=lambda m: (len(m), m)) # sort by length, then alphabetically

# ── STATISTICS AND UTILITIES ──────────────────────────────
len(movies)                  # count of items
min(ratings)                 # minimum value
max(ratings)                 # maximum value
sum(ratings)                 # total
sum(ratings) / len(ratings)  # average (or use statistics.mean)

list(range(10))              # [0, 1, 2, ..., 9]
list(range(1, 11))           # [1, 2, ..., 10]
list(range(0, 10, 2))        # [0, 2, 4, 6, 8]
list(range(10, 0, -1))       # [10, 9, ..., 1]  countdown

# ── COPY — THIS IS IMPORTANT ──────────────────────────────
original  = [1, 2, 3]
bad_copy  = original           # ← NOT a copy — SAME object in memory
bad_copy.append(4)             # modifies BOTH bad_copy AND original!

good_copy = original.copy()    # shallow copy
good_copy = original[:]        # slice copy (same effect)
good_copy = list(original)     # constructor copy

import copy
deep_copy = copy.deepcopy(nested)  # for nested lists — copies all levels
```

---

### Tuples — ordered, immutable

```python
# Create
record    = ("Inception", 2010, 8.8, "Nolan")
single    = (42,)           # ← trailing comma required for 1-item tuple
coords    = 10, 20          # parentheses optional
empty_t   = ()

# Access — same as list
record[0]           # "Inception"
record[-1]          # "Nolan"
record[1:3]         # (2010, 8.8)

# Unpack — very common in Python
title, year, rating, director = record
first, *rest = record        # first="Inception", rest=[2010, 8.8, "Nolan"]
*head, last  = record        # head=["Inception", 2010, 8.8], last="Nolan"

# Functions returning multiple values return a tuple
def stats(values):
    return min(values), max(values), sum(values) / len(values)

low, high, avg = stats([8.8, 7.5, 9.0])   # clean unpack

# Named tuple — tuple with field names (lightweight struct / record)
from collections import namedtuple
Movie = namedtuple("Movie", ["title", "year", "rating", "director"])
inception = Movie("Inception", 2010, 8.8, "Nolan")

# Access by name OR by position
inception.title     # "Inception"
inception.rating    # 8.8
inception[0]        # "Inception"

# Immutability: tuples as dict keys (lists can't be dict keys)
locations = {(28.6, 77.2): "Delhi", (51.5, -0.1): "London"}
```

---

### Dictionaries — key-value pairs, ordered (Python 3.7+)

```python
# ── CREATE ────────────────────────────────────────────────
movie = {
    "title":    "Inception",
    "year":     2010,
    "rating":   8.8,
    "director": "Christopher Nolan",
    "genres":   ["Sci-Fi", "Thriller", "Action"]
}

# From two lists
keys   = ["a", "b", "c"]
values = [1, 2, 3]
d = dict(zip(keys, values))     # {"a": 1, "b": 2, "c": 3}

# Keyword syntax
d = dict(title="Inception", year=2010)

# ── ACCESS ────────────────────────────────────────────────
movie["title"]                     # "Inception"  — KeyError if missing
movie.get("title")                 # "Inception"  — None if missing
movie.get("budget", "Unknown")     # "Unknown"    — custom default if missing
movie.get("genres", [])[0]        # "Sci-Fi"     — safe nested access

# ── MODIFY ────────────────────────────────────────────────
movie["rating"]   = 9.0            # update existing
movie["language"] = "English"      # add new key
del movie["language"]              # delete key (KeyError if missing)
movie.pop("language", None)        # delete and return; None if missing (safe)
movie.update({"rating": 9.0, "language": "English"})  # update multiple
movie |= {"rating": 9.0}           # Python 3.9+ merge operator

# ── MEMBERSHIP ────────────────────────────────────────────
"title"     in movie           # True  — checks KEYS
"Inception" in movie           # False — not in keys
"Inception" in movie.values()  # True  — linear scan of values

# ── ITERATE ───────────────────────────────────────────────
for key in movie:                          # iterate keys
    print(key)
for key, value in movie.items():          # iterate key-value pairs
    print(f"{key}: {value}")
for value in movie.values():              # iterate values
    print(value)

# ── DICT COMPREHENSION ────────────────────────────────────
squared     = {x: x**2 for x in range(6)}            # {0:0, 1:1, 2:4, 3:9, 4:16, 5:25}
filtered    = {k: v for k, v in movie.items() if v}  # exclude falsy values
inverted    = {v: k for k, v in scores.items()}       # flip keys and values

# ── PRACTICAL PATTERNS ────────────────────────────────────
# Counting occurrences
from collections import Counter
genres_list = ["Sci-Fi", "Drama", "Sci-Fi", "Thriller", "Drama", "Drama"]
counts = Counter(genres_list)      # Counter({'Drama': 3, 'Sci-Fi': 2, 'Thriller': 1})
counts.most_common(2)              # [('Drama', 3), ('Sci-Fi', 2)]

# Default dict — no KeyError for missing keys
from collections import defaultdict
genre_movies = defaultdict(list)
for title, genre in [("Inception", "Sci-Fi"), ("Parasite", "Drama"), ("Interstellar", "Sci-Fi")]:
    genre_movies[genre].append(title)
# defaultdict(list, {'Sci-Fi': ['Inception', 'Interstellar'], 'Drama': ['Parasite']})
```

---

### Sets — unordered, unique values, O(1) lookup

```python
# Create
genres  = {"Sci-Fi", "Thriller", "Drama"}
genres2 = set(["Action", "Drama", "Comedy"])
empty_s = set()          # ← NOT {} (that creates an empty dict)

# Add / remove
genres.add("Action")
genres.discard("Drama")     # no error if not present
genres.remove("Drama")      # KeyError if not present
genres.pop()                # remove and return arbitrary element

# ── SET OPERATIONS (mirrors SQL set operations) ───────────
a = {"Sci-Fi", "Thriller", "Drama"}
b = {"Drama", "Action", "Comedy"}

a | b          # {"Sci-Fi", "Thriller", "Drama", "Action", "Comedy"}  UNION
a & b          # {"Drama"}                                             INTERSECT
a - b          # {"Sci-Fi", "Thriller"}                               EXCEPT
a ^ b          # {"Sci-Fi", "Thriller", "Action", "Comedy"}           symmetric difference

a.issubset(b)        # False — is a ⊆ b?
a.issuperset(b)      # False — is a ⊇ b?
a.isdisjoint(b)      # False — do a and b share no elements?

# ── PRIMARY USE CASE: deduplication ───────────────────────
ids_with_dupes = [101, 202, 101, 303, 202, 101]
unique_ids = list(set(ids_with_dupes))     # [101, 202, 303]  (order not guaranteed)

# To deduplicate while preserving order:
seen = set()
unique_ordered = [x for x in ids_with_dupes if not (x in seen or seen.add(x))]
# Or simpler:
unique_ordered = list(dict.fromkeys(ids_with_dupes))    # [101, 202, 303]

# Fast membership testing — use set, not list, for large lookups
large_blacklist = set(range(1_000_000))
999_999 in large_blacklist   # O(1) — instant
999_999 in list(range(1_000_000))  # O(n) — scans up to 1M elements
```

---

## 4. Control Flow

### if / elif / else

```python
rating = 8.8

# Multi-way branch
if rating >= 9.0:
    tier = "Masterpiece"
elif rating >= 8.0:
    tier = "Great"
elif rating >= 7.0:
    tier = "Good"
else:
    tier = "Below average"

print(tier)   # Great

# One-liner ternary (inline if)
tier = "Good" if rating >= 7 else "Below average"
tier = ("Great" if rating >= 8 else "Good") if rating >= 7 else "Below average"

# Guard clauses — fail fast, avoid deep nesting
def process_movie(movie):
    if movie is None:
        return None          # guard: handle edge case early
    if not movie.get("title"):
        raise ValueError("Movie must have a title")
    # main logic here — only reached if guards pass
    return movie["title"].upper()
```

---

### for loops — all patterns

```python
movies = ["Inception", "Parasite", "Pulp Fiction"]

# Basic iteration
for movie in movies:
    print(movie)

# With index — ALWAYS use enumerate, never range(len(...))
for i, movie in enumerate(movies):
    print(f"{i+1}. {movie}")

# enumerate with custom start
for i, movie in enumerate(movies, start=1):
    print(f"Rank {i}: {movie}")

# Parallel iteration with zip
titles  = ["Inception", "Parasite", "Barbie"]
ratings = [8.8, 8.5, 7.0]
years   = [2010, 2019, 2023]

for title, rating, year in zip(titles, ratings, years):
    print(f"{title} ({year}): {rating}")

# zip stops at the SHORTEST iterable
# Use zip_longest to include all elements
from itertools import zip_longest
for t, r in zip_longest(titles, ratings[:2], fillvalue="N/A"):
    print(t, r)

# Dictionary iteration
movie_dict = {"Inception": 8.8, "Parasite": 8.5, "Barbie": 7.0}
for title, rating in movie_dict.items():
    print(f"{title}: {rating}")

# Nested loops
directors = ["Nolan", "Bong"]
genres    = ["Sci-Fi", "Drama"]
for d in directors:
    for g in genres:
        print(f"{d} + {g}")

# break / continue / else on loop
for movie in movies:
    if movie == "Pulp Fiction":
        break           # exit loop entirely

for movie in movies:
    if len(movie) > 10:
        continue        # skip this iteration
    print(movie)

# Loop else: runs if loop completed WITHOUT hitting break
for movie in movies:
    if movie == "Avengers":
        break
else:
    print("Avengers not found")   # prints because no break occurred
```

---

### while loops

```python
count = 0
while count < 5:
    print(count)
    count += 1

# Infinite loop with break
while True:
    line = input("Enter value (or 'done'): ")
    if line == "done":
        break
    process(line)

# while with complex condition
retries = 0
max_retries = 3
while retries < max_retries:
    try:
        data = fetch_data()
        break           # success — exit
    except ConnectionError:
        retries += 1
        print(f"Retry {retries}/{max_retries}")
```

---

### Comprehensions — Python's most powerful pattern

```python
movies  = ["Inception", "Parasite", "Pulp Fiction", "Barbie"]
ratings = [8.8, 8.5, 8.9, 7.0]

# ── LIST COMPREHENSION ────────────────────────────────────
# [expression for item in iterable if condition]

upper    = [m.upper() for m in movies]                    # transform all
long     = [m for m in movies if len(m) > 8]             # filter
scores   = [r * 10 for r in ratings if r >= 8]           # transform + filter

# Equivalent for-loop (comprehension is faster and more Pythonic)
scores = []
for r in ratings:
    if r >= 8:
        scores.append(r * 10)

# Nested comprehension
matrix   = [[1,2,3],[4,5,6],[7,8,9]]
flat     = [x for row in matrix for x in row]             # [1,2,3,4,5,6,7,8,9]
transpos = [[row[i] for row in matrix] for i in range(3)] # transpose

# ── DICT COMPREHENSION ────────────────────────────────────
rating_map   = {t: r for t, r in zip(movies, ratings)}   # {"Inception": 8.8, ...}
high_only    = {t: r for t, r in rating_map.items() if r > 8}
score_lookup = {t: round(r * 10) for t, r in rating_map.items()}

# ── SET COMPREHENSION ─────────────────────────────────────
first_chars  = {m[0] for m in movies}     # {'I', 'P', 'B'}  (unique first letters)
unique_years = {m["year"] for m in movie_list}

# ── GENERATOR EXPRESSION ─────────────────────────────────
# Like a list comprehension but LAZY — doesn't build the list in memory
# Use when you only need to iterate once or compute an aggregate
total  = sum(r * 10 for r in ratings)            # more memory-efficient
avg    = sum(r for r in ratings) / len(ratings)
top    = max(r for r in ratings if r > 7)
any_over_9 = any(r > 9 for r in ratings)
all_positive = all(r > 0 for r in ratings)

# For large files — process line by line without loading everything
total_lines = sum(1 for _ in open("big_file.csv"))
```

---

## 5. Functions

### Defining functions — full reference

```python
# Basic
def greet(name):
    """Docstring explains what the function does.
    
    Parameters
    ----------
    name : str
        The person's name.
    
    Returns
    -------
    str
        Greeting message.
    """
    return f"Hello, {name}!"

# Multiple return values (tuple)
def min_max_avg(values):
    return min(values), max(values), sum(values) / len(values)

lo, hi, avg = min_max_avg([8.8, 7.5, 9.0, 8.2])
```

---

### All argument types

```python
# 1. Positional arguments — order matters
def describe(title, year, rating):
    return f"{title} ({year}): {rating}/10"

describe("Inception", 2010, 8.8)              # positional
describe(year=2010, title="Inception", rating=8.8)  # keyword (any order)

# 2. Default arguments — must come AFTER positional
def describe(title, year=2023, rating=None, language="English"):
    r = f"{rating}/10" if rating else "unrated"
    return f"{title} ({year}): {r} [{language}]"

describe("Dune Part Two")                     # uses all defaults
describe("Dune Part Two", rating=8.3)         # override just rating

# 3. *args — variable number of positional arguments
def total(*amounts):
    return sum(amounts)

total(10, 20)          # 30
total(10, 20, 30, 40)  # 100

# *args collects into a tuple
def log(*messages):
    for i, msg in enumerate(messages, 1):
        print(f"  [{i}] {msg}")

# 4. **kwargs — variable number of keyword arguments
def tag_movie(**attributes):
    return ", ".join(f"{k}={v}" for k, v in attributes.items())

tag_movie(title="Inception", year=2010, rating=8.8)
# 'title=Inception, year=2010, rating=8.8'

# 5. Positional-only (/) and keyword-only (*) — Python 3.8+
def calculate(a, b, /, operation, *, round_result=False):
    # a, b are positional-only (before /)
    # operation must be positional
    # round_result is keyword-only (after *)
    result = a + b if operation == "add" else a - b
    return round(result) if round_result else result

calculate(10, 5, "add", round_result=True)   # 15

# 6. Full signature with all types
def full_function(pos1, pos2, *args, kw_only=True, **kwargs):
    print(f"pos: {pos1}, {pos2}")
    print(f"args: {args}")
    print(f"kw_only: {kw_only}")
    print(f"kwargs: {kwargs}")

full_function(1, 2, 3, 4, kw_only=False, extra="hello")
# pos: 1, 2 | args: (3, 4) | kw_only: False | kwargs: {'extra': 'hello'}
```

---

### Lambda functions

```python
# lambda arguments: expression
double = lambda x: x * 2
double(5)      # 10

# Most common use: as key= argument for sorting
movies = [("Inception", 8.8), ("Parasite", 8.5), ("Barbie", 7.0)]

movies.sort(key=lambda x: x[1])               # sort by rating (index 1)
movies.sort(key=lambda x: x[1], reverse=True) # sort descending

# Sort by multiple criteria
movies.sort(key=lambda x: (-x[1], x[0]))      # rating desc, then title asc

# With map / filter — though list comprehensions are usually cleaner
ratings = [8.8, 7.5, 9.0, 8.2]
scaled  = list(map(lambda r: round(r * 10), ratings))  # [88, 75, 90, 82]
high    = list(filter(lambda r: r > 8.5, ratings))     # [8.8, 9.0]

# Rule: if a lambda gets complex (needs conditions, multiple lines), write a def
# Lambda: key=lambda x: x[1]   ← OK
# Lambda: key=lambda x: x[1] if x[1] else 0  ← still OK
# Lambda: (complex multi-step logic)  ← write a def
```

---

### Variable scope (LEGB)

```python
# LEGB: Local → Enclosing → Global → Built-in
# Python looks up variables in this order

GLOBAL_THRESHOLD = 8.0        # global

def rate_movie(rating):
    label = "good" if rating >= GLOBAL_THRESHOLD else "poor"  # uses global
    return label

# global keyword — modify a global inside a function (use rarely)
counter = 0
def increment():
    global counter
    counter += 1

# nonlocal — modify enclosing function's variable
def outer():
    total = 0
    def inner(value):
        nonlocal total
        total += value
    inner(10)
    inner(20)
    return total    # 30

# Closures — inner function remembers its enclosing scope
def make_adder(n):
    def adder(x):
        return x + n      # n is captured from enclosing scope
    return adder

add5  = make_adder(5)
add10 = make_adder(10)
add5(3)    # 8
add10(3)   # 13
```

---

### Decorators — functions that wrap functions

```python
import functools
import time

# A decorator is a function that takes a function and returns a wrapped version
def timer(func):
    @functools.wraps(func)    # preserves original function's name and docstring
    def wrapper(*args, **kwargs):
        start  = time.time()
        result = func(*args, **kwargs)
        end    = time.time()
        print(f"{func.__name__} took {end - start:.3f}s")
        return result
    return wrapper

@timer
def slow_analysis(df):
    # ... some expensive operation ...
    return df.groupby("category")["revenue"].sum()

# @timer is shorthand for: slow_analysis = timer(slow_analysis)

# Practical data decorator: validate DataFrame input
def requires_columns(*cols):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(df, *args, **kwargs):
            missing = [c for c in cols if c not in df.columns]
            if missing:
                raise ValueError(f"Missing columns: {missing}")
            return func(df, *args, **kwargs)
        return wrapper
    return decorator

@requires_columns("customer_id", "order_date", "revenue")
def compute_rfm(df):
    # ... safe to assume these columns exist ...
    pass
```

---

## 6. File I/O

### Text files

```python
# Write
with open("movies.txt", "w", encoding="utf-8") as f:
    f.write("Inception\n")
    f.writelines(["Parasite\n", "Pulp Fiction\n"])   # write multiple lines

# Append (doesn't overwrite)
with open("movies.txt", "a") as f:
    f.write("Barbie\n")

# Read — all at once
with open("movies.txt", "r", encoding="utf-8") as f:
    content = f.read()       # one big string
    lines   = f.readlines()  # list of lines (includes \n)

# Read — line by line (memory-efficient for large files)
with open("movies.txt", "r") as f:
    for line in f:
        print(line.strip())    # .strip() removes trailing \n
```

---

### CSV files — the analyst's bread and butter

```python
import csv

# Write
data = [
    ["title",     "year", "rating"],
    ["Inception", 2010,   8.8],
    ["Parasite",  2019,   8.5],
]
with open("movies.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerows(data)

# Read as list of dicts (column name → value)
with open("movies.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["title"], float(row["rating"]))
# {'title': 'Inception', 'year': '2010', 'rating': '8.8'}
# Note: ALL values are strings — convert explicitly

# In practice, always use pandas for CSV (chapter 8)
# csv module is for: no-pandas environments, very large files (streaming), custom delimiters
```

---

### JSON

```python
import json

movie = {
    "title":  "Inception",
    "year":   2010,
    "rating": 8.8,
    "genres": ["Sci-Fi", "Thriller"]
}

# Write to file
with open("movie.json", "w") as f:
    json.dump(movie, f, indent=2, ensure_ascii=False)   # indent=2 for readable; ensure_ascii=False for Unicode

# Read from file
with open("movie.json", "r") as f:
    loaded = json.load(f)

# Convert to/from string (API responses come as strings)
json_string  = json.dumps(movie)              # dict → string
back_to_dict = json.loads(json_string)        # string → dict

# Handle non-serialisable types (dates, numpy, etc.)
import json
from datetime import datetime
import numpy as np

class DataEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, np.integer):
            return int(obj)
        if isinstance(obj, np.floating):
            return float(obj)
        return super().default(obj)

json.dumps({"date": datetime.now(), "count": np.int64(42)}, cls=DataEncoder)
```

---

### pathlib — modern file system operations

```python
from pathlib import Path

# Build paths (handles / vs \ cross-platform)
base     = Path("data")
olist    = base / "olist"
orders   = olist / "orders.csv"

# Construct from string
p = Path(r"C:\Users\Pooja\data")   # Windows
p = Path("/home/pooja/data")       # Linux/Mac

# Check
orders.exists()           # True/False
orders.is_file()          # True if regular file
olist.is_dir()            # True if directory

# File info
orders.name               # 'orders.csv'
orders.stem               # 'orders'
orders.suffix             # '.csv'
orders.parent             # Path('data/olist')
orders.stat().st_size     # file size in bytes

# Create directories
olist.mkdir(parents=True, exist_ok=True)   # create including parents, no error if exists

# List files
for f in base.iterdir():           # all items in directory
    print(f)
csvs = list(base.glob("*.csv"))   # all CSVs in base
csvs = list(base.rglob("*.csv"))  # all CSVs recursively (subdirectories too)

# Read / write via pathlib
text = orders.read_text(encoding="utf-8")      # read entire file as string
orders.write_text("content", encoding="utf-8") # write entire file

# Combine with pandas
import pandas as pd
for csv_file in Path("data/olist").glob("*.csv"):
    df = pd.read_csv(csv_file)
    print(f"{csv_file.name}: {df.shape}")
```

---
---

## 7. pandas Fundamentals

### What is pandas?

pandas gives you two data structures:
- **Series** — 1D labelled array (one column of data)
- **DataFrame** — 2D labelled table (rows + columns)

Think of a DataFrame as a SQL table you can manipulate with code. Every operation you'd write in SQL has a pandas equivalent — and this document will call them out explicitly.

```python
pip install pandas
import pandas as pd
pd.set_option("display.max_columns", 50)        # show all columns
pd.set_option("display.float_format", "{:.2f}".format)  # 2dp for floats
```

---

### Series — deep dive

```python
import pandas as pd

# Create
ratings = pd.Series([8.8, 8.5, 8.9, 7.0, 9.0])
# 0    8.8
# 1    8.5   ← default integer index
# dtype: float64

# Named index (like a dict with order)
ratings = pd.Series(
    [8.8, 8.5, 8.9, 7.0],
    index=["Inception", "Parasite", "Pulp Fiction", "Barbie"],
    name="imdb_rating"
)

# From dict — dict keys become the index
ratings = pd.Series({"Inception": 8.8, "Parasite": 8.5, "Barbie": 7.0})

# Access
ratings["Inception"]               # 8.8  — label access
ratings[["Inception", "Parasite"]] # two values  — returns Series
ratings[0]                         # 8.8  — integer position (if integer index)

# Slicing
ratings["Inception":"Parasite"]    # inclusive slice on labels
ratings.iloc[0:2]                  # position-based slice

# Boolean filter — returns a Series
ratings[ratings > 8.0]             # Inception: 8.8, Parasite: 8.5
ratings[(ratings > 8.0) & (ratings < 9.0)]

# Arithmetic — vectorised, no loop needed
ratings * 10                       # all × 10
ratings + 0.5                      # all + 0.5
ratings > 8.0                      # boolean Series (mask)

# Statistics
ratings.mean()        # 8.2
ratings.median()      # 8.65
ratings.std()         # 0.83
ratings.min()         # 7.0
ratings.max()         # 8.9
ratings.sum()
ratings.quantile(0.75)  # 75th percentile
ratings.describe()    # count, mean, std, min, 25%, 50%, 75%, max
ratings.value_counts()  # frequency count of each unique value
ratings.nunique()     # count of unique values
ratings.isnull().sum()  # count NaN values
```

---

### DataFrame — complete creation patterns

```python
# From dict of lists
movies_df = pd.DataFrame({
    "title":      ["Inception", "Parasite", "Pulp Fiction", "Barbie"],
    "year":       [2010, 2019, 1994, 2023],
    "rating":     [8.8, 8.5, 8.9, 7.0],
    "director":   ["Nolan", "Bong", "Tarantino", "Gerwig"],
    "box_office": [836.8, 263.1, 214.2, 1441.1]   # millions
})

# From list of dicts (each dict = one row)
movies_df = pd.DataFrame([
    {"title": "Inception",   "year": 2010, "rating": 8.8},
    {"title": "Parasite",    "year": 2019, "rating": 8.5},
    {"title": "Pulp Fiction","year": 1994, "rating": 8.9},
])

# From list of tuples with column names
movies_df = pd.DataFrame(
    [("Inception", 2010, 8.8), ("Parasite", 2019, 8.5)],
    columns=["title", "year", "rating"]
)

# From CSV (see Chapter 8)
movies_df = pd.read_csv("movies.csv")

# Empty DataFrame with schema
df = pd.DataFrame(columns=["order_id", "customer_id", "amount", "date"])
```

---

### Inspecting a DataFrame — first things to do

```python
# ── SIZE AND STRUCTURE ────────────────────────────────────
movies_df.shape           # (4, 5)  — rows × columns
movies_df.ndim            # 2
len(movies_df)            # 4  — row count
movies_df.size            # 20 — total cells

# ── COLUMN INFO ───────────────────────────────────────────
movies_df.columns         # Index(['title', 'year', 'rating', 'director', 'box_office'])
movies_df.dtypes          # dtype per column
movies_df.index           # RangeIndex(start=0, stop=4, step=1)
movies_df.info()          # column names, non-null count, dtype, memory usage

# ── FIRST LOOK AT DATA ────────────────────────────────────
movies_df.head(5)         # first 5 rows (default)
movies_df.tail(3)         # last 3 rows
movies_df.sample(2)       # 2 random rows
movies_df.sample(frac=0.1)  # 10% random sample

# ── STATISTICS ────────────────────────────────────────────
movies_df.describe()      # count, mean, std, min, quartiles, max  (numeric only)
movies_df.describe(include="object")   # for string columns: count, unique, top, freq
movies_df.describe(include="all")      # all columns

movies_df["rating"].mean()
movies_df["rating"].median()
movies_df["rating"].mode()[0]          # most frequent value
movies_df["year"].value_counts()
movies_df["director"].nunique()        # count unique values
```

---

### loc and iloc — the complete guide

```python
# KEY RULE:
# loc  = LABEL-based  (column names, index labels)
# iloc = INTEGER-based (position 0, 1, 2... regardless of index label)

# ── iloc: position-based ──────────────────────────────────
movies_df.iloc[0]            # first row as Series
movies_df.iloc[-1]           # last row
movies_df.iloc[0:2]          # rows 0 and 1 (stop exclusive — like Python slice)
movies_df.iloc[0, 2]         # row 0, column 2 — single value
movies_df.iloc[[0, 2, 3]]    # specific rows: 0, 2, 3
movies_df.iloc[:, 0:3]       # all rows, columns 0-2
movies_df.iloc[:, -1]        # all rows, last column
movies_df.iloc[[0,1], [0,2]] # rows 0,1 and columns 0,2

# ── loc: label-based ──────────────────────────────────────
movies_df.loc[0, "title"]         # row 0, column 'title' — "Inception"
movies_df.loc[0:2, "title":"rating"]  # rows 0-2, columns title through rating
                                       # ⚠️ loc INCLUSIVE on both ends (unlike iloc)
movies_df.loc[:, "rating"]        # all rows, rating column
movies_df.loc[movies_df["rating"] > 8, "title"]  # rows where rating>8, title column

# ── at / iat — fastest single value access ────────────────
movies_df.at[0, "title"]    # label-based single cell
movies_df.iat[0, 0]         # position-based single cell

# ── Setting values with loc ───────────────────────────────
movies_df.loc[0, "rating"]  = 9.0           # update one cell
movies_df.loc[movies_df["year"] < 2000, "era"] = "Classic"  # conditional set

# ── SettingWithCopyWarning — how to avoid it ──────────────
# ❌ Can trigger warning — modifying a slice
subset = movies_df[movies_df["year"] > 2000]
subset["new_col"] = "modern"                 # WARNING — may not modify original

# ✅ Use .copy() explicitly
subset = movies_df[movies_df["year"] > 2000].copy()
subset["new_col"] = "modern"                 # safe — separate object

# ✅ Or use .loc on the original
movies_df.loc[movies_df["year"] > 2000, "new_col"] = "modern"
```

---

### Adding and modifying columns — all patterns

```python
# Simple assignment
movies_df["score_100"]  = movies_df["rating"] * 10
movies_df["is_modern"]  = movies_df["year"] >= 2000
movies_df["revenue_B"]  = (movies_df["box_office"] / 1000).round(2)

# Conditional column with np.where (CASE WHEN equivalent)
import numpy as np
movies_df["tier"] = np.where(movies_df["rating"] >= 8.5, "Top Tier", "Standard")

# Multi-level CASE WHEN with np.select
conditions = [
    movies_df["rating"] >= 9.0,
    movies_df["rating"] >= 8.0,
    movies_df["rating"] >= 7.0,
]
choices = ["Masterpiece", "Great", "Good"]
movies_df["tier"] = np.select(conditions, choices, default="Average")

# Method chaining with assign() — functional style, no mutation
result = (
    movies_df
    .assign(score_100  = lambda df: df["rating"] * 10)
    .assign(is_modern  = lambda df: df["year"] >= 2000)
    .assign(tier       = lambda df: np.select(
        [df["rating"] >= 9, df["rating"] >= 8],
        ["Masterpiece", "Great"],
        default="Good"
    ))
    .query("is_modern == True")
    .sort_values("score_100", ascending=False)
)

# apply() — most flexible, slower (runs a Python function per element/row)
movies_df["title_len"]   = movies_df["title"].apply(len)
movies_df["title_upper"] = movies_df["title"].apply(str.upper)
movies_df["rating_str"]  = movies_df["rating"].apply(lambda r: f"{r:.1f}/10")

# apply across rows (axis=1) — row-wise
movies_df["display"] = movies_df.apply(
    lambda row: f"{row['title']} ({row['year']}) — {row['rating']:.1f}",
    axis=1
)

# map() — fastest for element-wise on a Series (uses a dict or function)
tier_map = {2010: "2010s", 2019: "2010s", 1994: "1990s", 2023: "2020s"}
movies_df["decade"] = movies_df["year"].map(tier_map)
# map returns NaN for missing keys — use fillna() after if needed

# ── RENAME, DROP, REORDER ─────────────────────────────────
movies_df.rename(columns={"box_office": "box_office_usd"}, inplace=True)
movies_df.rename(columns=str.lower, inplace=True)  # all columns to lowercase
movies_df.columns = movies_df.columns.str.lower().str.replace(" ", "_").str.strip()  # clean all

movies_df.drop(columns=["score_100", "revenue_B"], inplace=True)
movies_df.drop(columns=movies_df.columns[movies_df.isnull().mean() > 0.5].tolist(), inplace=True)  # drop >50% null

col_order = ["title", "year", "director", "rating", "box_office_usd"]
movies_df = movies_df[col_order]
```

---

## 8. Loading Data

### Reading CSV — all options explained

```python
import pandas as pd

# Minimal
df = pd.read_csv("data/orders.csv")

# Full options
df = pd.read_csv(
    "data/orders.csv",
    sep=",",                    # delimiter: "," (default), "\t" (TSV), "|" (pipe)
    header=0,                   # row number for column names (0=first row, None=no header)
    names=["id","date","amt"],  # supply column names manually (use with header=None)
    index_col=None,             # column to use as row index
    usecols=["order_id", "customer_id", "total"],  # load only these columns (faster!)
    nrows=1000,                 # read only first 1000 rows
    skiprows=5,                 # skip first 5 rows (useful for files with metadata at top)
    skipfooter=2,               # skip last 2 rows
    na_values=["NA","N/A","null","NULL","","n/a","nan"],  # treat these as NaN
    keep_default_na=True,       # also treat pandas defaults as NaN
    parse_dates=["order_date", "delivery_date"],    # auto-parse as datetime
    infer_datetime_format=True, # speed up date parsing
    dtype={
        "customer_id": str,     # prevent int IDs losing leading zeros or becoming floats
        "zip_code":    str,     # zip codes like "01234" — keep as string
        "amount":      float,
    },
    encoding="utf-8",           # or "latin-1" / "cp1252" for European files
    low_memory=False,           # avoids DtypeWarning on mixed-type columns
    thousands=",",              # "1,234" → 1234.0
    decimal=".",                # "1.5" — use "," for European locale files
    comment="#",                # skip lines starting with #
    chunksize=50000,            # return an iterable of chunks (for very large files)
)

# Reading in chunks (large files that don't fit in memory)
chunks = []
for chunk in pd.read_csv("big_file.csv", chunksize=50_000):
    # process each chunk
    chunk_result = chunk.groupby("category")["revenue"].sum()
    chunks.append(chunk_result)
final = pd.concat(chunks).groupby(level=0).sum()

# Read multiple CSVs from a folder and stack them
from pathlib import Path
all_dfs = []
for f in Path("data/monthly").glob("*.csv"):
    chunk = pd.read_csv(f)
    chunk["source_file"] = f.name    # track which file each row came from
    all_dfs.append(chunk)
combined = pd.concat(all_dfs, ignore_index=True)
```

---

### Reading other formats

```python
# Excel
df = pd.read_excel("report.xlsx")                          # first sheet
df = pd.read_excel("report.xlsx", sheet_name="Revenue")   # by name
df = pd.read_excel("report.xlsx", sheet_name=2)           # by index (0-based)
df = pd.read_excel("report.xlsx", sheet_name=None)        # all sheets → dict

# JSON
df = pd.read_json("data.json")
df = pd.read_json("data.json", orient="records")           # list of row dicts
df = pd.read_json("data.json", orient="columns")           # dict of column arrays
df = pd.read_json("data.json", lines=True)                 # JSONL (one JSON per line)

# Parquet — fast columnar format for large data
# pip install pyarrow
df = pd.read_parquet("data.parquet")
df = pd.read_parquet("data.parquet", columns=["id","date","revenue"])  # column pruning

# Clipboard — paste data directly from Excel/browser
df = pd.read_clipboard()

# From URL
df = pd.read_csv("https://raw.githubusercontent.com/.../data.csv")

# HTML tables — scrape tables from a webpage
dfs = pd.read_html("https://en.wikipedia.org/wiki/List_of_highest-grossing_films")
df  = dfs[0]    # first table on the page

# SQL (covered in Chapter 21)
import sqlite3
conn = sqlite3.connect("orders.db")
df   = pd.read_sql("SELECT * FROM orders WHERE status = 'delivered'", conn)
```

---

### Saving data

```python
# CSV — most universal
df.to_csv("output/cleaned.csv", index=False)     # index=False → don't write row numbers
df.to_csv("output/cleaned.csv", index=False, encoding="utf-8-sig")  # utf-8-sig for Excel compatibility

# Excel
df.to_excel("output/report.xlsx", sheet_name="Orders", index=False)

# Multiple sheets
with pd.ExcelWriter("output/multi_sheet.xlsx", engine="openpyxl") as writer:
    df_orders.to_excel(writer,  sheet_name="Orders",   index=False)
    df_sellers.to_excel(writer, sheet_name="Sellers",  index=False)
    df_summary.to_excel(writer, sheet_name="Summary",  index=False)

# Parquet — best for large data / pipelines
df.to_parquet("output/orders.parquet", index=False, compression="snappy")

# JSON
df.to_json("output/data.json", orient="records", indent=2)
```

---

## 9. Data Cleaning

### The cleaning checklist — always follow this order

```
1. Understand the data  → shape, dtypes, head(), info()
2. Null audit           → isnull().sum(), null %
3. Duplicate check      → duplicated().sum()
4. Fix data types       → astype(), to_datetime(), to_numeric()
5. Standardise strings  → strip, case, replace
6. Handle nulls         → drop, fill, impute
7. Handle outliers      → flag, cap, or drop
8. Validate ranges      → business rule checks
```

---

### Null handling — complete guide

```python
# ── AUDIT ─────────────────────────────────────────────────
df.isnull().sum()                         # null count per column
df.isnull().mean().mul(100).round(1)      # null % per column (MUCH more useful)
df.isnull().any(axis=1).sum()             # rows with at least one null
df.notnull().all(axis=1).sum()            # rows with zero nulls

# Show columns with > 5% nulls
null_pct = df.isnull().mean() * 100
null_pct[null_pct > 5].sort_values(ascending=False)

# Null heatmap (visual audit)
import seaborn as sns, matplotlib.pyplot as plt
plt.figure(figsize=(12, 5))
sns.heatmap(df.isnull(), cbar=False, yticklabels=False, cmap="viridis")
plt.title("Null Value Heatmap")
plt.tight_layout()
plt.show()

# ── DROP ──────────────────────────────────────────────────
df.dropna()                                # drop rows with ANY null  ← careful, aggressive
df.dropna(how="all")                       # drop rows where ALL values are null
df.dropna(subset=["customer_id", "order_date"])  # only care about these columns
df.dropna(axis=1)                          # drop COLUMNS with any null
df.dropna(axis=1, thresh=int(len(df)*0.5)) # drop columns that are >50% null

# ── FILL ──────────────────────────────────────────────────
# Fill with a constant
df["rating"].fillna(0, inplace=True)
df["language"].fillna("Unknown", inplace=True)
df["description"].fillna("No description", inplace=True)

# Fill with computed statistics
df["rating"].fillna(df["rating"].mean(),   inplace=True)  # mean imputation
df["rating"].fillna(df["rating"].median(), inplace=True)  # median (better for skewed data)
df["status"].fillna(df["status"].mode()[0], inplace=True) # mode for categorical

# Fill by group mean — more accurate imputation
df["rating"] = df.groupby("category")["rating"].transform(
    lambda x: x.fillna(x.mean())
)

# Forward / backward fill (for time-series)
df["price"].fillna(method="ffill", inplace=True)   # carry last known value forward
df["price"].fillna(method="bfill", inplace=True)   # fill backwards from next known value

# Fill different columns with different values
df.fillna({"rating": 0.0, "language": "Unknown", "budget": df["budget"].median()}, inplace=True)
```

---

### Duplicate handling

```python
# ── AUDIT ─────────────────────────────────────────────────
df.duplicated().sum()                            # count exact duplicate rows
df.duplicated(subset=["order_id"]).sum()         # duplicate keys
df.duplicated(subset=["customer_id", "date"]).sum()  # composite key duplicates

# Show duplicates — both copies
df[df.duplicated(keep=False)]
# Show only the "later" duplicates (first occurrence is kept)
df[df.duplicated(keep="first")]

# ── DROP ──────────────────────────────────────────────────
df.drop_duplicates(inplace=True)                            # exact row duplicates
df.drop_duplicates(subset=["order_id"], keep="first", inplace=True)  # keep first by order_id
df.drop_duplicates(subset=["order_id"], keep="last",  inplace=True)  # keep last

# Keep the record with the latest date (when there are multiple per key)
df_sorted = df.sort_values("updated_at", ascending=False)
df_deduped = df_sorted.drop_duplicates(subset=["customer_id"], keep="first")
```

---

### Fixing data types

```python
# ── CHECK ─────────────────────────────────────────────────
df.dtypes            # column → dtype mapping
df.dtypes.value_counts()  # how many of each type

# ── CONVERT ───────────────────────────────────────────────
df["year"]        = df["year"].astype(int)
df["rating"]      = df["rating"].astype(float)
df["customer_id"] = df["customer_id"].astype(str)
df["is_active"]   = df["is_active"].astype(bool)
df["status"]      = df["status"].astype("category")   # memory-efficient for low-cardinality

# Safe numeric conversion (errors → NaN instead of crashing)
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")
# "coerce" converts un-parseable values to NaN
# "ignore" leaves them as-is
# "raise"  throws an error (default)

# Datetime conversion (Chapter 23 has full reference)
df["order_date"] = pd.to_datetime(df["order_date"])
df["order_date"] = pd.to_datetime(df["order_date"], format="%d/%m/%Y")   # explicit format
df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")     # bad dates → NaT

# Categorical (saves significant memory for repeated string values)
df["category"] = df["category"].astype("category")
df["category"].cat.categories   # see all valid categories
df["category"].cat.codes        # integer code for each category
```

---

### Standardising strings

```python
# ── CLEAN COLUMN NAMES FIRST ──────────────────────────────
df.columns = (
    df.columns
    .str.lower()
    .str.strip()
    .str.replace(" ", "_")
    .str.replace(r"[^\w]", "_", regex=True)   # replace non-word chars
)

# ── CLEAN STRING COLUMNS ──────────────────────────────────
df["city"]   = df["city"].str.strip().str.title()
df["email"]  = df["email"].str.lower().str.strip()
df["status"] = df["status"].str.upper().str.strip()

# ── VALUE MAPPING ─────────────────────────────────────────
# Fix known typos or abbreviations
status_map = {
    "DELIEVRED": "DELIVERED",
    "SHIPED":    "SHIPPED",
    "C":         "CANCELLED",
    "D":         "DELIVERED",
}
df["status"] = df["status"].replace(status_map)

# Replace across the whole DataFrame
df.replace({"N/A": None, "null": None, "-": None, "": None}, inplace=True)

# ── STRING OPERATIONS VIA .str ACCESSOR ───────────────────
# All Python string methods work through .str
df["email"].str.lower()
df["name"].str.contains("Nolan", na=False)        # na=False → NaN → False
df["title"].str.startswith("The")
df["code"].str.len()
df["description"].str.strip()
df["name"].str.split(" ")                          # → list per cell
df["name"].str.split(" ", expand=True)             # → DataFrame (one column per part)
df["name"].str.split(" ").str[0]                   # first word only
df["description"].str.replace(r"\s+", " ", regex=True)  # collapse multiple spaces
df["code"].str.extract(r"(\d+)")                   # first digit sequence (regex capture group)
df["code"].str.extractall(r"(\d+)")                # all digit sequences
df["ref"].str.contains(r"^ORD-\d{5}$", regex=True) # validate format

# ── COMMON CLEANING PATTERNS ──────────────────────────────
# Remove non-numeric characters from phone
df["phone"] = df["phone"].str.replace(r"[^\d]", "", regex=True)

# Extract year from strings like "2023-Q1 report"
df["year"] = df["report_name"].str.extract(r"(\d{4})").astype(float)

# Normalise product codes (strip spaces, uppercase, remove dots)
df["sku"] = df["sku"].str.upper().str.replace(r"[\s\.]", "", regex=True)
```

---

### Outlier detection and treatment

```python
import numpy as np

col = "payment_value"

# ── DETECT ────────────────────────────────────────────────
# Method 1: Z-score (assumes roughly normal distribution)
z = (df[col] - df[col].mean()) / df[col].std()
outliers_z = df[z.abs() > 3]
print(f"Z-score outliers (|z|>3): {len(outliers_z)} rows ({len(outliers_z)/len(df):.1%})")

# Method 2: IQR (more robust for skewed distributions — preferred for financial data)
Q1  = df[col].quantile(0.25)
Q3  = df[col].quantile(0.75)
IQR = Q3 - Q1
lower_fence = Q1 - 1.5 * IQR
upper_fence = Q3 + 1.5 * IQR
outliers_iqr = df[(df[col] < lower_fence) | (df[col] > upper_fence)]
print(f"IQR outliers: {len(outliers_iqr)} rows")
print(f"Normal range: [{lower_fence:.2f}, {upper_fence:.2f}]")

# Method 3: Percentile-based
low_pct  = df[col].quantile(0.01)
high_pct = df[col].quantile(0.99)
outliers_pct = df[(df[col] < low_pct) | (df[col] > high_pct)]

# ── TREAT ─────────────────────────────────────────────────
# Option A: Flag and investigate (safest first step)
df["is_outlier"] = ((df[col] < lower_fence) | (df[col] > upper_fence)).astype(int)

# Option B: Cap (Winsorise) — clip to fence values
df[col + "_capped"] = df[col].clip(lower=lower_fence, upper=upper_fence)

# Option C: Remove (aggressive — only when clearly errors)
df_clean = df[(df[col] >= lower_fence) & (df[col] <= upper_fence)].copy()

# Option D: Log-transform (for right-skewed distributions like revenue)
df["log_amount"] = np.log1p(df[col])   # log1p = log(1+x), handles zeros safely
```

---

## 10. Filtering, Sorting, Selecting

### Boolean filtering — the complete pattern library

```python
# ── SINGLE CONDITION ──────────────────────────────────────
high_rated = movies_df[movies_df["rating"] > 8.5]
post_2000  = movies_df[movies_df["year"]   >= 2000]
nolan      = movies_df[movies_df["director"] == "Nolan"]

# ── COMPOUND CONDITIONS ───────────────────────────────────
# & = AND, | = OR, ~ = NOT — MUST use parentheses around each condition
modern_great = movies_df[(movies_df["year"] > 2000) & (movies_df["rating"] > 8.0)]
nolan_or_bong= movies_df[(movies_df["director"] == "Nolan") | (movies_df["director"] == "Bong")]
not_nolan    = movies_df[~(movies_df["director"] == "Nolan")]

# ── RANGE AND LIST FILTERS ────────────────────────────────
mid_rated   = movies_df[movies_df["rating"].between(7.5, 8.5)]   # inclusive
selected    = movies_df[movies_df["director"].isin(["Nolan", "Bong", "Gerwig"])]
excluded    = movies_df[~movies_df["director"].isin(["Nolan"])]

# ── STRING FILTERS ────────────────────────────────────────
has_dark    = movies_df[movies_df["title"].str.contains("Dark", na=False)]
starts_the  = movies_df[movies_df["title"].str.startswith("The")]
english     = movies_df[movies_df["language"].str.lower() == "english"]

# ── NULL FILTERS ──────────────────────────────────────────
has_rating  = movies_df[movies_df["rating"].notna()]
no_rating   = movies_df[movies_df["rating"].isna()]

# ── query() — SQL-like syntax, very readable ──────────────
# SQL: SELECT * FROM movies WHERE rating > 8 AND year > 2000
movies_df.query("rating > 8 and year > 2000")
movies_df.query("director in ['Nolan', 'Bong']")
movies_df.query("rating > @threshold")    # @ references a Python variable

threshold = 8.0
movies_df.query("rating > @threshold and year >= 2000")

# ── MULTIPLE FILTERS — CHAINING (readable pattern) ────────
result = (
    movies_df
    .query("year >= 2000")
    .query("rating > 7.5")
    .query("director != 'Unknown'")
    [["title", "year", "rating"]]   # select columns
    .sort_values("rating", ascending=False)
    .head(10)
)
```

---

### Sorting

```python
# Single column
df.sort_values("rating", ascending=False)   # highest first
df.sort_values("rating", ascending=True)    # lowest first

# Multiple columns (SQL: ORDER BY year DESC, rating DESC)
df.sort_values(["year", "rating"], ascending=[False, False])

# NaN positioning
df.sort_values("rating", na_position="last")    # NaN at end
df.sort_values("rating", na_position="first")   # NaN at start

# Sort by computed value
df.sort_values(by="box_office", key=lambda col: col.abs())   # sort by absolute value

# Stable sort — preserves original order for ties (default)
df.sort_values("rating", kind="mergesort")

# Sort index
df.sort_index(ascending=True)
df.sort_index(ascending=False)
```

---

## 11. GroupBy and Aggregation

### The split-apply-combine pattern

GroupBy works in three stages:
1. **Split** — divide data into groups based on one or more columns
2. **Apply** — run a function on each group independently
3. **Combine** — collect the results back into a single DataFrame

This is the exact equivalent of SQL's `GROUP BY`.

```python
# SQL:    SELECT director, COUNT(*), AVG(rating) FROM movies GROUP BY director
# pandas:
movies_df.groupby("director").agg(
    film_count = ("title",  "count"),
    avg_rating = ("rating", "mean")
)
```

---

### All aggregation patterns

```python
# ── SINGLE FUNCTION ───────────────────────────────────────
df.groupby("director")["rating"].mean()
df.groupby("director")["rating"].sum()
df.groupby("director")["rating"].count()
df.groupby("director")["rating"].min()
df.groupby("director")["rating"].max()
df.groupby("director")["rating"].std()
df.groupby("director")["rating"].median()
df.groupby("director")["rating"].first()   # first value in group
df.groupby("director")["rating"].last()    # last value in group
df.groupby("director")["rating"].nunique() # distinct values

# ── NAMED AGGREGATION (cleanest syntax) ───────────────────
result = df.groupby("director").agg(
    total_films      = ("title",      "count"),
    avg_rating       = ("rating",     "mean"),
    max_rating       = ("rating",     "max"),
    min_rating       = ("rating",     "min"),
    rating_spread    = ("rating",     lambda x: x.max() - x.min()),
    total_box_office = ("box_office", "sum"),
    median_box_off   = ("box_office", "median"),
).round(2)

# ── MULTIPLE FUNCTIONS ON MULTIPLE COLUMNS ────────────────
result = df.groupby("director").agg({
    "title":      ["count"],
    "rating":     ["mean", "max", "std"],
    "box_office": ["sum", "mean"]
})
# Result has multi-level columns: ('rating', 'mean'), ('rating', 'max'), etc.
result.columns = ["_".join(c) for c in result.columns]  # flatten to: 'rating_mean', etc.

# ── MULTIPLE GROUPBY COLUMNS ──────────────────────────────
# SQL: GROUP BY year, language
df.groupby(["year", "language"])["revenue"].sum()
df.groupby(["year", "language"]).agg(
    orders  = ("order_id", "count"),
    revenue = ("amount",   "sum")
).reset_index()    # converts MultiIndex to regular columns

# ── GROUP BY WITH DATE EXTRACTION ─────────────────────────
df["month"] = df["order_date"].dt.to_period("M")
monthly = df.groupby("month").agg(orders=("order_id","count"), revenue=("amount","sum"))

# ── FILTER GROUPS (HAVING equivalent) ────────────────────
# SQL: HAVING COUNT(*) > 5
active_customers = df.groupby("customer_id").filter(lambda g: len(g) > 5)
high_value_cats  = df.groupby("category").filter(lambda g: g["revenue"].sum() > 10_000)

# ── TRANSFORM — window function equivalent ────────────────
# Returns same-length result aligned with original DataFrame
df["director_avg"]    = df.groupby("director")["rating"].transform("mean")
df["director_rank"]   = df.groupby("director")["rating"].transform("rank")
df["director_pct"]    = df.groupby("director")["box_office"].transform(lambda x: x / x.sum())
df["cum_revenue"]     = df.groupby("director")["box_office"].transform("cumsum")
df["diff_from_avg"]   = df["rating"] - df["director_avg"]   # deviation from group mean

# ── APPLY — most flexible, runs Python function per group ──
def top_film(group):
    """Return the top-rated film for each director."""
    return group.nlargest(1, "rating")

top_per_dir = df.groupby("director").apply(top_film).reset_index(drop=True)

def zscore(group, col):
    """Z-score within each group."""
    return (group[col] - group[col].mean()) / group[col].std()

df["rating_zscore"] = df.groupby("director").apply(
    lambda g: zscore(g, "rating")
).reset_index(level=0, drop=True)
```

---

### Pivot tables and crosstabs

```python
# ── PIVOT TABLE — the analyst's Swiss army knife ──────────
# SQL equivalent: SELECT year, director, SUM(box_office) ... GROUP BY year, director  (then pivot)

result = pd.pivot_table(
    movies_df,
    values="box_office",          # cell values
    index="director",             # rows
    columns="year",               # columns
    aggfunc="sum",                # how to aggregate
    fill_value=0,                 # replace NaN with 0
    margins=True,                 # add row/column totals  ("All" label)
    margins_name="Total",
    dropna=True,                  # exclude columns where all values are NaN
)

# Multiple aggregation functions
result = pd.pivot_table(
    df,
    values=["revenue", "rating"],
    index="category",
    aggfunc={"revenue": ["sum", "mean"], "rating": "mean"}
)

# ── CROSSTAB — frequency count between two categorical columns ──
# Count
pd.crosstab(df["gender"], df["department"])

# Proportions (as %)
pd.crosstab(df["gender"], df["department"], normalize=True).mul(100).round(1)
# normalize="index" → row proportions; normalize="columns" → column proportions

# With values — like pivot_table but simpler syntax for aggregation
pd.crosstab(
    df["category"],
    df["status"],
    values=df["revenue"],
    aggfunc="sum"
).fillna(0)

# ── VALUE COUNTS — frequency tables ──────────────────────
df["status"].value_counts()                     # most common first
df["status"].value_counts(normalize=True)       # proportions (0.0 to 1.0)
df["status"].value_counts(dropna=False)         # include NaN in count
df["status"].value_counts(normalize=True).mul(100).round(1)  # as percentage

# ── BINNING ───────────────────────────────────────────────
# pd.cut — fixed-width bins
df["age_band"] = pd.cut(
    df["age"],
    bins=[0, 25, 40, 60, 100],
    labels=["<25", "25-40", "40-60", "60+"],
    right=True,        # (left, right]  — is right endpoint included?
    include_lowest=True
)

# pd.qcut — equal-frequency bins (like NTILE in SQL)
df["revenue_quartile"] = pd.qcut(df["revenue"], q=4, labels=["Q1","Q2","Q3","Q4"])
df["rfm_score"]        = pd.qcut(df["revenue"].rank(method="first"), q=5, labels=[1,2,3,4,5])
# rank(method="first") handles duplicate values — assigns unique ranks
# duplicates="drop" in qcut removes duplicate bin edges (for highly skewed data)
```

---

## 12. Merging and Joining

### merge() — SQL JOIN in pandas

```python
# Sample data
orders    = pd.DataFrame({"order_id": [1,2,3,4], "customer_id": [10,11,12,10], "total": [100,200,150,80]})
customers = pd.DataFrame({"customer_id": [10,11,99], "name": ["Alice","Bob","Charlie"], "city": ["Delhi","Mumbai","Chennai"]})

# ── INNER JOIN ────────────────────────────────────────────
# SQL: SELECT * FROM orders INNER JOIN customers ON orders.customer_id = customers.customer_id
result = pd.merge(orders, customers, on="customer_id", how="inner")
# 3 rows: customer_id 12 (no match) and customer_id 99 (no orders) both excluded

# ── LEFT JOIN ─────────────────────────────────────────────
result = pd.merge(orders, customers, on="customer_id", how="left")
# 4 rows: customer_id 12 included with NaN for name and city

# ── RIGHT JOIN ────────────────────────────────────────────
result = pd.merge(orders, customers, on="customer_id", how="right")
# 4 rows: customer_id 99 included with NaN for order cols

# ── FULL OUTER JOIN ───────────────────────────────────────
result = pd.merge(orders, customers, on="customer_id", how="outer")
# 5 rows: includes both unmatched orders AND unmatched customers

# ── DIFFERENT COLUMN NAMES ────────────────────────────────
pd.merge(orders, customers, left_on="customer_id", right_on="cust_id")

# ── MULTIPLE JOIN KEYS ────────────────────────────────────
pd.merge(orders, products, on=["product_id", "product_version"])

# ── SUFFIX HANDLING ───────────────────────────────────────
# Both tables have a 'name' column — add suffixes to distinguish
pd.merge(df1, df2, on="id", suffixes=("_left", "_right"))

# ── ANTI-JOIN: rows with NO match ─────────────────────────
# SQL: WHERE NOT EXISTS (SELECT 1 FROM customers WHERE ...)
merged = pd.merge(orders, customers, on="customer_id", how="left", indicator=True)
unmatched = merged[merged["_merge"] == "left_only"].drop("_merge", axis=1)

# ── VERIFY THE JOIN ───────────────────────────────────────
# Always check for row multiplication after a merge
print(f"Left: {len(orders)}, Right: {len(customers)}, Result: {len(result)}")
# If result >> left, you have a 1:N relationship you didn't account for

# If duplicates in join key cause explosion:
customers_deduped = customers.drop_duplicates(subset=["customer_id"])
result = pd.merge(orders, customers_deduped, on="customer_id", how="left")
```

---

### concat() — UNION ALL in pandas

```python
# Stack rows (UNION ALL)
combined = pd.concat([df_2022, df_2023], ignore_index=True)
# ignore_index=True → resets index to 0,1,2... (avoid duplicate index values)

# Stack with source tag
df_2022["year"] = 2022
df_2023["year"] = 2023
combined = pd.concat([df_2022, df_2023], ignore_index=True)

# Using keys (creates MultiIndex — use when you need to identify source)
combined = pd.concat([df_2022, df_2023], keys=["2022", "2023"])
combined.loc["2022"]   # access 2022 rows

# Stack columns side by side (axis=1)
wider = pd.concat([df_sales, df_returns], axis=1)

# Concat only if both have data (handle empties)
dfs = [df for df in [df_2021, df_2022, df_2023] if not df.empty]
combined = pd.concat(dfs, ignore_index=True) if dfs else pd.DataFrame()
```

---

## 13. Reshaping

### melt() — wide to long (unpivot)

```python
# Wide format (one column per month — common in Excel reports)
wide = pd.DataFrame({
    "product": ["Laptop", "Phone", "Tablet"],
    "jan_rev": [50000, 30000, 20000],
    "feb_rev": [55000, 32000, 18000],
    "mar_rev": [48000, 35000, 22000],
})

# Melt to long format (better for pandas analysis)
long = pd.melt(
    wide,
    id_vars=["product"],          # columns to keep as-is
    value_vars=["jan_rev", "feb_rev", "mar_rev"],  # columns to unpivot
    var_name="month",             # new column for the old column names
    value_name="revenue"          # new column for the values
)
#   product    month  revenue
# 0  Laptop  jan_rev    50000
# 1   Phone  jan_rev    30000
# ...

# Clean up the month column
long["month"] = long["month"].str.replace("_rev", "")
```

---

### pivot() — long to wide

```python
# Long format back to wide
pivoted = long.pivot(
    index="product",      # unique row identifier
    columns="month",      # values become column names
    values="revenue"      # what fills the cells
)

# Flatten column index
pivoted.columns.name = None
pivoted = pivoted.reset_index()
```

---

### stack() and unstack()

```python
# stack(): move column level to row level (wide → long for MultiIndex)
df_multi = pd.DataFrame(
    np.random.rand(4, 3),
    index=["A", "B", "C", "D"],
    columns=["X", "Y", "Z"]
)
stacked   = df_multi.stack()    # Series with MultiIndex (row, col)
unstacked = stacked.unstack()   # back to original

# Useful with pivot_table results
pivot = df.groupby(["year", "category"])["revenue"].sum().unstack("category").fillna(0)
# Rows = year, columns = category — classic time-series analysis view
```

---

### explode() — expand lists in cells

```python
# Common with JSON/API data where one cell contains a list
df = pd.DataFrame({
    "title":  ["Inception", "Parasite"],
    "genres": [["Sci-Fi", "Thriller"], ["Drama", "Thriller", "Crime"]]
})

exploded = df.explode("genres")
#       title    genres
# 0  Inception    Sci-Fi
# 0  Inception  Thriller
# 1   Parasite     Drama
# 1   Parasite  Thriller
# 1   Parasite     Crime

# Now you can use value_counts, groupby, etc. on individual genre values
exploded["genres"].value_counts()
```

---

## 14. Window Functions and Time Series

### Rolling and expanding windows

```python
# Rolling: aggregate over a sliding window of N rows
df["rolling_7d"]   = df["daily_revenue"].rolling(7).mean()   # 7-day moving avg
df["rolling_30d"]  = df["daily_revenue"].rolling(30).mean()  # 30-day moving avg
df["rolling_7_std"]= df["daily_revenue"].rolling(7).std()    # 7-day rolling stddev
df["rolling_7_min"]= df["daily_revenue"].rolling(7).min()    # 7-day rolling min
df["rolling_7_max"]= df["daily_revenue"].rolling(7).max()    # 7-day rolling max

# min_periods: how many values needed before producing a result
df["rolling_7d"] = df["daily_revenue"].rolling(7, min_periods=1).mean()
# min_periods=1 → start producing results even with < 7 rows (for start of series)

# Expanding: cumulative from first row to current row
df["cumulative_revenue"] = df["daily_revenue"].expanding().sum()    # running total
df["running_mean"]       = df["daily_revenue"].expanding().mean()   # running average
df["cumulative_max"]     = df["daily_revenue"].expanding().max()    # running max

# Shift — LAG/LEAD equivalent
df["prev_day_rev"]   = df["daily_revenue"].shift(1)      # LAG(1)  — yesterday's value
df["next_day_rev"]   = df["daily_revenue"].shift(-1)     # LEAD(1) — tomorrow's value
df["prev_week_rev"]  = df["daily_revenue"].shift(7)      # same day last week
df["prev_month_rev"] = df["daily_revenue"].shift(30)     # same day last month

# Period-over-period calculations
df["wow_abs"]  = df["daily_revenue"] - df["daily_revenue"].shift(7)     # week-over-week absolute
df["wow_pct"]  = df["daily_revenue"].pct_change(7) * 100                # week-over-week %
df["mom_pct"]  = df["daily_revenue"].pct_change(30) * 100               # month-over-month %
df["yoy_pct"]  = df["daily_revenue"].pct_change(365) * 100              # year-over-year %

# Cumulative sum (running total)
df["running_total"] = df["daily_revenue"].cumsum()

# Ranking within groups (like ROW_NUMBER/RANK in SQL)
df["rank_by_rev"]     = df["daily_revenue"].rank(ascending=False)
df["rank_within_cat"] = df.groupby("category")["revenue"].rank(
    method="dense", ascending=False   # dense = no gaps for ties
)
```

---

### Datetime handling in pandas

```python
# Parse
df["order_date"] = pd.to_datetime(df["order_date"])
df["order_date"] = pd.to_datetime(df["order_date"], format="%d/%m/%Y")  # specific format
df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")    # bad dates → NaT

# Extract components via .dt accessor
df["year"]        = df["order_date"].dt.year
df["month"]       = df["order_date"].dt.month          # 1–12
df["month_name"]  = df["order_date"].dt.month_name()   # "January"
df["day"]         = df["order_date"].dt.day
df["dayofweek"]   = df["order_date"].dt.dayofweek      # 0=Monday, 6=Sunday
df["day_name"]    = df["order_date"].dt.day_name()     # "Monday"
df["quarter"]     = df["order_date"].dt.quarter
df["week"]        = df["order_date"].dt.isocalendar().week
df["hour"]        = df["order_date"].dt.hour
df["is_weekend"]  = df["order_date"].dt.dayofweek >= 5  # True for Sat/Sun

# Period truncation
df["month_start"] = df["order_date"].dt.to_period("M").dt.to_timestamp()
df["year_start"]  = df["order_date"].dt.to_period("Y").dt.to_timestamp()
df["week_start"]  = df["order_date"] - pd.to_timedelta(df["order_date"].dt.dayofweek, unit="D")
df["quarter_str"] = df["order_date"].dt.year.astype(str) + "-Q" + df["order_date"].dt.quarter.astype(str)

# Duration
df["days_to_deliver"]  = (df["delivered_date"] - df["order_date"]).dt.days
df["hours_to_deliver"] = (df["delivered_date"] - df["order_date"]).dt.total_seconds() / 3600

# Date offsets
df["due_date"]   = df["order_date"] + pd.DateOffset(days=7)
df["next_month"] = df["order_date"] + pd.DateOffset(months=1)

# Resample (requires datetime index)
df.set_index("order_date", inplace=True)
monthly = df["revenue"].resample("ME").sum()       # monthly total
weekly  = df["revenue"].resample("W").mean()       # weekly mean
daily   = df["revenue"].resample("D").sum().fillna(0)  # daily (fill missing days with 0)

# Generate a date range
date_range = pd.date_range("2023-01-01", "2023-12-31", freq="D")   # daily
biz_days   = pd.date_range("2023-01-01", "2023-12-31", freq="B")   # business days only
months     = pd.date_range("2023-01-01", periods=12, freq="ME")    # 12 month-ends
```

---

## 15. Performance and Memory Optimisation

### Why it matters

A 1M-row DataFrame with default dtypes might use 500MB. The same data, optimised, might use 50MB — and run 5× faster.

```python
# ── DIAGNOSE ──────────────────────────────────────────────
df.info(memory_usage="deep")         # actual memory usage per column
df.memory_usage(deep=True).sum()     # total bytes
df.memory_usage(deep=True) / 1024**2 # in MB

# ── OPTIMISE NUMERIC TYPES ────────────────────────────────
# Downcast integers and floats to smallest type that fits
import numpy as np

# Integer columns
for col in df.select_dtypes(include="int64").columns:
    df[col] = pd.to_numeric(df[col], downcast="integer")  # tries int8/int16/int32/int64

# Float columns
for col in df.select_dtypes(include="float64").columns:
    df[col] = pd.to_numeric(df[col], downcast="float")    # tries float32/float64

# Manual downcast
df["year"]  = df["year"].astype("int16")    # 2 bytes vs 8 bytes
df["score"] = df["score"].astype("float32") # 4 bytes vs 8 bytes

# ── CATEGORICAL DTYPE — biggest win for string columns ────
# If a column has low cardinality (few unique values repeated many times):
# "status" column with 5 unique values across 1M rows → MASSIVE savings

df["status"]   = df["status"].astype("category")    # stores int codes internally
df["category"] = df["category"].astype("category")
df["country"]  = df["country"].astype("category")

# Demonstrate savings
original_mem = df["status"].memory_usage(deep=True)
df["status"] = df["status"].astype("category")
new_mem = df["status"].memory_usage(deep=True)
print(f"Savings: {(1 - new_mem/original_mem):.0%}")  # often 80-95% reduction

# ── CHUNKSIZE FOR LARGE FILES ─────────────────────────────
# Process files larger than RAM
result_parts = []
for chunk in pd.read_csv("huge_file.csv", chunksize=100_000):
    part = chunk.groupby("category")["revenue"].sum()
    result_parts.append(part)
final_result = pd.concat(result_parts).groupby(level=0).sum()

# ── usecols — don't load what you don't need ──────────────
# If the file has 50 columns but you only need 5:
df = pd.read_csv("orders.csv", usecols=["order_id", "date", "customer_id", "amount", "status"])

# ── eval() — faster for arithmetic on large DataFrames ────
# For 100k+ rows, eval() is faster than direct pandas arithmetic
df["margin_pct"] = df.eval("(revenue - cost) / revenue * 100")
mask = df.eval("revenue > 1000 and margin_pct > 20")
df_filtered = df[mask]
```

---
---

## 16. NumPy

### What is NumPy and why use it?

NumPy (Numerical Python) provides the `ndarray` — an n-dimensional array that:
- Stores all elements as the **same type** (unlike Python lists which allow mixed types)
- Is stored in a **contiguous block of memory** (fast access)
- Supports **vectorised operations** — operations apply to every element simultaneously, without writing loops

**Why does this matter?**
```python
# Pure Python — loop needed, slow
import time
n = 1_000_000
data = list(range(n))

start = time.time()
result = [x * 2 for x in data]
print(f"Python list: {time.time()-start:.3f}s")

# NumPy — vectorised, ~10-100x faster
import numpy as np
arr = np.arange(n)

start = time.time()
result = arr * 2        # no loop — C code under the hood
print(f"NumPy array: {time.time()-start:.3f}s")
```

```python
pip install numpy
import numpy as np
```

---

### Creating arrays — all patterns

```python
# From Python list
arr1d = np.array([8.8, 8.5, 8.9, 7.0, 9.0])           # 1D
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])    # 2D (3×3 matrix)
arr3d = np.array([[[1,2],[3,4]], [[5,6],[7,8]]])        # 3D

# Specify dtype explicitly
arr = np.array([1, 2, 3], dtype=np.float32)
arr = np.array([1.1, 2.2], dtype=np.int32)  # truncates to int: [1, 2]

# ── UTILITY CONSTRUCTORS ──────────────────────────────────
np.zeros(5)                   # [0. 0. 0. 0. 0.]
np.zeros((3, 4))              # 3×4 matrix of zeros
np.ones(5)                    # [1. 1. 1. 1. 1.]
np.ones((2, 3), dtype=int)    # 2×3 integer matrix of ones
np.full(5, 8.5)               # [8.5 8.5 8.5 8.5 8.5]
np.full((3, 3), fill_value=7) # 3×3 matrix of 7s
np.eye(3)                     # 3×3 identity matrix (1s on diagonal)
np.empty((2, 3))              # uninitialized (fast, values undefined)

# ── RANGES ────────────────────────────────────────────────
np.arange(10)                 # [0 1 2 3 4 5 6 7 8 9]  like range()
np.arange(1, 10, 2)           # [1 3 5 7 9]  (start, stop, step)
np.linspace(0, 1, 5)          # [0.   0.25 0.5  0.75 1.  ]  5 evenly spaced between 0 and 1
np.logspace(0, 3, 4)          # [1. 10. 100. 1000.]  logarithmically spaced

# ── RANDOM ────────────────────────────────────────────────
np.random.seed(42)            # set seed for reproducibility (important for analysis!)
np.random.rand(5)             # 5 uniform random floats [0, 1)
np.random.rand(3, 4)          # 3×4 array of random floats
np.random.randn(5)            # 5 values from standard normal N(0, 1)
np.random.randint(1, 100, size=10)  # 10 random integers 1-99
np.random.choice([1, 2, 3, 4, 5], size=10, replace=True)   # random sample with replacement
np.random.choice([1, 2, 3, 4, 5], size=3,  replace=False)  # without replacement
np.random.normal(loc=50, scale=10, size=1000)  # normal dist, mean=50, std=10
np.random.uniform(low=0, high=100, size=500)   # uniform distribution
```

---

### Array properties and reshaping

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.shape        # (2, 3)   — rows × columns
arr.ndim         # 2        — number of dimensions
arr.size         # 6        — total elements
arr.dtype        # int64    — element type
arr.itemsize     # 8        — bytes per element
arr.nbytes       # 48       — total bytes

# ── RESHAPE ───────────────────────────────────────────────
arr.reshape(3, 2)             # 2×3 → 3×2 (6 elements, new shape)
arr.reshape(6)                # flatten to 1D
arr.reshape(-1)               # same (-1 = infer size)
arr.reshape(2, -1)            # 2 rows, infer columns from total size
arr.flatten()                 # always returns a copy
arr.ravel()                   # returns view if possible (more efficient)

# ── TRANSPOSE ─────────────────────────────────────────────
arr.T                         # transpose (rows ↔ columns)
arr.T.shape                   # (3, 2)

# ── TYPE CONVERSION ───────────────────────────────────────
arr.astype(float)
arr.astype(np.float32)
arr.astype(str)
```

---

### Indexing and slicing — 2D arrays

```python
arr = np.array([[10, 20, 30],
                [40, 50, 60],
                [70, 80, 90]])

# ── ELEMENT ACCESS ─────────────────────────────────────────
arr[0, 0]          # 10   — row 0, col 0
arr[1, 2]          # 60   — row 1, col 2
arr[-1, -1]        # 90   — last row, last col

# ── ROW AND COLUMN SLICING ────────────────────────────────
arr[0]             # [10 20 30]  — entire first row
arr[:, 0]          # [10 40 70]  — entire first column
arr[:, -1]         # [30 60 90]  — last column

# ── SUB-ARRAYS ────────────────────────────────────────────
arr[0:2, 0:2]      # [[10 20]   — top-left 2×2 block
                   #  [40 50]]
arr[::2, ::2]      # [[10 30]   — every other row and column
                   #  [70 90]]

# ── BOOLEAN INDEXING ──────────────────────────────────────
mask = arr > 40
arr[mask]          # [50 60 70 80 90]  — flat array of matching elements
arr[arr > 40]      # same

# Modify values matching a condition
arr[arr < 30] = 0  # set all values < 30 to 0
```

---

### Vectorised operations — no loops needed

```python
ratings = np.array([8.8, 8.5, 8.9, 7.0, 9.0])

# ── ELEMENT-WISE ARITHMETIC ───────────────────────────────
ratings * 10           # [88.0 85.0 89.0 70.0 90.0]
ratings + 1            # [9.8 9.5 9.9 8.0 10.0]
ratings ** 2           # [77.44 72.25 79.21 49.0 81.0]
np.sqrt(ratings)       # square root of each element
np.log(ratings)        # natural log
np.log10(ratings)      # base-10 log
np.exp(ratings)        # e^x
np.abs([-1, -2, 3])   # absolute values: [1 2 3]
np.round(ratings, 1)   # round to 1 decimal

# ── COMPARISON → BOOLEAN ARRAY ────────────────────────────
ratings > 8.5          # [True False True False True]
ratings == 8.8         # [True False False False False]

# ── BOOLEAN OPERATIONS ────────────────────────────────────
(ratings > 8) & (ratings < 9)    # both conditions (element-wise AND)
(ratings < 7) | (ratings > 9)    # either condition (element-wise OR)
~(ratings > 8)                   # NOT

# ── AGGREGATE FUNCTIONS ───────────────────────────────────
ratings.sum()           # 42.2
ratings.mean()          # 8.44
ratings.std()           # 0.703...
ratings.var()           # variance
ratings.min()           # 7.0
ratings.max()           # 9.0
ratings.argmin()        # index of minimum value: 3
ratings.argmax()        # index of maximum value: 4
np.median(ratings)      # median: 8.8
np.percentile(ratings, [25, 50, 75])  # quartiles

# ── 2D AGGREGATES WITH axis ───────────────────────────────
m = np.array([[1, 2, 3], [4, 5, 6]])
m.sum()           # 21  — all elements
m.sum(axis=0)     # [5 7 9]  — column totals (collapse rows)
m.sum(axis=1)     # [6 15]   — row totals (collapse columns)
m.mean(axis=0)    # column means
m.max(axis=1)     # row maxima
```

---

### Broadcasting

Broadcasting is NumPy's mechanism for applying operations between arrays of different shapes. Understanding it lets you write vectorised code that would otherwise need nested loops.

**Rule:** NumPy compares shapes from right to left. Dimensions are compatible if they are equal OR one of them is 1.

```python
# Scalar × array: scalar is broadcast to all elements
arr = np.array([1, 2, 3, 4])
arr * 2               # [2 4 6 8]  — scalar 2 broadcast to shape (4,)

# 1D × 2D: the 1D array is broadcast across rows
prices  = np.array([100, 200, 300])        # shape (3,)
tax_rates = np.array([[0.1], [0.2]])       # shape (2, 1)
prices * tax_rates
# [[10.  20.  30. ]    ← row 0: prices × 0.1
#  [20.  40.  60. ]]   ← row 1: prices × 0.2

# Practical: subtract column means to centre data
data = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
col_means = data.mean(axis=0)   # [4. 5. 6.]  shape (3,)
centred   = data - col_means    # broadcast (3,) across (3,3)
# [[-3. -3. -3.], [0. 0. 0.], [3. 3. 3.]]

# Normalise each column (z-score)
col_std = data.std(axis=0)
normalised = (data - col_means) / col_std
```

---

### NumPy for data analysis

```python
# ── SORTING ───────────────────────────────────────────────
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])
np.sort(arr)                    # [1 1 2 3 4 5 6 9]  returns sorted copy
arr.sort()                      # in-place
np.argsort(arr)                 # [1 3 6 0 2 4 7 5]  indices that would sort arr

# ── CONDITIONAL ASSIGNMENT ────────────────────────────────
# np.where(condition, value_if_true, value_if_false)
ratings = np.array([8.8, 7.5, 9.0, 6.5, 8.2])
labels  = np.where(ratings >= 8, "Good", "Poor")
# ['Good' 'Poor' 'Good' 'Poor' 'Good']

# Multi-level conditions
bins    = np.select(
    [ratings >= 9, ratings >= 8, ratings >= 7],
    ["Masterpiece", "Great", "Good"],
    default="Average"
)

# ── STATISTICAL ANALYSIS ──────────────────────────────────
data = np.random.normal(loc=100, scale=15, size=1000)  # 1000 values from N(100, 15)

np.mean(data)          # ≈ 100
np.std(data)           # ≈ 15
np.median(data)        # ≈ 100
np.percentile(data, [10, 25, 50, 75, 90])  # percentiles
np.corrcoef(arr1, arr2)                    # correlation matrix

# Histogram of values (returns bin counts and edges — for plotting logic)
counts, edges = np.histogram(data, bins=20)

# ── BINNING (like pd.qcut / NTILE) ────────────────────────
percentile_boundaries = np.percentile(data, [0, 20, 40, 60, 80, 100])
bins = np.digitize(data, percentile_boundaries[1:-1])  # 1-5 quintile labels

# ── UNIQUE VALUES AND COUNTS ──────────────────────────────
arr = np.array([1, 2, 2, 3, 3, 3, 4])
np.unique(arr)                      # [1 2 3 4]
values, counts = np.unique(arr, return_counts=True)
# values=[1,2,3,4], counts=[1,2,3,1]

# ── SET OPERATIONS ────────────────────────────────────────
a = np.array([1, 2, 3, 4, 5])
b = np.array([3, 4, 5, 6, 7])
np.intersect1d(a, b)              # [3 4 5]
np.union1d(a, b)                  # [1 2 3 4 5 6 7]
np.setdiff1d(a, b)                # [1 2]  — in a but not b
```

---

## 17. Chart Selection and Design

### Choosing the right chart

The most common mistake in data visualisation is using the wrong chart type. Here is the decision framework:

| You want to show... | Chart type | When to avoid |
|--------------------|-----------|---------------|
| **Distribution of one variable** | Histogram, KDE, Box plot | Pie chart |
| **Comparison across categories** | Bar chart (vertical or horizontal) | 3D bars, exploded pie |
| **Trend over time** | Line chart | Bar chart for dense time series |
| **Relationship between two variables** | Scatter plot | Line chart (for non-ordered x) |
| **Composition (parts of a whole)** | Stacked bar, pie (if ≤5 categories) | Pie chart with many slices |
| **Ranking** | Horizontal bar (sorted) | Vertical bar when labels are long |
| **Correlation matrix** | Heatmap | Any other type |
| **Distribution across categories** | Box plot, Violin plot | Bar chart of means only |
| **Frequency of two categorical vars** | Grouped bar, Heatmap | Back-to-back bars |

---

### Visual design principles for analysts

```
1. Data-ink ratio — remove everything that doesn't add information
   ✓ Remove grid lines you don't need
   ✓ Remove chart borders (spines)
   ✓ Use light grey for non-data elements

2. Colour purposefully
   ✓ One highlight colour for "the answer"
   ✓ Sequential palette for ordered data (light → dark)
   ✓ Diverging palette for data with a meaningful midpoint (e.g. profit/loss)
   ✓ Qualitative palette for categories (distinct, no implied order)
   ✗ Avoid rainbow colour maps — they mislead

3. Label directly — avoid legends where possible
   ✓ Label data points or line ends directly
   ✗ Legends require eye movement to decode

4. Sort your bars
   ✓ Sort horizontal bar charts by value (descending)
   ✗ Alphabetical ordering is almost never useful

5. Titles answer the question
   ✓ "November revenue was 23% above forecast"  ← tells the story
   ✗ "Monthly Revenue"  ← just labels the axis
```

---

## 18. Matplotlib — Complete Reference

### Anatomy of a figure

```python
import matplotlib.pyplot as plt

# A Figure contains one or more Axes objects
# An Axes is one "plot" (has x-axis, y-axis, title, data)
# An Axis is the actual x or y scale

fig, ax = plt.subplots(figsize=(10, 6))
# fig = Figure (the whole canvas)
# ax  = Axes   (the plot area)

# Style the Axes
ax.set_title("Monthly Revenue — Olist 2018", fontsize=14, fontweight="bold", pad=15)
ax.set_xlabel("Month",   fontsize=11)
ax.set_ylabel("Revenue (R$)", fontsize=11)
ax.tick_params(labelsize=9)
ax.grid(True, axis="y", alpha=0.3, linestyle="--")  # light horizontal gridlines only

# Remove top and right spines (cleaner look)
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)
```

---

### Line chart — full reference

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker

months  = list(range(1, 13))
revenue = [45000, 52000, 48000, 61000, 55000, 70000, 68000, 72000, 65000, 80000, 90000, 110000]
rolling = [None, None, (45000+52000+48000)/3, ...]  # 3-month rolling avg

fig, ax = plt.subplots(figsize=(12, 6))

# Main line
ax.plot(months, revenue,
        color="#2196F3",          # hex colour
        linewidth=2.5,
        marker="o",               # marker style: "o", "s", "^", "D", "x", "*"
        markersize=6,
        markerfacecolor="white",  # hollow markers
        markeredgewidth=2,
        label="Monthly Revenue",
        zorder=3)                 # zorder controls overlap order

# Add value labels on each point
for m, r in zip(months, revenue):
    ax.annotate(f"R${r/1000:.0f}k",
                xy=(m, r),
                xytext=(0, 10),
                textcoords="offset points",
                ha="center",
                fontsize=8,
                color="grey")

# Y-axis formatting
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f"R${x/1000:.0f}k"))

# Highlight a specific region
ax.axvspan(11, 12, alpha=0.15, color="gold", label="Holiday Season")
ax.axhline(y=60000, color="red", linestyle="--", alpha=0.5, label="Target R$60k")

ax.set_title("Olist Monthly Revenue 2018", fontsize=14, fontweight="bold")
ax.set_xlabel("Month")
ax.set_ylabel("Revenue (R$)")
ax.legend(frameon=False)
ax.set_xticks(months)
ax.set_xticklabels(["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"])

plt.tight_layout()
plt.savefig("olist_revenue.png", dpi=150, bbox_inches="tight")
plt.show()
```

---

### Bar chart — full reference

```python
import matplotlib.pyplot as plt
import numpy as np

categories  = ["Electronics", "Clothing", "Home", "Sports", "Beauty", "Books"]
revenue_23  = [120000, 85000, 65000, 45000, 35000, 25000]
revenue_22  = [100000, 80000, 60000, 40000, 30000, 20000]

fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# ── SIMPLE BAR ────────────────────────────────────────────
colors = ["#2196F3" if r > 60000 else "#90CAF9" for r in revenue_23]  # highlight top categories
bars = axes[0].bar(categories, revenue_23, color=colors, edgecolor="white", width=0.6)

# Add value labels on top of bars
for bar in bars:
    h = bar.get_height()
    axes[0].text(bar.get_x() + bar.get_width()/2., h + 1000,
                 f"${h/1000:.0f}k", ha="center", va="bottom", fontsize=9, fontweight="bold")

axes[0].set_title("Revenue by Category (2023)", fontweight="bold")
axes[0].set_ylabel("Revenue (USD)")
axes[0].set_ylim(0, max(revenue_23) * 1.15)
axes[0].tick_params(axis="x", rotation=30)

# ── GROUPED BAR ───────────────────────────────────────────
x     = np.arange(len(categories))
width = 0.35
axes[1].bar(x - width/2, revenue_22, width, label="2022", color="#90CAF9", edgecolor="white")
axes[1].bar(x + width/2, revenue_23, width, label="2023", color="#2196F3", edgecolor="white")

axes[1].set_title("Revenue by Category: 2022 vs 2023", fontweight="bold")
axes[1].set_xticks(x)
axes[1].set_xticklabels(categories, rotation=30, ha="right")
axes[1].legend(frameon=False)

plt.tight_layout()
plt.show()
```

---

### Scatter, histogram, and subplots

```python
# ── SCATTER ───────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(9, 7))

scatter = ax.scatter(
    df["rating"], df["box_office"],
    s=df["year"].apply(lambda y: (y - 1990) * 5 + 20),  # size = relative to year
    c=df["rating"],            # colour = rating value
    cmap="RdYlGn",             # red-yellow-green colourmap
    alpha=0.7,
    edgecolors="white",
    linewidth=0.5
)
plt.colorbar(scatter, ax=ax, label="Rating")

# Annotate notable points
for _, row in df.nlargest(3, "box_office").iterrows():
    ax.annotate(row["title"],
                (row["rating"], row["box_office"]),
                xytext=(5, 10), textcoords="offset points",
                fontsize=8, ha="left",
                arrowprops=dict(arrowstyle="->", color="grey", lw=0.8))

ax.set_xlabel("IMDb Rating")
ax.set_ylabel("Box Office ($M)")
ax.set_title("Rating vs Box Office (bubble size = recency)")

# ── HISTOGRAM ─────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(9, 5))
n, bins, patches = ax.hist(df["rating"], bins=25, edgecolor="white",
                            color="#2196F3", alpha=0.8)

# Colour bars below mean red, above mean blue
mean_rating = df["rating"].mean()
for patch, left_edge in zip(patches, bins):
    if left_edge < mean_rating:
        patch.set_facecolor("#EF5350")

ax.axvline(mean_rating, color="black", linestyle="--", linewidth=1.5,
           label=f"Mean: {mean_rating:.2f}")
ax.axvline(df["rating"].median(), color="orange", linestyle=":", linewidth=1.5,
           label=f"Median: {df['rating'].median():.2f}")
ax.legend(frameon=False)
ax.set_xlabel("Rating")
ax.set_ylabel("Frequency")

# ── SUBPLOTS ──────────────────────────────────────────────
fig, axes = plt.subplots(2, 3, figsize=(18, 10))
fig.suptitle("Data Analysis Dashboard", fontsize=16, fontweight="bold", y=1.02)

# axes[row, col] indexing
axes[0, 0].plot(x_data, y_data)
axes[0, 1].bar(categories, values)
axes[0, 2].scatter(x, y, alpha=0.5)
axes[1, 0].hist(dist_data, bins=20)
axes[1, 1].boxplot([group1, group2, group3])
axes[1, 2].pie(sizes, labels=labels, autopct="%1.1f%%")

plt.tight_layout()
plt.savefig("dashboard.png", dpi=150, bbox_inches="tight")
plt.show()
```

---

## 19. Seaborn — Statistical Visualisation

### Setup and themes

```python
import seaborn as sns
import matplotlib.pyplot as plt

# Built-in themes
sns.set_theme(style="whitegrid")    # white background + light grey grid
sns.set_theme(style="darkgrid")     # dark background (good for dense charts)
sns.set_theme(style="white")        # clean, no grid
sns.set_theme(style="ticks")        # minimal — axes and ticks only
sns.set_theme(style="dark")         # dark, no grid

# Context (scales fonts and line widths for different outputs)
sns.set_context("notebook")         # default — good for Jupyter
sns.set_context("paper")            # smaller (for publications)
sns.set_context("talk")             # larger fonts (for presentations)
sns.set_context("poster")           # largest

# Colour palettes
sns.color_palette("Set2")           # qualitative — distinct categories
sns.color_palette("Blues")          # sequential — ordered numeric
sns.color_palette("RdBu")           # diverging — around a midpoint (profit/loss)
sns.color_palette("viridis")        # perceptually uniform, colourblind-friendly
```

---

### Distribution plots

```python
# ── HISTOGRAM + KDE ───────────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Left: basic histogram with density curve
sns.histplot(data=df, x="payment_value", kde=True, bins=40,
             color="steelblue", ax=axes[0])
axes[0].set_title("Payment Distribution")

# Right: KDE comparison by group
sns.kdeplot(data=df, x="payment_value", hue="payment_type",
            fill=True, alpha=0.3, ax=axes[1])
axes[1].set_title("Payment Distribution by Type")
plt.tight_layout()
plt.show()

# ── BOX PLOT — quartiles + outliers ──────────────────────
fig, ax = plt.subplots(figsize=(12, 6))
sns.boxplot(data=df, x="product_category", y="price",
            order=df.groupby("product_category")["price"].median().sort_values().index,
            palette="Blues", width=0.6, ax=ax)
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, ha="right")
ax.set_title("Price Distribution by Category\n(sorted by median)")

# ── VIOLIN PLOT — distribution shape + quartiles ─────────
sns.violinplot(data=df, x="category", y="revenue",
               inner="quartile",   # show quartiles inside violin
               palette="Set2")
```

---

### Relationship and categorical plots

```python
# ── SCATTER WITH REGRESSION ───────────────────────────────
sns.regplot(data=df, x="rating", y="box_office",
            scatter_kws={"alpha": 0.5, "s": 50},
            line_kws={"color": "red", "linewidth": 1.5})
plt.title("Rating vs Box Office with Trend Line")

# ── SCATTER COLOURED BY CATEGORY ──────────────────────────
sns.scatterplot(data=df, x="rating", y="box_office",
                hue="genre",            # colour by genre
                size="year",            # size by year
                style="language",       # marker style by language
                palette="Set2",
                sizes=(30, 200))
plt.legend(bbox_to_anchor=(1.05, 1), loc="upper left")  # legend outside plot

# ── HEATMAP — correlation matrix ──────────────────────────
corr = df[["rating", "box_office", "runtime_min", "year"]].corr()

# Mask the upper triangle (avoid redundancy)
mask = np.triu(np.ones_like(corr, dtype=bool))

fig, ax = plt.subplots(figsize=(8, 7))
sns.heatmap(
    corr, mask=mask,
    annot=True,             # show values in cells
    fmt=".2f",              # format as 2 decimal places
    cmap="coolwarm",        # diverging colourmap (red=positive, blue=negative)
    center=0,               # midpoint of colour scale at 0
    vmin=-1, vmax=1,        # fix colour scale to [-1, 1]
    square=True,            # square cells
    linewidths=0.5,         # grid lines between cells
    cbar_kws={"shrink": 0.8},
    ax=ax
)
ax.set_title("Correlation Matrix — Movie Features")

# ── PAIR PLOT — all pairs of numeric columns ──────────────
sns.pairplot(
    df[["rating", "box_office", "runtime_min", "year"]],
    diag_kind="hist",       # diagonal: histogram or "kde"
    plot_kws={"alpha": 0.5, "s": 30},
    corner=True             # show only lower triangle
)
plt.suptitle("Pairwise Relationships", y=1.02)
```

---

### FacetGrid — small multiples

```python
# One chart per category — powerful for comparison
g = sns.FacetGrid(df, col="genre", col_wrap=3, height=4, aspect=1.2)
g.map(sns.histplot, "rating", bins=15, color="steelblue")
g.set_titles(col_template="{col_name}")
g.set_axis_labels("Rating", "Count")
g.figure.suptitle("Rating Distribution by Genre", y=1.02)
plt.tight_layout()
plt.show()

# FacetGrid with scatter
g = sns.FacetGrid(df, col="admission_type", hue="gender", height=5)
g.map(plt.scatter, "age", "billing_amount", alpha=0.4, s=20)
g.add_legend()
```

---
---

## 20. EDA — Exploratory Data Analysis

### What is EDA and why does it matter?

**Exploratory Data Analysis (EDA)** is the process of examining a dataset before formal modelling or reporting. It was systematised by statistician John Tukey in the 1970s. The goal is not to answer a specific question — it is to understand the data: its shape, quality, distributions, relationships, and anomalies.

**Why EDA is non-negotiable:**

| Skip EDA and you risk... | Example consequence |
|--------------------------|---------------------|
| Modelling on bad data | Your "fraud rate is 0.1%" finding is wrong because cancellations were miscoded as fraud |
| Missing the real story | Revenue appears flat — but EDA reveals two segments: B2B growing 40%, B2C declining 20% |
| Wrong statistical tests | You apply a t-test assuming normality — but the distribution is heavily skewed |
| Misleading visualisations | Average delivery time = 5 days, but P90 = 25 days — the average hides everything |
| Anchoring on outliers | One ₹10M order inflates average order value by 200% |

**The mental model:** EDA is detective work. You are not yet solving the case — you are gathering clues, building hypotheses, and figuring out what questions to ask next.

---

### The 6-Stage EDA Framework

```
Stage 1: Understand the data source
Stage 2: Structural inspection (shape, types, nulls, duplicates)
Stage 3: Univariate analysis (one variable at a time)
Stage 4: Bivariate analysis (pairs of variables)
Stage 5: Multivariate analysis (many variables together)
Stage 6: Formulate and document hypotheses
```

We will work through every stage in depth.

---

### Stage 1: Understand the Data Source

Before writing a single line of code, answer these questions:

| Question | Why it matters |
|----------|----------------|
| Where did this data come from? | Transactional DB? Survey? API export? Determines biases and gaps |
| What does one row represent? | One order? One customer? One product? Wrong assumption → wrong analysis |
| What time period does it cover? | Are you missing recent months? Does it span a COVID period? |
| How was it collected? | Manual entry → expect typos. Automated system → expect schema issues |
| What are the known data quality issues? | Every production system has known quirks — ask the data owner |
| What business question am I trying to answer? | Guides which variables matter |

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_theme(style="whitegrid")

# Load the dataset
df = pd.read_csv("data/olist_orders_dataset.csv")

# First, document what you understand
print("""
Dataset: Olist Brazilian E-Commerce
Source:  Kaggle — exported from Olist's production database (2016–2018)
Grain:   One row = one order
Known issues: Some delivered_date fields are null for non-delivered orders (expected)
""")
```

---

### Stage 2: Structural Inspection

This stage answers: *Is the data well-formed? How complete is it?*

```python
# ── SIZE ──────────────────────────────────────────────────
print(f"Rows:    {df.shape[0]:,}")
print(f"Columns: {df.shape[1]}")
print(f"Cells:   {df.size:,}")
print(f"Memory:  {df.memory_usage(deep=True).sum() / 1024**2:.1f} MB")

# ── FIRST LOOK ────────────────────────────────────────────
df.head()         # first 5 rows — check values look right
df.tail()         # last 5 rows — check end of file isn't corrupt
df.sample(10)     # random sample — reduces confirmation bias vs always seeing first rows

# ── COLUMN TYPES ──────────────────────────────────────────
df.dtypes
# order_id                           object   ← should be str ✓
# customer_id                        object   ← should be str ✓
# order_status                       object   ← categorical ✓
# order_purchase_timestamp           object   ← should be datetime! ✗
# order_delivered_customer_date      object   ← should be datetime! ✗

# → ACTION: parse date columns

df.info()   # condensed view: column name, non-null count, dtype
# Columns with non-null count < total rows → have nulls

# ── NULL AUDIT ────────────────────────────────────────────
null_report = pd.DataFrame({
    "dtype":       df.dtypes,
    "null_count":  df.isnull().sum(),
    "null_pct":    (df.isnull().mean() * 100).round(1),
    "sample_value": df.iloc[0]   # first non-null value
}).query("null_count > 0").sort_values("null_pct", ascending=False)

print(null_report)
# order_delivered_customer_date   2965 nulls (2.9%)  ← expected for non-delivered orders
# order_approved_at                 160 nulls (0.2%)  ← investigate

# ── INFERENCES FROM NULLS ─────────────────────────────────
# Null patterns are not random — they tell a story.
# 2.9% null in delivered_date → check: are these all non-delivered orders?
null_delivered  = df[df["order_delivered_customer_date"].isnull()]["order_status"].value_counts()
print(null_delivered)
# If all nulls are in "shipped", "invoiced", etc. → nulls are STRUCTURAL (expected)
# If nulls appear in "delivered" status → DATA QUALITY PROBLEM

# ── DUPLICATE CHECK ───────────────────────────────────────
print(f"Exact duplicate rows:     {df.duplicated().sum()}")
print(f"Duplicate order_id:       {df.duplicated(subset=['order_id']).sum()}")
print(f"Unique order_ids:         {df['order_id'].nunique()}")
# If unique order_ids ≠ len(df) → multiple rows per order (JOIN explosion or real duplicates)

# ── VALUE RANGES ──────────────────────────────────────────
df.describe()
# Check: are min/max values sensible?
# year 2100 in a 2018 dataset → data entry error
# negative price → data error or legitimate credit
# age 200 → error
# rating 0.0 → real "unrated" or default fill?
```

---

### Stage 3: Univariate Analysis

Univariate analysis examines **one variable at a time**. You are asking: *What does this variable look like? What is its range, centre, spread, and shape?*

#### 3A: Numeric variables

```python
# Parse dates first (needed for later analysis)
date_cols = ["order_purchase_timestamp", "order_delivered_customer_date",
             "order_estimated_delivery_date", "order_approved_at"]
for col in date_cols:
    df[col] = pd.to_datetime(df[col], errors="coerce")

# Merge payments for numeric analysis
payments = pd.read_csv("data/olist_order_payments_dataset.csv")
order_totals = payments.groupby("order_id")["payment_value"].sum().reset_index()
order_totals.columns = ["order_id", "total_payment"]
df = df.merge(order_totals, on="order_id", how="left")

# ── SUMMARY STATISTICS ────────────────────────────────────
stats = df["total_payment"].describe(percentiles=[.1, .25, .5, .75, .9, .95, .99])
print(stats)
# count     99441
# mean        160.45    ← average order value
# std         218.71    ← very high std relative to mean → heavily skewed or outliers
# min           0.00    ← orders of $0? Worth investigating
# 10%          29.99
# 25%          56.99    ← Q1
# 50%          96.99    ← median (much lower than mean → right skew)
# 75%         179.99    ← Q3
# 90%         350.00
# 95%         499.99
# 99%         960.00
# max       13664.08   ← maximum order ($13k) — outlier?

# KEY INSIGHT: mean ($160) >> median ($97) → RIGHT SKEWED distribution
# This is typical of transaction data — most orders are small, a few are very large

# ── VISUALISE THE DISTRIBUTION ────────────────────────────
fig, axes = plt.subplots(1, 3, figsize=(18, 5))
fig.suptitle("Payment Value Distribution", fontsize=14, fontweight="bold")

# 1. Histogram
axes[0].hist(df["total_payment"].dropna(), bins=50, color="steelblue", edgecolor="white")
axes[0].axvline(df["total_payment"].mean(),   color="red",    linestyle="--", label="Mean")
axes[0].axvline(df["total_payment"].median(), color="orange", linestyle=":",  label="Median")
axes[0].set_title("Histogram")
axes[0].set_xlabel("Payment Value (R$)")
axes[0].legend(frameon=False)

# 2. Log-transformed histogram (reveals structure hidden by outliers)
axes[1].hist(np.log1p(df["total_payment"].dropna()), bins=50, color="teal", edgecolor="white")
axes[1].set_title("Log-transformed (reveals true shape)")
axes[1].set_xlabel("log(1 + Payment Value)")

# 3. Box plot (shows quartiles and outlier dots)
axes[2].boxplot(df["total_payment"].dropna(), vert=True, patch_artist=True,
                boxprops=dict(facecolor="lightblue"))
axes[2].set_title("Box Plot")
axes[2].set_ylabel("Payment Value (R$)")

plt.tight_layout()
plt.show()
```

#### Reading histograms — what to look for

```
SHAPE → what it means → what to do

Normal (bell curve):
  → data is symmetrically distributed around the mean
  → mean ≈ median; parametric stats (t-test, etc.) are valid

Right skew (long tail on right):
  → typical for financial/transaction data
  → mean > median; outliers on the high end
  → consider log-transform before modelling

Left skew (long tail on left):
  → common for satisfaction scores, test results (many near max)
  → mean < median

Bimodal (two peaks):
  → TWO POPULATIONS MIXED TOGETHER
  → e.g. a "delivery time" bimodal: domestic vs international?
  → VERY IMPORTANT: investigate what the two groups are — segment separately

Uniform (flat):
  → all values equally likely — often sampling/binning artefact
  → or data is genuinely spread (e.g. random IDs)

Spike at boundary:
  → rounding/truncation (many values at exactly 0, 100, or 1000)
  → or a default fill (someone set unknowns to 0)
```

```python
# Detect skewness numerically
from scipy import stats
skew = df["total_payment"].skew()
kurt = df["total_payment"].kurt()
print(f"Skewness: {skew:.2f}")  # >1 = strongly right-skewed, <-1 = left-skewed
print(f"Kurtosis: {kurt:.2f}")  # >3 = heavy tails (more outliers than normal)

# Rule of thumb:
# |skewness| < 0.5  → approximately symmetric
# 0.5 to 1.0       → moderately skewed
# > 1.0            → highly skewed → consider transform or non-parametric methods
```

---

#### 3B: Categorical variables

```python
# ── VALUE COUNTS ──────────────────────────────────────────
status_counts = df["order_status"].value_counts()
status_pct    = df["order_status"].value_counts(normalize=True).mul(100).round(1)
print(pd.concat([status_counts, status_pct], axis=1, keys=["count", "%"]))

# order_status  count      %
# delivered     96478   96.5%
# shipped         1107    1.1%
# canceled         625    0.6%
# ...

# INFERENCE: 96.5% delivery rate — look strong, but:
# - Are the 1.1% "shipped" orders simply in transit?
# - Are the 0.6% cancellations concentrated in certain categories or sellers?
# Always look at what's NOT the majority — it often tells a more interesting story

# ── VISUALISE CATEGORICAL ─────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# Ordered bar chart
status_counts.plot.barh(ax=axes[0], color="steelblue")
axes[0].set_title("Orders by Status")
axes[0].invert_yaxis()   # most common at top

# Pie chart (only good for ≤ 5 categories)
top5 = df["order_status"].value_counts().head(5)
axes[1].pie(top5, labels=top5.index, autopct="%1.1f%%", startangle=140)
axes[1].set_title("Order Status Mix")

plt.tight_layout()
plt.show()

# ── HIGH-CARDINALITY CATEGORICAL ──────────────────────────
# Columns like city (1000+ unique values) — don't plot all, summarise
n_unique   = df["customer_city"].nunique()
top10      = df["customer_city"].value_counts().head(10)
pct_in_top10 = top10.sum() / len(df) * 100
print(f"\ncustomer_city: {n_unique} unique values")
print(f"Top 10 cities account for {pct_in_top10:.1f}% of all orders")
```

---

### Stage 4: Bivariate Analysis

Bivariate analysis examines **pairs of variables**. You are asking: *Do these two variables relate to each other? How?*

#### 4A: Numeric vs Numeric

```python
# ── CORRELATION ───────────────────────────────────────────
# Calculate: merge in delivery days and review scores
df["delivery_days"] = (df["order_delivered_customer_date"] - df["order_purchase_timestamp"]).dt.days

reviews = pd.read_csv("data/olist_order_reviews_dataset.csv")
review_avg = reviews.groupby("order_id")["review_score"].mean()
df = df.join(review_avg.rename("avg_review"), on="order_id")

corr_data = df[["total_payment", "delivery_days", "avg_review"]].dropna()
corr_matrix = corr_data.corr()
print(corr_matrix)
#                  total_payment  delivery_days  avg_review
# total_payment         1.000000       0.031234   -0.004521
# delivery_days         0.031234       1.000000   -0.241358  ← longer delivery → lower score
# avg_review           -0.004521      -0.241358    1.000000

# INFERENCE: delivery_days and avg_review have r = -0.24
# Moderate negative correlation: longer deliveries tend to get lower reviews
# r = 0.24 is not strong, but it IS meaningful — worth investigating thresholds

# Interpreting correlation coefficients (Pearson's r):
# |r| < 0.1  → negligible
# |r| 0.1–0.3 → weak
# |r| 0.3–0.5 → moderate
# |r| 0.5–0.7 → strong
# |r| > 0.7  → very strong
# SIGN: positive = both increase together; negative = one increases, other decreases
# ⚠️ Correlation ≠ causation — always look for confounders

# ── SCATTER PLOT ──────────────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# Scatter: delivery days vs review score
sample = corr_data.sample(2000, random_state=42)   # sample for readability
axes[0].scatter(sample["delivery_days"], sample["avg_review"],
                alpha=0.2, s=10, color="steelblue")
# Add trend line
z = np.polyfit(corr_data["delivery_days"].dropna(), corr_data["avg_review"].dropna(), 1)
p = np.poly1d(z)
x_line = np.linspace(0, corr_data["delivery_days"].max(), 100)
axes[0].plot(x_line, p(x_line), "r--", linewidth=1.5, label=f"Trend (r={corr_matrix.loc['delivery_days','avg_review']:.2f})")
axes[0].set_xlabel("Delivery Days")
axes[0].set_ylabel("Avg Review Score")
axes[0].set_title("Delivery Speed vs Customer Satisfaction")
axes[0].legend(frameon=False)

# Correlation heatmap
mask = np.triu(np.ones_like(corr_matrix, dtype=bool))
sns.heatmap(corr_matrix, mask=mask, annot=True, fmt=".2f",
            cmap="coolwarm", center=0, vmin=-1, vmax=1,
            square=True, ax=axes[1])
axes[1].set_title("Correlation Matrix")

plt.tight_layout()
plt.show()
```

---

#### 4B: Numeric vs Categorical — Group comparison

```python
# Question: Does delivery time differ by order status?
# Are certain states taking longer to deliver?

# ── GROUPBY STATISTICS ────────────────────────────────────
customers = pd.read_csv("data/olist_customers_dataset.csv")
df = df.merge(customers[["customer_id", "customer_state"]], on="customer_id", how="left")

state_analysis = df.groupby("customer_state").agg(
    order_count  = ("order_id",       "count"),
    avg_days     = ("delivery_days",  "mean"),
    median_days  = ("delivery_days",  "median"),
    p90_days     = ("delivery_days",  lambda x: x.quantile(0.9)),
    avg_review   = ("avg_review",     "mean"),
    late_rate    = ("delivery_days",  lambda x: (x > 10).mean() * 100)
).round(2).sort_values("avg_days", ascending=False)

print(state_analysis.head(10))
# STATE  order_count  avg_days  median_days  p90_days  avg_review  late_rate
# RR          213       29.5       27.0       52.0       3.80       68.1
# AP          105       28.7       26.5       49.0       3.71       65.7
# AM         629       24.1       22.0       43.0       3.89       58.3
# SP       41746       7.9         6.0       16.0       4.08       18.4

# INFERENCE: States like RR (Roraima) take 4× longer than SP (São Paulo)
# These are Northern states — likely remote areas with poor logistics infrastructure
# The late_rate column confirms: 68% of orders to RR arrive beyond 10 days
# This directly explains why avg_review in RR (3.80) < SP (4.08)

# ── BOX PLOTS PER GROUP ───────────────────────────────────
# Show full distribution, not just averages
top_states = state_analysis.sort_values("order_count", ascending=False).head(8).index
df_top = df[df["customer_state"].isin(top_states)]

fig, ax = plt.subplots(figsize=(14, 6))
order = df_top.groupby("customer_state")["delivery_days"].median().sort_values().index
sns.boxplot(data=df_top, x="customer_state", y="delivery_days",
            order=order, palette="Blues", ax=ax)
ax.set_title("Delivery Time Distribution by State (Top 8 by Volume)", fontweight="bold")
ax.set_xlabel("State")
ax.set_ylabel("Delivery Days")
ax.set_ylim(0, 40)   # cap at 40 to show distribution, note outliers beyond
plt.tight_layout()
plt.show()

# ── AVOIDING THE AVERAGES TRAP ────────────────────────────
# Mean delivery time for state SP: 7.9 days — looks fine
# But P90 is 16 days — 10% of customers in SP wait 16+ days
# Average hides this tail. Always check percentiles.

# ── T-TEST: is the difference statistically significant? ──
from scipy import stats

sp_days  = df[df["customer_state"] == "SP"]["delivery_days"].dropna()
rr_days  = df[df["customer_state"] == "RR"]["delivery_days"].dropna()
t_stat, p_value = stats.ttest_ind(sp_days, rr_days)
print(f"SP vs RR delivery time: t={t_stat:.1f}, p={p_value:.2e}")
# t=-45.3, p=3.2e-89 → extremely significant difference
# But significance ≠ importance: effect size matters too

# Cohen's d (effect size)
pooled_std = np.sqrt((sp_days.std()**2 + rr_days.std()**2) / 2)
d = (rr_days.mean() - sp_days.mean()) / pooled_std
print(f"Cohen's d = {d:.2f}")   # d > 0.8 = large effect
# d = 2.8 → enormous effect — states are genuinely very different
```

---

#### 4C: Categorical vs Categorical

```python
# Question: Is there a relationship between payment type and order status?

# ── CONTINGENCY TABLE ─────────────────────────────────────
payments = pd.read_csv("data/olist_order_payments_dataset.csv")
df_pay = df.merge(payments.drop_duplicates("order_id")[["order_id","payment_type"]], on="order_id")

contingency = pd.crosstab(
    df_pay["order_status"],
    df_pay["payment_type"],
    normalize="index"   # row proportions → % of each status using each payment method
).mul(100).round(1)

print(contingency)
# payment_type   boleto  credit_card  debit_card  voucher
# order_status
# canceled        30.7       45.2         2.4       21.7
# delivered       16.7       76.4         1.7        5.2

# INFERENCE: Cancelled orders have a MUCH higher voucher rate (21.7% vs 5.2% for delivered)
# And lower credit card rate (45.2% vs 76.4%)
# Hypothesis: Vouchers may be used for orders that are more likely to be cancelled
# (gift cards, promotional codes → impulse purchases → higher cancellation?)

# ── CHI-SQUARE TEST: are these distributions different? ───
from scipy.stats import chi2_contingency
contingency_raw = pd.crosstab(df_pay["order_status"], df_pay["payment_type"])
chi2, p, dof, expected = chi2_contingency(contingency_raw)
print(f"Chi-square: {chi2:.1f}, p={p:.2e}, dof={dof}")
# Chi-square: 3241.2, p=1.3e-690 → extremely significant
# The payment type distribution IS different across order statuses

# ── VISUALISE ─────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(12, 6))
contingency.T.plot.bar(ax=ax, width=0.7, colormap="Set2", edgecolor="white")
ax.set_title("Payment Type Mix by Order Status", fontweight="bold")
ax.set_xlabel("Payment Type")
ax.set_ylabel("% of Orders")
ax.legend(title="Order Status", bbox_to_anchor=(1.01, 1), frameon=False)
plt.xticks(rotation=0)
plt.tight_layout()
plt.show()
```

---

### Stage 5: Multivariate Analysis

Multivariate analysis looks at **many variables simultaneously**. Key techniques:

```python
# ── PAIR PLOT — visual correlation matrix with distributions ──
numeric_cols = ["total_payment", "delivery_days", "avg_review"]
df_sample = df[numeric_cols].dropna().sample(2000, random_state=42)

g = sns.pairplot(df_sample, diag_kind="hist",
                 plot_kws={"alpha": 0.3, "s": 10},
                 diag_kws={"bins": 30, "color": "steelblue"})
g.figure.suptitle("Pairwise Relationships — Olist", y=1.02)
plt.show()

# ── GROUPED SCATTER — three variables at once ──────────────
fig, ax = plt.subplots(figsize=(12, 8))
states_sample = df[df["customer_state"].isin(top_states)].dropna(
    subset=["delivery_days", "avg_review", "total_payment"]
)

for state, group in states_sample.groupby("customer_state"):
    ax.scatter(group["delivery_days"], group["avg_review"],
               s=group["total_payment"] / 10,    # size = order value
               alpha=0.3, label=state)

ax.set_xlabel("Delivery Days")
ax.set_ylabel("Review Score")
ax.set_title("Delivery Days vs Review Score (bubble size = order value)")
ax.legend(title="State", bbox_to_anchor=(1.01, 1), frameon=False)
plt.tight_layout()
plt.show()

# ── HEATMAP OF AGGREGATED DATA — often more readable than scatter ──
# Reviews by (delivery_band × payment_type)
df["delivery_band"] = pd.cut(df["delivery_days"],
                              bins=[0, 5, 10, 20, 50, 999],
                              labels=["0-5d","5-10d","10-20d","20-50d","50+d"])

pivot = pd.pivot_table(df_pay,
                       values="avg_review",
                       index="delivery_band",
                       columns="payment_type",
                       aggfunc="mean")

fig, ax = plt.subplots(figsize=(10, 5))
sns.heatmap(pivot, annot=True, fmt=".2f", cmap="RdYlGn",
            vmin=3.0, vmax=5.0, ax=ax)
ax.set_title("Avg Review Score by Delivery Speed & Payment Type")
plt.tight_layout()
plt.show()

# INFERENCE: The heatmap shows review scores drop dramatically for 20+ day deliveries
# regardless of payment type — delivery speed is the primary driver of satisfaction
```

---

### Stage 6: Missing Data Patterns

Understanding WHY data is missing is crucial. There are three types:

```python
# ── THREE TYPES OF MISSING DATA ───────────────────────────

# MCAR — Missing Completely At Random
# The missingness has nothing to do with any variable (data entry error, sensor glitch)
# → Safe to drop or impute with mean/median
# Example: a few random order_ids failed to record delivery date due to system error

# MAR — Missing At Random
# Missingness depends on OTHER observed variables, not the missing variable itself
# Example: customers who paid with voucher are less likely to leave a review
# → More sophisticated imputation needed (group-based, model-based)

# MNAR — Missing Not At Random
# Missingness depends on the variable's OWN (unobserved) value
# Example: high-value orders are less likely to record a delivery date (security policy)
# → Most dangerous: dropping or imputing introduces systematic bias

# Test for MCAR vs MAR:
# Compare the distribution of other variables between missing and non-missing groups
missing_review_group    = df[df["avg_review"].isnull()]
non_missing_review_group = df[df["avg_review"].notna()]

print("Orders WITH review scores:")
print(non_missing_review_group["total_payment"].describe())
print("\nOrders WITHOUT review scores:")
print(missing_review_group["total_payment"].describe())

# If the distributions are similar → likely MCAR
# If significantly different → MAR or MNAR → don't just drop these rows

# ── NULL PATTERN VISUALISATION ────────────────────────────
import matplotlib.pyplot as plt

# Heatmap of all nulls
fig, ax = plt.subplots(figsize=(14, 4))
sns.heatmap(df.isnull(), cbar=False, yticklabels=False,
            cmap=["white", "salmon"], ax=ax)
ax.set_title("Null Value Map (salmon = null)")
plt.tight_layout()
plt.show()

# Look for PATTERNS in the heatmap:
# Diagonal stripes → same records are missing in multiple columns
# Vertical stripes → entire column is mostly null
# Random scattered → likely MCAR
```

---

### Stage 6: Formulate Hypotheses

After EDA, document what you found and what questions it raises.

```python
# This is not code — this is the most important output of EDA
# Write it as comments or a markdown cell

eda_findings = """
=== EDA FINDINGS — Olist Orders Dataset ===

1. DATA QUALITY
   - 2.9% null in order_delivered_date: EXPECTED (non-delivered orders) — OK to leave as null
   - 0.2% null in order_approved_at: INVESTIGATE — approval should always exist for delivered orders
   - No exact duplicate rows; no duplicate order_ids ✓
   - All date columns stored as object (string) — must parse on load

2. DISTRIBUTION INSIGHTS
   - Payment value is heavily RIGHT SKEWED (mean $160 >> median $97; skewness = 4.8)
   - Recommendation: use median for "typical order value" reporting, not mean
   - Log-transform payment values before any regression analysis

3. GEOGRAPHIC PATTERNS
   - Northern states (RR, AP, AM) have 3-4x longer delivery times than SP/RJ
   - This directly causes lower review scores in those states (r=-0.24)
   - Late delivery rate: 68% in RR vs 18% in SP — operational priority

4. PAYMENT TYPE PATTERNS
   - Cancelled orders use vouchers at 4× the rate of delivered orders (21.7% vs 5.2%)
   - Hypothesis: vouchers indicate lower customer commitment → higher cancellation intent
   - Recommendation: investigate with marketing team; consider voucher validation rules

5. CUSTOMER SATISFACTION
   - Review score is most strongly correlated with delivery speed
   - Orders delivered in ≤5 days: avg 4.3/5 | Orders in 20+ days: avg 3.2/5
   - Even credit card users (typically highest-value) rate badly with slow delivery
   - Delivery time improvement = highest-ROI lever for review score improvement

6. NEXT ANALYSIS STEPS
   - Segment analysis: do seller characteristics predict delivery failures?
   - Time-series: is delivery performance improving or worsening over 2016–2018?
   - Model: can we predict cancellation at order placement time?
"""
print(eda_findings)
```

---

### Complete EDA Template

Copy this template for every new dataset you encounter:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats
from pathlib import Path

sns.set_theme(style="whitegrid")
pd.set_option("display.max_columns", 50)
pd.set_option("display.float_format", "{:.2f}".format)

def run_eda(df: pd.DataFrame, target_col: str = None, name: str = "Dataset") -> dict:
    """
    Complete EDA pipeline. Returns a summary dict.
    
    Parameters
    ----------
    df         : DataFrame to analyse
    target_col : primary numeric column of interest
    name       : dataset name for plot titles
    """
    print(f"\n{'='*60}")
    print(f"EDA Report: {name}")
    print(f"{'='*60}")

    # ── 1. STRUCTURE ──────────────────────────────────────────
    print(f"\n[1] STRUCTURE")
    print(f"  Shape:  {df.shape[0]:,} rows × {df.shape[1]} columns")
    print(f"  Memory: {df.memory_usage(deep=True).sum()/1024**2:.1f} MB")
    print(f"\n  Dtypes:")
    print(df.dtypes.value_counts().to_string())

    # ── 2. NULL AUDIT ─────────────────────────────────────────
    print(f"\n[2] NULL AUDIT")
    nulls = pd.DataFrame({
        "null_count": df.isnull().sum(),
        "null_pct":   (df.isnull().mean()*100).round(1)
    }).query("null_count > 0").sort_values("null_pct", ascending=False)
    if nulls.empty:
        print("  No nulls found ✓")
    else:
        print(nulls.to_string())

    # ── 3. DUPLICATES ─────────────────────────────────────────
    print(f"\n[3] DUPLICATES")
    n_dups = df.duplicated().sum()
    print(f"  Exact duplicates: {n_dups} ({n_dups/len(df):.1%})")

    # ── 4. NUMERIC SUMMARY ────────────────────────────────────
    print(f"\n[4] NUMERIC COLUMNS")
    num_cols = df.select_dtypes(include="number").columns.tolist()
    if num_cols:
        summary = df[num_cols].describe(percentiles=[.1,.25,.5,.75,.9,.99]).T
        summary["skewness"] = df[num_cols].skew()
        summary["iqr"] = summary["75%"] - summary["25%"]
        print(summary[["count","mean","std","min","50%","max","skewness"]].round(2).to_string())

    # ── 5. CATEGORICAL SUMMARY ────────────────────────────────
    print(f"\n[5] CATEGORICAL COLUMNS")
    cat_cols = df.select_dtypes(include="object").columns.tolist()
    for col in cat_cols[:8]:   # limit to first 8
        n_unique = df[col].nunique()
        top_val  = df[col].value_counts().index[0] if n_unique > 0 else "N/A"
        top_pct  = df[col].value_counts(normalize=True).iloc[0] * 100 if n_unique > 0 else 0
        print(f"  {col:<40} {n_unique:>6} unique  top: '{top_val}' ({top_pct:.1f}%)")

    # ── 6. CORRELATION ────────────────────────────────────────
    if len(num_cols) >= 2:
        print(f"\n[6] CORRELATIONS")
        corr = df[num_cols].corr()
        # Find top correlations (excluding diagonal)
        corr_pairs = corr.where(~np.eye(len(corr), dtype=bool)).stack()
        print("  Top 5 absolute correlations:")
        print(corr_pairs.abs().nlargest(5).to_string())

    # ── 7. VISUALISATIONS ─────────────────────────────────────
    n_num = min(len(num_cols), 4)
    if n_num > 0:
        fig, axes = plt.subplots(2, n_num, figsize=(5*n_num, 10))
        if n_num == 1:
            axes = axes.reshape(2, 1)
        for i, col in enumerate(num_cols[:n_num]):
            col_data = df[col].dropna()
            axes[0, i].hist(col_data, bins=30, color="steelblue", edgecolor="white")
            axes[0, i].axvline(col_data.mean(),   color="red",    linestyle="--", alpha=0.7)
            axes[0, i].axvline(col_data.median(), color="orange", linestyle=":",  alpha=0.7)
            axes[0, i].set_title(f"{col}\nμ={col_data.mean():.1f} | M={col_data.median():.1f}")
            axes[1, i].boxplot(col_data, vert=True, patch_artist=True,
                               boxprops=dict(facecolor="lightblue", alpha=0.7))
            axes[1, i].set_title(f"{col} — Outliers")
        fig.suptitle(f"{name} — Numeric Distributions", fontsize=14, fontweight="bold")
        plt.tight_layout()
        plt.savefig(f"eda_{name.lower().replace(' ','_')}_distributions.png",
                    dpi=120, bbox_inches="tight")
        plt.show()

    # ── RETURN SUMMARY ────────────────────────────────────────
    return {
        "shape": df.shape,
        "null_cols": nulls.index.tolist(),
        "null_pct_max": nulls["null_pct"].max() if not nulls.empty else 0,
        "n_duplicates": n_dups,
        "numeric_cols": num_cols,
        "categorical_cols": cat_cols,
    }

# Usage:
df = pd.read_csv("data/olist_orders_dataset.csv")
summary = run_eda(df, target_col="payment_value", name="Olist Orders")
```

---

### Common EDA Inferences — Pattern Library

```
PATTERN: Mean >> Median in distribution
INFERENCE: Right-skewed data. Outliers inflating mean.
ACTION: Use median for "typical" reporting. Log-transform for modelling.

PATTERN: Bimodal distribution
INFERENCE: Two sub-populations are mixed in one column.
ACTION: Find the grouping variable (geography? product type? time period?). Segment separately.

PATTERN: High null rate in one column after filtering
INFERENCE: The nulls are structural — they depend on another variable's value.
ACTION: Don't impute; instead model the null as a category or join conditionally.

PATTERN: Correlation weakens after removing one outlier customer/region
INFERENCE: The correlation was driven by that outlier, not a real general trend.
ACTION: Report with and without outlier; flag the outlier as a special case.

PATTERN: Strong correlation between two variables you didn't expect
INFERENCE: May be genuine, or may be confounded (both driven by a third variable).
ACTION: Check partial correlation (control for third variable). Don't report causation.

PATTERN: Review scores drop after a specific date
INFERENCE: Something changed on that date — a supplier, a pricing policy, a fulfillment partner.
ACTION: Treat as a breakpoint. Analyse pre-post separately.

PATTERN: Some category has 10× more rows than expected
INFERENCE: Either it's truly dominant, or there's a join that multiplied rows (check for duplicates).
ACTION: Verify row counts at each stage of the join.

PATTERN: Your "total" doesn't add up
INFERENCE: Overlap (a customer can appear in multiple segments), or double-counting (daily data 
           summed without removing intra-day duplicates).
ACTION: Use DISTINCT counts. Verify grain of each table before joining.
```

---
---

## 21. sqlite3 — Running SQL from Python

### Why Python + SQL together?

SQL is for querying structured data. Python is for everything else: cleaning, joining, transforming, visualising, automating, and building pipelines. The combination is what real data analysts and engineers use daily.

```python
import sqlite3
import pandas as pd
from pathlib import Path
```

---

### Connecting and creating tables

```python
# Connect (creates the .db file if it doesn't exist)
conn = sqlite3.connect("olist_analysis.db")
cursor = conn.cursor()

# Enable foreign keys (OFF by default in SQLite)
cursor.execute("PRAGMA foreign_keys = ON")

# Create tables
cursor.execute("""
    CREATE TABLE IF NOT EXISTS movies (
        movie_id   INTEGER PRIMARY KEY AUTOINCREMENT,
        title      TEXT    NOT NULL,
        year       INTEGER CHECK (year BETWEEN 1888 AND 2100),
        rating     REAL    CHECK (rating BETWEEN 0 AND 10),
        director   TEXT
    )
""")
conn.commit()

# Insert single row
cursor.execute(
    "INSERT INTO movies (title, year, rating, director) VALUES (?, ?, ?, ?)",
    ("Inception", 2010, 8.8, "Nolan")
)
conn.commit()

# Insert many rows at once (much faster than looping)
cursor.executemany(
    "INSERT INTO movies (title, year, rating, director) VALUES (?, ?, ?, ?)",
    [
        ("Parasite",     2019, 8.5, "Bong"),
        ("Pulp Fiction", 1994, 8.9, "Tarantino"),
        ("Barbie",       2023, 7.0, "Gerwig"),
    ]
)
conn.commit()

# ⚠️ CRITICAL: NEVER use string formatting for SQL
# This is SQL injection vulnerability:
user_input = "Inception'; DROP TABLE movies;--"
# cursor.execute(f"SELECT * FROM movies WHERE title = '{user_input}'")  ← DANGEROUS
# ✅ ALWAYS use parameterised queries:
cursor.execute("SELECT * FROM movies WHERE title = ?", (user_input,))   # SAFE
```

---

### Querying and fetching results

```python
# Fetch all rows as list of tuples
cursor.execute("SELECT * FROM movies WHERE rating > 8.0")
rows = cursor.fetchall()
for row in rows:
    print(row)    # (1, 'Inception', 2010, 8.8, 'Nolan')

# Fetch one row
cursor.execute("SELECT * FROM movies WHERE title = ?", ("Inception",))
row = cursor.fetchone()   # None if no match

# Fetch in batches (for large results)
cursor.execute("SELECT * FROM movies")
while batch := cursor.fetchmany(100):   # 100 rows at a time
    process(batch)

# Get column names
cursor.execute("SELECT * FROM movies LIMIT 1")
columns = [d[0] for d in cursor.description]
print(columns)   # ['movie_id', 'title', 'year', 'rating', 'director']
```

---

### The pandas + sqlite3 power combo

```python
# ── LOAD CSVs INTO SQLITE ─────────────────────────────────
import pandas as pd
import sqlite3

conn = sqlite3.connect("olist.db")

# Load each CSV into the database
csv_files = {
    "orders":    "data/olist_orders_dataset.csv",
    "customers": "data/olist_customers_dataset.csv",
    "payments":  "data/olist_order_payments_dataset.csv",
    "products":  "data/olist_products_dataset.csv",
    "sellers":   "data/olist_sellers_dataset.csv",
    "order_items": "data/olist_order_items_dataset.csv",
    "reviews":   "data/olist_order_reviews_dataset.csv",
}

for table_name, file_path in csv_files.items():
    df = pd.read_csv(file_path)
    df.to_sql(table_name, conn, if_exists="replace", index=False)
    print(f"✓ {table_name}: {len(df):,} rows")

# ── QUERY WITH SQL, GET BACK A DATAFRAME ──────────────────
# Now you can use full SQL syntax on your data
query = """
    SELECT
        c.customer_state,
        COUNT(DISTINCT o.order_id)         AS order_count,
        ROUND(SUM(p.payment_value), 2)     AS total_revenue,
        ROUND(AVG(p.payment_value), 2)     AS avg_order_value,
        ROUND(AVG(r.review_score), 2)      AS avg_review
    FROM orders o
    JOIN customers c      ON o.customer_id  = c.customer_id
    JOIN payments p       ON o.order_id     = p.order_id
    LEFT JOIN reviews r   ON o.order_id     = r.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY c.customer_state
    ORDER BY total_revenue DESC
    LIMIT 15
"""
state_summary = pd.read_sql(query, conn)
print(state_summary.to_string())

# ── PARAMETRISED QUERY IN pd.read_sql ─────────────────────
min_reviews  = 100
min_rating   = 4.0
query = """
    SELECT director, COUNT(*) AS films, AVG(rating) AS avg_rating
    FROM movies
    GROUP BY director
    HAVING COUNT(*) > ? AND AVG(rating) > ?
"""
result = pd.read_sql(query, conn, params=(min_reviews, min_rating))

# ── WRITE ANALYSIS RESULTS BACK TO DB ─────────────────────
rfm_df.to_sql("customer_rfm", conn, if_exists="replace", index=False)
# Now the RFM table is in the database — query it with SQL

conn.close()
```

---

### Context manager and helper patterns

```python
# Context manager: connection auto-closes on exit
with sqlite3.connect("movies.db") as conn:
    df = pd.read_sql("SELECT * FROM movies WHERE rating > 8", conn)
# conn automatically closed here

# Reusable query function
def query_db(sql: str, db_path: str = "analysis.db", params=None) -> pd.DataFrame:
    """Run a SQL query and return a DataFrame."""
    with sqlite3.connect(db_path) as conn:
        return pd.read_sql(sql, conn, params=params)

# Usage
top_sellers = query_db("""
    SELECT seller_id, COUNT(*) AS orders, SUM(price) AS revenue
    FROM order_items GROUP BY seller_id ORDER BY revenue DESC LIMIT 10
""")

# Reusable write function
def write_to_db(df: pd.DataFrame, table: str, db_path: str = "analysis.db",
                if_exists: str = "replace") -> None:
    """Write a DataFrame to SQLite."""
    with sqlite3.connect(db_path) as conn:
        df.to_sql(table, conn, if_exists=if_exists, index=False)
        print(f"✓ Written {len(df):,} rows to '{table}'")
```

---

## 22. SQLAlchemy — Database Abstraction

```python
pip install sqlalchemy psycopg2-binary   # add psycopg2 for PostgreSQL
from sqlalchemy import create_engine, text
import pandas as pd
```

### Connection strings for all databases

```python
# SQLite (local file)
engine = create_engine("sqlite:///movies.db")
engine = create_engine("sqlite:///data/olist.db")
engine = create_engine("sqlite:///:memory:")   # in-memory, no file

# PostgreSQL
engine = create_engine("postgresql://username:password@host:5432/database")
engine = create_engine("postgresql+psycopg2://user:pass@localhost/mydb")

# MySQL
engine = create_engine("mysql+pymysql://user:pass@host:3306/dbname")

# SQL Server
engine = create_engine("mssql+pyodbc://user:pass@host/dbname?driver=ODBC+Driver+17")

# Connection string from environment variable (best practice for production)
import os
engine = create_engine(os.environ["DATABASE_URL"])
```

---

### Core read/write patterns

```python
# Read SQL → DataFrame (same as sqlite3 but works for any DB)
df = pd.read_sql("SELECT * FROM orders LIMIT 1000", engine)
df = pd.read_sql_table("orders", engine)   # read entire table

# Write DataFrame → table
df.to_sql("orders_clean", engine, if_exists="replace", index=False,
          chunksize=10_000)   # write 10k rows at a time (better for large DataFrames)

# Parametrised query with text()
with engine.connect() as conn:
    result = conn.execute(
        text("SELECT * FROM movies WHERE year = :year AND rating > :min_r"),
        {"year": 2023, "min_r": 8.0}
    )
    rows = result.fetchall()
    df   = pd.DataFrame(rows, columns=result.keys())
```

---

## 23. Datetime Handling — Complete Guide

```python
import pandas as pd
from datetime import datetime, date, timedelta
```

### Python native datetime

```python
from datetime import datetime, date, timedelta, timezone

# Current moment
now   = datetime.now()                    # local time
utc   = datetime.now(timezone.utc)        # UTC-aware
today = date.today()                      # date only

# Parse from string
d1 = datetime.strptime("15/06/2024",         "%d/%m/%Y")
d2 = datetime.strptime("2024-06-15 14:30",   "%Y-%m-%d %H:%M")
d3 = datetime.strptime("June 15, 2024",       "%B %d, %Y")
d4 = datetime.strptime("15-Jun-24",           "%d-%b-%y")

# Format to string
now.strftime("%Y-%m-%d")           # '2024-06-15'
now.strftime("%d/%m/%Y")           # '15/06/2024'
now.strftime("%B %d, %Y")         # 'June 15, 2024'
now.strftime("%Y-%m-%d %H:%M:%S") # '2024-06-15 14:30:00'
now.strftime("%A")                 # 'Saturday'  (full weekday name)
now.strftime("%b")                 # 'Jun'        (abbreviated month)

# Arithmetic with timedelta
tomorrow   = today + timedelta(days=1)
last_week  = today - timedelta(weeks=1)
in_30_days = today + timedelta(days=30)
diff       = (today - date(2024, 1, 1)).days   # days since Jan 1
```

---

### pandas datetime — comprehensive reference

```python
# ── PARSE ─────────────────────────────────────────────────
df["date"] = pd.to_datetime(df["date"])                          # auto-detect format
df["date"] = pd.to_datetime(df["date"], format="%d/%m/%Y")      # explicit format (faster)
df["date"] = pd.to_datetime(df["date"], format="mixed")         # pandas 2.0+: mixed formats
df["date"] = pd.to_datetime(df["date"], errors="coerce")        # bad dates → NaT
df["date"] = pd.to_datetime(df["date"], unit="s")               # from Unix timestamp (seconds)
df["date"] = pd.to_datetime(df["date"], unit="ms")              # from Unix milliseconds

# ── EXTRACT COMPONENTS ────────────────────────────────────
df["year"]          = df["date"].dt.year               # 2024
df["month"]         = df["date"].dt.month              # 1–12
df["month_name"]    = df["date"].dt.month_name()       # "January"
df["month_abbr"]    = df["date"].dt.strftime("%b")     # "Jan"
df["day"]           = df["date"].dt.day                # 1–31
df["dayofweek"]     = df["date"].dt.dayofweek          # 0=Monday, 6=Sunday
df["day_name"]      = df["date"].dt.day_name()         # "Monday"
df["is_weekend"]    = df["date"].dt.dayofweek >= 5     # True for Sat/Sun
df["quarter"]       = df["date"].dt.quarter            # 1–4
df["week"]          = df["date"].dt.isocalendar().week  # ISO week number
df["day_of_year"]   = df["date"].dt.dayofyear          # 1–366
df["hour"]          = df["date"].dt.hour               # 0–23
df["minute"]        = df["date"].dt.minute
df["second"]        = df["date"].dt.second

# ── TRUNCATION ────────────────────────────────────────────
df["month_start"]   = df["date"].dt.to_period("M").dt.to_timestamp()
df["year_start"]    = df["date"].dt.to_period("Y").dt.to_timestamp()
df["quarter_start"] = df["date"].dt.to_period("Q").dt.to_timestamp()
df["week_start"]    = df["date"] - pd.to_timedelta(df["date"].dt.dayofweek, unit="D")

# Formatted strings
df["year_month"]    = df["date"].dt.strftime("%Y-%m")     # "2024-06"
df["quarter_label"] = df["date"].dt.year.astype(str) + "-Q" + df["date"].dt.quarter.astype(str)
# "2024-Q2"

# ── DURATION CALCULATION ──────────────────────────────────
df["days_diff"]    = (df["end_date"] - df["start_date"]).dt.days
df["hours_diff"]   = (df["end_date"] - df["start_date"]).dt.total_seconds() / 3600
df["months_diff"]  = (df["end_date"].dt.year - df["start_date"].dt.year) * 12 \
                   + (df["end_date"].dt.month - df["start_date"].dt.month)

# ── DATE OFFSETS ──────────────────────────────────────────
df["plus_7_days"]  = df["date"] + pd.DateOffset(days=7)
df["plus_1_month"] = df["date"] + pd.DateOffset(months=1)
df["plus_1_year"]  = df["date"] + pd.DateOffset(years=1)
df["next_biz_day"] = df["date"] + pd.offsets.BDay(1)      # next business day

# ── FILTERING BY DATE ─────────────────────────────────────
df[df["date"] >= "2023-01-01"]
df[df["date"].between("2023-01-01", "2023-12-31")]
df[df["date"].dt.year == 2023]
df[df["date"].dt.month.isin([6, 7, 8])]    # summer months
df[df["date"].dt.dayofweek < 5]            # weekdays only

# ── RESAMPLE ──────────────────────────────────────────────
# Requires datetime index
df.set_index("date", inplace=True)

# Frequency codes:
# "D" = calendar day, "B" = business day
# "W" = week (Sunday end), "W-MON" = week ending Monday
# "ME" = month end, "MS" = month start (pandas 2.2+)
# "QE" = quarter end, "YE" = year end
# "h" = hourly, "min" = minute, "s" = second

monthly = df["revenue"].resample("ME").sum().reset_index()
weekly  = df["revenue"].resample("W").agg(["sum", "mean", "count"])
daily   = df["revenue"].resample("D").sum().fillna(0)   # fill missing days with 0

# ── DATE RANGE GENERATION ─────────────────────────────────
# Generate every day in 2024
all_days  = pd.date_range("2024-01-01", "2024-12-31", freq="D")

# Generate 12 month-ends
months    = pd.date_range("2024-01-01", periods=12, freq="ME")

# Generate business days
biz_days  = pd.date_range("2024-01-01", "2024-12-31", freq="B")

# Fill gaps in time series (important for rolling calculations)
full_range = pd.date_range(df.index.min(), df.index.max(), freq="D")
df_filled  = df.reindex(full_range, fill_value=0)  # 0 revenue for missing days
```

---

## 24. String Cleaning and Regex

### pandas string operations — complete reference

```python
# All via the .str accessor — works column-wide without loops
col = df["description"]

# ── CASE AND WHITESPACE ───────────────────────────────────
col.str.lower()
col.str.upper()
col.str.title()
col.str.strip()                   # both ends
col.str.lstrip()                  # left only
col.str.rstrip()                  # right only
col.str.replace(r"\s+", " ", regex=True)   # collapse multiple spaces

# ── EXTRACT AND SPLIT ─────────────────────────────────────
col.str.split(",")                          # split into list
col.str.split(",", expand=True)             # split into separate columns
col.str.split(",").str[0]                   # first part only
col.str.get(0)                              # same as .str[0]
col.str.slice(0, 5)                         # first 5 characters
col.str.extract(r"(\d{4})")                 # extract 4-digit year (first match)
col.str.extractall(r"(\d+)")               # extract ALL digit sequences

# ── SEARCH AND MATCH ──────────────────────────────────────
col.str.contains("error", case=False, na=False)    # case-insensitive search
col.str.startswith("ORD-")
col.str.endswith(".csv")
col.str.match(r"^\d{5}$")                   # regex: exactly 5 digits (full string match)
col.str.fullmatch(r"[A-Z]{2}\d{4}")        # must match entire string

# ── REPLACE ───────────────────────────────────────────────
col.str.replace("São Paulo", "SP")
col.str.replace(r"\d+", "NUM", regex=True) # replace all digit sequences with "NUM"
col.str.replace(r"[^\w\s]", "", regex=True) # remove all punctuation
```

---

### Regex patterns for common data cleaning tasks

```python
import re
import pandas as pd

# ── PHONE NUMBERS ─────────────────────────────────────────
phones = pd.Series(["+91 (123) 456-7890", "123.456.7890", "  123 456 7890  ", "(123)4567890"])
# Extract only digits
phones.str.replace(r"[^\d]", "", regex=True)
# Result: ['911234567890', '1234567890', '1234567890', '1234567890']

# ── EMAIL VALIDATION ──────────────────────────────────────
emails = pd.Series(["user@example.com", "invalid-email", "user@company.org", ""])
is_valid = emails.str.match(r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$")
# [True, False, True, False]

# Extract domain
emails.str.extract(r"@(.+)$")
# [['example.com'], [NaN], ['company.org'], [NaN]]

# ── PRODUCT CODES ─────────────────────────────────────────
codes = pd.Series(["SKU-12345", "sku12345 ", "SKU 12345", "12345"])
# Standardise: uppercase, no spaces, prefix SKU-
cleaned = (codes.str.upper()
                .str.replace(r"[^A-Z0-9]", "", regex=True)   # remove non-alphanumeric
                .str.replace(r"^SKU", "", regex=True)          # remove existing SKU prefix
                .apply(lambda x: f"SKU-{x}"))
# ['SKU-12345', 'SKU-12345', 'SKU-12345', 'SKU-12345']

# ── EXTRACT NUMBERS FROM MIXED STRINGS ────────────────────
descriptions = pd.Series(["Price: $45.99", "2 items @ $12.50 each", "free shipping"])
prices = descriptions.str.extract(r"\$(\d+\.?\d*)")[0].astype(float)
# [45.99, 12.50, NaN]

# ── ADDRESS PARSING ───────────────────────────────────────
addresses = pd.Series(["123 MG Road, Bengaluru, Karnataka 560001",
                        "456 Anna Salai, Chennai, Tamil Nadu 600002"])
df_addr = addresses.str.extract(
    r"(?P<street>.+),\s*(?P<city>[^,]+),\s*(?P<state>[^,]+)\s+(?P<pincode>\d{6})"
)
print(df_addr)
#         street        city        state  pincode
# 0  123 MG Road  Bengaluru  Karnataka   560001
# 1  456 Anna Salai   Chennai  Tamil Nadu  600002

# ── DATE FORMAT NORMALISATION ─────────────────────────────
dates = pd.Series(["15/06/2024", "2024-06-15", "June 15, 2024", "15-Jun-24"])
# Attempt to parse with multiple strategies
parsed_dates = pd.to_datetime(dates, infer_datetime_format=True, errors="coerce")
# For truly mixed formats, handle case by case
```

---

## 25. Error Handling and Writing Clean Code

### try / except — full patterns

```python
# ── BASIC ─────────────────────────────────────────────────
try:
    result = 10 / 0
except ZeroDivisionError:
    result = None
    print("Cannot divide by zero")

# ── MULTIPLE EXCEPTION TYPES ──────────────────────────────
try:
    value = int("abc")
except (ValueError, TypeError) as e:
    print(f"Conversion failed: {type(e).__name__}: {e}")

# ── ELSE AND FINALLY ──────────────────────────────────────
try:
    df = pd.read_csv("data.csv")
except FileNotFoundError:
    df = pd.DataFrame()
    print("File not found — using empty DataFrame")
except pd.errors.EmptyDataError:
    df = pd.DataFrame()
    print("File is empty")
except Exception as e:
    # Catch-all (use sparingly — can hide bugs)
    print(f"Unexpected error: {type(e).__name__}: {e}")
    raise   # re-raise after logging
else:
    print(f"Loaded {len(df):,} rows")    # runs ONLY if no exception
finally:
    print("Finished loading attempt")    # ALWAYS runs

# ── SAFE LOADING PATTERN ──────────────────────────────────
from pathlib import Path

def safe_read_csv(path: str, **kwargs) -> pd.DataFrame:
    """Load CSV safely. Returns empty DataFrame if file missing or empty."""
    p = Path(path)
    if not p.exists():
        print(f"⚠️  File not found: {path}")
        return pd.DataFrame()
    if p.stat().st_size == 0:
        print(f"⚠️  Empty file: {path}")
        return pd.DataFrame()
    try:
        df = pd.read_csv(path, **kwargs)
        print(f"✓  {p.name}: {len(df):,} rows × {df.shape[1]} cols")
        return df
    except Exception as e:
        print(f"✗  Failed to read {path}: {e}")
        return pd.DataFrame()
```

---

### Writing clean, reusable code

```python
# ── TYPE HINTS — document what functions expect and return ──
from typing import Optional, List, Dict, Tuple

def compute_rfm(
    df: pd.DataFrame,
    customer_col: str,
    date_col: str,
    value_col: str,
    n_segments: int = 5,
    reference_date: Optional[pd.Timestamp] = None,
) -> pd.DataFrame:
    """
    Compute RFM (Recency-Frequency-Monetary) scores.

    Parameters
    ----------
    df             : Transaction-level DataFrame. One row = one transaction.
    customer_col   : Column name for customer identifier.
    date_col       : Column name for transaction date.
    value_col      : Column name for transaction value.
    n_segments     : Number of score bins (default 5 = quintiles).
    reference_date : 'Today' for recency calculation. Defaults to max date in df.

    Returns
    -------
    DataFrame with one row per customer and columns:
        customer_col, recency_days, frequency, monetary, r_score, f_score, m_score, segment

    Examples
    --------
    >>> rfm = compute_rfm(orders, "customer_id", "order_date", "revenue")
    >>> rfm.groupby("segment")["monetary"].mean()
    """
    if reference_date is None:
        reference_date = df[date_col].max()

    rfm = df.groupby(customer_col).agg(
        recency_days = (date_col,  lambda x: (reference_date - x.max()).days),
        frequency    = (date_col,  "count"),
        monetary     = (value_col, "sum"),
    ).reset_index()

    labels = list(range(1, n_segments + 1))
    rfm["r_score"] = pd.qcut(rfm["recency_days"],
                              n_segments, labels=labels[::-1], duplicates="drop")
    rfm["f_score"] = pd.qcut(rfm["frequency"].rank(method="first"),
                              n_segments, labels=labels, duplicates="drop")
    rfm["m_score"] = pd.qcut(rfm["monetary"],
                              n_segments, labels=labels, duplicates="drop")

    for col in ["r_score", "f_score", "m_score"]:
        rfm[col] = rfm[col].astype(float).astype(int)

    conditions = [
        (rfm["r_score"] >= 4) & (rfm["f_score"] >= 4),
        (rfm["r_score"] >= 3) & (rfm["f_score"] >= 3),
        (rfm["r_score"] >= 4) & (rfm["f_score"] <= 2),
        (rfm["r_score"] <= 2) & (rfm["f_score"] >= 3),
    ]
    rfm["segment"] = np.select(conditions,
                                ["Champions","Loyal","New","At Risk"],
                                default="Others")
    return rfm

# ── METHOD CHAINING — readable pipeline ───────────────────
# Instead of:
df1 = df.copy()
df1["month"] = df1["date"].dt.to_period("M")
df1 = df1[df1["status"] == "delivered"]
df1 = df1.groupby("month")["revenue"].sum().reset_index()

# Use:
result = (
    df
    .copy()
    .assign(month=lambda d: d["date"].dt.to_period("M"))
    .query("status == 'delivered'")
    .groupby("month")["revenue"]
    .sum()
    .reset_index()
    .rename(columns={"revenue": "monthly_revenue"})
)
```

---

## Appendix A: SQL-to-Python Mapping

```
╔═══════════════════════════════════════════════════════════════════╗
║              SQL ↔ PANDAS REFERENCE                               ║
╠═══════════════════════════════════════════════════════════════════╣
║  SQL CLAUSE               PANDAS EQUIVALENT                       ║
║  ─────────────────────────────────────────────────────            ║
║  SELECT col               df["col"]                               ║
║  SELECT col1, col2        df[["col1", "col2"]]                    ║
║  SELECT *                 df                                      ║
║  SELECT DISTINCT col      df["col"].unique()                      ║
║                           df.drop_duplicates()                    ║
║  FROM table               pd.read_csv("table.csv")                ║
║                           pd.read_sql("SELECT * FROM table", conn)║
║                                                                   ║
║  WHERE col > val          df[df["col"] > val]                     ║
║  WHERE col IN (a,b)       df[df["col"].isin([a, b])]              ║
║  WHERE col BETWEEN a AND b df[df["col"].between(a, b)]            ║
║  WHERE col LIKE '%x%'     df[df["col"].str.contains("x")]         ║
║  WHERE col IS NULL        df[df["col"].isna()]                    ║
║  WHERE col IS NOT NULL    df[df["col"].notna()]                   ║
║                                                                   ║
║  GROUP BY col             df.groupby("col")                       ║
║  GROUP BY col1, col2      df.groupby(["col1", "col2"])            ║
║  HAVING COUNT(*) > 5      .filter(lambda g: len(g) > 5)          ║
║                                                                   ║
║  ORDER BY col DESC        df.sort_values("col", ascending=False)  ║
║  LIMIT 10                 df.head(10)                             ║
║  OFFSET 20                df.iloc[20:]                            ║
║                                                                   ║
║  COUNT(*)                 len(df) or df["col"].count()            ║
║  COUNT(DISTINCT col)      df["col"].nunique()                     ║
║  SUM(col)                 df["col"].sum()                         ║
║  AVG(col)                 df["col"].mean()                        ║
║  MIN(col)                 df["col"].min()                         ║
║  MAX(col)                 df["col"].max()                         ║
║                                                                   ║
║  INNER JOIN               pd.merge(how="inner")                   ║
║  LEFT JOIN                pd.merge(how="left")                    ║
║  RIGHT JOIN               pd.merge(how="right")                   ║
║  FULL OUTER JOIN          pd.merge(how="outer")                   ║
║  UNION ALL                pd.concat([df1, df2])                   ║
║                                                                   ║
║  CASE WHEN c THEN a END   np.where(c, a, b)                       ║
║  CASE WHEN (multi)        np.select([c1,c2], [a1,a2], default=d)  ║
║                                                                   ║
║  LAG(col, 1)              df["col"].shift(1)                      ║
║  LEAD(col, 1)             df["col"].shift(-1)                     ║
║  ROW_NUMBER() OVER (...)  .groupby().cumcount() + 1               ║
║  NTILE(5)                 pd.qcut(q=5)                            ║
║  SUM() OVER (ORDER BY x)  df["col"].cumsum() or .expanding().sum()║
║  AVG() OVER (7 PRECEDING) df["col"].rolling(7).mean()             ║
║  RANK() OVER ()           df["col"].rank(method="dense")          ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## Appendix B: pip Install Guide

```bash
# ── CORE DATA STACK ───────────────────────────────────────
pip install pandas numpy matplotlib seaborn

# ── DATABASE ──────────────────────────────────────────────
pip install sqlalchemy
pip install psycopg2-binary     # PostgreSQL
pip install pymysql             # MySQL

# ── FILE FORMATS ──────────────────────────────────────────
pip install openpyxl            # Excel read/write (pd.read_excel, to_excel)
pip install xlrd                # Old .xls format (use openpyxl for .xlsx)
pip install pyarrow             # Parquet support
pip install fastparquet         # Alternative Parquet engine

# ── STATISTICS ────────────────────────────────────────────
pip install scipy               # Statistical tests, distributions
pip install statsmodels         # Regression, time-series, statistical models

# ── MACHINE LEARNING (next level) ─────────────────────────
pip install scikit-learn

# ── FASTER PANDAS ALTERNATIVE ─────────────────────────────
pip install polars              # 10-100× faster than pandas for large data

# ── ALL CORE AT ONCE ──────────────────────────────────────
pip install pandas numpy matplotlib seaborn sqlalchemy openpyxl pyarrow scipy

# ── CHECK AND MANAGE ──────────────────────────────────────
pip list                        # all installed packages
pip show pandas                 # details about one package
pip install --upgrade pandas    # upgrade to latest
pip freeze > requirements.txt   # export current environment
pip install -r requirements.txt # install from requirements file
```

---

## Appendix C: Common Errors and Fixes

| Error | Root Cause | Fix |
|-------|-----------|-----|
| `KeyError: 'column_name'` | Column doesn't exist | Check `df.columns`; strip spaces: `df.columns.str.strip()` |
| `ValueError: cannot convert float NaN to integer` | NaN in column before `astype(int)` | `df["col"].fillna(0).astype(int)` |
| `SettingWithCopyWarning` | Modifying a slice of a DataFrame | Add `.copy()`: `subset = df[mask].copy()` |
| `TypeError: unsupported operand` | Mixed types in arithmetic | Check `df.dtypes`; cast to matching type |
| `ParserError` in `read_csv` | Wrong delimiter or encoding | Try `sep="\t"` or `encoding="latin-1"` |
| `MemoryError` | File too large for RAM | Use `chunksize=`, `usecols=`, or Parquet format |
| `AttributeError: 'Series' has no attribute 'str'` | Column is not string type | `df["col"].astype(str).str.upper()` |
| `IndexError: single positional indexer is out-of-bounds` | `iloc` index ≥ len(df) | Check `len(df)` before accessing by position |
| Merge creates more rows than expected | 1:N relationship — right table has duplicates | `df2.drop_duplicates(subset=["key"])` before merge |
| `DtypeWarning` in `read_csv` | Mixed types in a column | Add `dtype={"col": str}` or `low_memory=False` |
| `OutOfBoundsDatetime` | Date string has year out of range (e.g. "00/01/01") | `errors="coerce"` in `pd.to_datetime()` |
| `UnicodeDecodeError` in `read_csv` | File has non-UTF-8 characters | Try `encoding="latin-1"` or `encoding="cp1252"` |
| Groupby gives unexpected NaN in results | There are NaN values in the groupby key | `df.dropna(subset=["group_col"])` before groupby |
| `ValueError: Cannot set a frame with no columns` | Filtering with wrong syntax | Use `&` and `|` (not `and`/`or`); wrap each condition in `()` |

---

*End of Python 101 — v2 Expanded Edition*
*Chat #2 · June 2026*
