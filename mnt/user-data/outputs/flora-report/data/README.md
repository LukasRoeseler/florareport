# Data directory

The FLoRA dataset (`flora.csv`) is **not** committed to this repository.
Instead, it is fetched fresh at build time by the GitHub Actions workflow
from the canonical FLoRA repository.

To work locally, download the latest version manually:

```bash
curl -L -o data/flora.csv \
  https://raw.githubusercontent.com/forrtproject/FLoRA/main/data/flora.csv
```

The workflow uses the `FLORA_CSV_URL` repository variable when set, falling
back to the URL above.
