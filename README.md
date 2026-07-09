# AZFP Studio

A single-file, dependency-free browser app for analysing **ASL Environmental Sciences AZFP** (Acoustic Zooplankton Fish Profiler) data — focused on the **vertical distribution of acoustic targets** such as zooplankton layers and diel vertical migration.

It reads raw AZFP binary files directly in the browser, converts recorded counts to calibrated volume backscattering strength (Sv), and provides an interactive echogram, distribution metrics, ancillary-sensor plots, region annotation, and batch export. Everything runs client-side — no server, no build step, no data leaves the machine.

> **Live app:** `https://<USERNAME>.github.io/azfp-studio/`
> Replace `<USERNAME>` with your GitHub username after enabling Pages (see [Deployment](#deployment)).

---

## Features

- **Native AZFP parser** — reads `.01A`–`.06A` binary files (big-endian, `0xFD02` compact-flash records) with the 124-byte header layout used by [echopype](https://github.com/OSOceanAcoustics/echopype). Handles both raw (2-byte) and averaged/summed (4-byte + 1-byte overflow) storage formats, auto-detects the 455 kHz channel, and merges multiple time-sequential files.
- **Multiple frequencies, independent schedules** — each transducer frequency (e.g. 67 kHz and 455 kHz) is treated as its own time series, so a secondary board sampled on a coarser interval is handled natively. Switch between frequencies from the sidebar.
- **Scales to multi-month deployments** — streaming ingest frees each file's bytes right after parsing, per-frequency data is stored as compact `Uint16` count grids, and a default **burst-averaged** load mode collapses each acquisition burst to one profile (an **Every-ping** mode keeps full resolution). The echogram uses level-of-detail rendering, and estimated in-app memory is shown in the header.
- **AZFPLink `.mfawcpl` support** — parses the deployment configuration and sets acquisition defaults (sound speed, pulse length, freshwater absorption) plus a deployment-metadata panel.
- **Calibrated echogram** — Sv via the ASL/Echoview AZFP equation, interactive pan/zoom, four palettes, adjustable dB range, ping averaging, hover readout, per-file boundary labels, and PNG export.
- **Vertical target distribution** — center of mass, inertia, index of aggregation, equivalent area, NASC, depth-layer table, mean-Sv profile, and a diel-vertical-migration (center-of-mass vs time) plot.
- **Ancillary sensors** — temperature (thermistor), tilt X/Y, pressure, and battery decoded from the per-ping ancillary channels.
- **Annotation** — draw and classify regions (zooplankton, fish, surface/ice, bottom, noise) with per-region Sv / NASC / center-of-mass statistics.
- **Project & batch I/O** — save/load a self-contained JSON project (calibration, deployment, view, annotations, and data), and export the Sv grid, center-of-mass/NASC time series, layer table, and a session report as CSV/JSON.
- **Methods tab** — full documentation of the parsing, range/depth geometry, Sv/TS equations, distribution indices, and references.

---

## Quick start

No installation required.

- **Online:** open the GitHub Pages URL above.
- **Offline / local:** download [`index.html`](index.html) and double-click it (or open it in any modern browser). It is fully self-contained.

The app opens on a synthetic demonstration deployment (a freshwater lake, 67 + 455 kHz) so you can explore every feature immediately. To analyse your own data, drag your files onto the drop zone in the left panel.

### Supported inputs

| File | Purpose |
| --- | --- |
| `.01A`–`.06A` | AZFP raw acoustic data (binary). Drop several from one deployment; they merge in time order. |
| `.XML` | Instrument coefficient file (per serial number) — supplies EL, DS, TVR, VTX0, BP for quantitative Sv. |
| `.mfawcpl` | AZFPLink deployment config — sets sound speed, pulse length, geometry, and populates the metadata panel. |
| `.json` | A previously saved AZFP Studio project. |

---

## Calibration notes

Two different files carry different information, and both are optional but recommended:

- The **`.mfawcpl`** describes *how* the data were acquired: sound speed, pulse length, digitization rate, range averaging, ping/burst timing. AZFP Studio uses it to set acquisition-side defaults.
- The **instrument coefficient `.XML`** carries the *transducer calibration* — EL, DS, TVR, VTX0, and the two-way beam pattern (BP). These are required for absolute Sv.

Without the coefficient XML, the app uses editable placeholder values (representative of a 455 kHz AZFP) and clearly flags Sv as approximate. All calibration parameters are editable in the sidebar at any time.

---

## Data-integrity warning

AZFP data files are **binary**. If they are ever moved through a text-mode transfer (some sync tools, email gateways, or copy-paste), every byte above 127 is replaced with the Unicode replacement character. This corruption is **lossy and irreversible** — the acoustic amplitudes cannot be recovered. AZFP Studio detects this and refuses such files with a clear message. Always copy `.01A` files straight from the instrument or SD card in binary mode.

---

## Methods summary

Counts are converted to Sv following the ASL / Echoview AZFP equation:

```
Sv = EL − 2.5/DS + N/(26214·DS) − TVR − 20·log10(VTX0)
   + 20·log10(R) + 2·α·R − 10·log10(c·τ/2) − Ψ
```

where `N` is the recorded count (averaged data is back-converted to `N`), `R` is range, `α` absorption, `τ` pulse duration, `c` sound speed, and `Ψ` the equivalent two-way beam angle. Distribution indices (center of mass, inertia, index of aggregation, equivalent area, NASC) are computed on linear s_v following Woillez et al. (2007) and Bez & Rivoirard. See the app's **Methods** tab for full detail and references.

---

## Deployment

This repository is configured to publish to **GitHub Pages** via GitHub Actions.

1. Create a new GitHub repository (e.g. `azfp-studio`) and push these files to the `main` branch.
2. In the repository, go to **Settings → Pages → Build and deployment** and set **Source** to **GitHub Actions**.
3. Push to `main` (or run the workflow manually). The [`deploy.yml`](.github/workflows/deploy.yml) workflow publishes the site.
4. The app will be live at `https://<USERNAME>.github.io/azfp-studio/`.

A `.nojekyll` file is included so GitHub Pages serves the files as-is without Jekyll processing.

---

## Repository layout

```
azfp-studio/
├── index.html                 # the entire application (open directly or via Pages)
├── README.md
├── LICENSE                     # MIT
├── CITATION.cff
├── .nojekyll
├── .gitignore
└── .github/workflows/deploy.yml
```

---

## References

- ASL Environmental Sciences — AZFP Operator's Manual and processing notes ([aslenv.com](https://aslenv.com/azfp.html)).
- Echoview — *AZFP Sv and TS* algorithm reference ([support.echoview.com](https://support.echoview.com/WebHelp/Reference/Algorithms/Echosounder/ASL/AZFP_Sv_and_TS.htm)).
- OSOceanAcoustics — *echopype* open-source AZFP reader ([github.com/OSOceanAcoustics/echopype](https://github.com/OSOceanAcoustics/echopype)).
- Woillez, M. et al. (2007). Spatial indicators. *ICES Journal of Marine Science*.
- Bez, N. & Rivoirard, J. Indices of aggregation.

---

## License

Released under the [MIT License](LICENSE).
