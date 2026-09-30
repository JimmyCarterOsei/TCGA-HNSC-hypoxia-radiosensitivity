# Software versions

Use Python **3.12.3**, as specified by Jimmy Carter Osei and matching the supplied notebook environment information. The four package versions below were supplied by Abdullah for the analysis environment.

| Software | Version | Repository record |
| --- | --- | --- |
| Python | 3.12.3 | `.python-version` and README setup instructions |
| scikit-survival | 0.25.0 | `requirements.txt` |
| lifelines | 0.30.0 | `requirements.txt` |
| pandas | 2.3.3 | `requirements.txt` |
| scikit-learn | 1.7.2 | `requirements.txt` |

Versions for NumPy, SciPy, Matplotlib, openpyxl and Jupyter have not been established from the original environment. They remain unpinned in requirements.txt. These records are a partial environment specification; they do not establish that the complete historical environment has been recovered or that the full workflow has been rerun with these pins.

After installing requirements.txt, run this inside the selected notebook kernel to check its interpreter and the four confirmed packages:

```python
import sys
from importlib.metadata import version

assert sys.version_info[:3] == (3, 12, 3), sys.version
expected = {
    "scikit-survival": "0.25.0",
    "lifelines": "0.30.0",
    "pandas": "2.3.3",
    "scikit-learn": "1.7.2",
}
print("Python", sys.version.split()[0])
for package, required in expected.items():
    installed = version(package)
    assert installed == required, (package, installed, required)
    print(package, installed)
```

For a new rerun, save `python -m pip freeze > environment-rerun.txt` with the run outputs. That file records the rerun environment and should not be described as the original analysis environment.
