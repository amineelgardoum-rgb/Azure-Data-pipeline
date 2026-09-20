# Azure-Data-pipeline — Repository Overview

**Repo:** [amineelgardoum-rgb/Azure-Data-pipeline](https://github.com/amineelgardoum-rgb/Azure-Data-pipeline)
**Type:** Azure Data Factory (ADF) Git-integrated project (ARM/JSON export, not application code)
**Factory name:** `tokyo-datapipeline-data-factory` (region: `germanywestcentral`)

> The repo has no real README content beyond a title, so this file summarizes what the JSON definitions actually configure — the standard folder layout ADF creates when a Data Factory instance is connected to a GitHub repo for source control (`factory/`, `linkedService/`, `dataset/`, `pipeline/`, `publish_config.json`).

## What the pipeline does

This is a **Tokyo Olympics data ingestion pipeline**. It copies five raw CSV files from a public GitHub dataset into an Azure Data Lake Storage Gen2 container, using ADF Copy activities. The source data comes from [darshilparmar/tokyo-olympic-azure-data-engineering-project](https://github.com/darshilparmar/tokyo-olympic-azure-data-engineering-project) — this looks like a common Azure/ADF learning project based on that tutorial dataset.

**Flow (`pipeline/pipeline1.json`):** five `Copy` activities run in a chained sequence (each depends on the previous one completing):

```
Atheletes → Coaches → Medals → EntriesGender → Teams
```

Each activity:
- **Source:** `DelimitedTextSource` reading over HTTP (`GET`) from a public raw-GitHub CSV URL
- **Sink:** `DelimitedTextSink` writing to Azure Data Lake Storage Gen2 (`AzureBlobFSWriteSettings`), quoting all text, `.txt`-flagged file extension setting
- **Destination:** ADLS filesystem `tokyo-olympic-data`, folder `raw-data`

## Linked Services (`linkedService/`)

| Linked Service | Type | Target |
|---|---|---|
| `AtheletesHTTP` | HttpServer (anonymous) | `.../data/Athletes.csv` |
| `CoachesHTTP` | HttpServer (anonymous) | `.../data/Coaches.csv` |
| `MedalsHTTP` | HttpServer (anonymous) | `.../data/Medals.csv` |
| `TeamsHTTP` | HttpServer (anonymous) | `.../data/Teams.csv` |
| `EntriesGender` | HttpServer (anonymous) | `.../data/EntriesGender.csv` |
| `AzureDataLakeStorage1` | AzureBlobFS | `https://tokyoolympicdataazure.dfs.core.windows.net/` |

All five HTTP sources point at raw files under:
`https://raw.githubusercontent.com/darshilparmar/tokyo-olympic-azure-data-engineering-project/main/data/`

## Datasets (`dataset/`)

Five source/sink pairs, one per CSV:

| Source dataset (HTTP) | Sink dataset (ADLS) | Output path |
|---|---|---|
| `Atheletes` | `AtheletesSinkADLS` | `tokyo-olympic-data/raw-data/Atheletes.csv` |
| `Coaches` | `CoachesADSLSink` | `tokyo-olympic-data/raw-data/Coaches.csv` |
| `Medals` | `MedalsSink` | `tokyo-olympic-data/raw-data/Medals.csv` |
| `Teams` | `TeamsADSLsink` | `tokyo-olympic-data/raw-data/Teams.csv` |
| `EntriesGender` | `EntriesGenderADLSsink` | `tokyo-olympic-data/raw-data/EntriesGender.csv` |

## Factory & publish config

- `factory/tokyo-datapipeline-data-factory.json` — factory resource definition with a system-assigned managed identity, deployed to `germanywestcentral`.
- `publish_config.json` — `{"publishBranch":"main","enableGitComment":true}`, i.e. the ADF "publish" (ARM template export) branch is `main` with Git commit messages enabled.

## Notes / things worth flagging

- **Naming typos** are consistent throughout ("Atheletes" instead of "Athletes", "ADSL" instead of "ADLS") — harmless but worth knowing if you're searching the resource names.
- **`AzureDataLakeStorage1.json` contains an `encryptedCredential` field.** It's a base64-encoded ADF-internal credential reference (tied to that specific Data Factory's managed identity), not a raw secret/key — but treat any such field as sensitive and avoid reusing it outside the original factory.
- This is purely an **ingestion/landing-zone pipeline** (HTTP → raw ADLS folder). There's no transformation, Data Flow, or downstream (e.g. Synapse/Databricks) step defined in this repo — it appears to be the first stage of a larger tutorial-style pipeline.
