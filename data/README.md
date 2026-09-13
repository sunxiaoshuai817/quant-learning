# Data Guide

The repository does not redistribute full commercial market datasets. This keeps the project lightweight and respects the terms of third-party data providers.

## Directory convention

```text
data/
├── sample/       # Small redistributable or synthetic datasets committed to Git
├── raw/          # Full local datasets ignored by Git
└── processed/    # Optional derived datasets; commit only small, licensed files
```

## Reproduction modes

- **Demo mode:** use a small dataset under `data/sample/` to verify the complete research pipeline.
- **Full mode:** obtain data with your own Tushare or JoinQuant account and store it under `data/raw/` or the project-specific local data directory.

Before committing any dataset, confirm that its license permits public redistribution. API tokens and account credentials must be supplied through environment variables and must never be committed.

