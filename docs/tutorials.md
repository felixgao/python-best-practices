# Tutorials


## Performance Optimization & Tuning

The following are tutorials regarding how to performance tune and optimize 
critical portions of code.

- [Profiling Tools & Strategy](tutorials/performance/profiling_tools.md): A 
  general guide for how to profile python code.
- [Lookup Tables](tutorials/performance/lookup_tables.md): Performance of 
  different lookup table implementations for both  single and batch requests.
- [Array Multiprocessing](tutorials/performance/array_multiprocessing.md): How
  to efficiently use all machine resources to process [pd.Series], 
  [pd.DataFrame] and [np.ndarray].


## Packaging

These tutorials describe how to create sharable libraries which can be used 
across projects.

- [Structure](tutorials/packaging/structure.md): How to structure a package 
  using [setuptools].
- [Code Style](tutorials/packaging/code_style.md): Recommended code style tools
  and, practices, and reference.
- [Logging](tutorials/packaging/logging.md): How to configure package-level 
  logging.
- [API Design](tutorials/packaging/api_design.md): High level design principles
  for package python APIs (and how it differs from Java).
- [Testing](tutorials/packaging/testing.md): How to test a python package so
  it is easy to test for all developers and CICD tools.


[setuptools]: https://setuptools.readthedocs.io/en/latest/
[pd.Series]: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html
[pd.Dataframe]: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html
[np.ndarray]: https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.html