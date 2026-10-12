[![Day 2 CI](https://github.com/weaver010/day2-ci-assignment/actions/workflows/ci.yml/badge.svg)](https://github.com/weaver010/day2-ci-assignment/actions/workflows/ci.yml)

# day2-ci-assignment

A tiny Python temperature-conversion library used to practise building a complete GitHub Actions CI workflow.

## Usage

```python
from temperature import celsius_to_fahrenheit
celsius_to_fahrenheit(100)  # 212.0
```

## Run tests locally

```bash
pip install -r requirements-dev.txt
pytest -v
```

See [PIPELINE.md](PIPELINE.md) for the pipeline design.
