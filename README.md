# Bloqade Rydberg Blockade Performance Demo

This repository contains a compact neutral-atom simulation workflow using QuEra's **Bloqade Analog** Python interface. The demo studies a two-atom Rydberg blockade primitive and connects atom spacing, Rydberg interaction strength, local analog emulation, final-shot measurement probabilities, and double-excitation suppression.

The goal is not to provide a production-level neutral-atom performance model. The goal is to demonstrate a transparent performance-modeling workflow:

```text
atom geometry → Rydberg interaction scale → Bloqade Analog emulation → measurement probabilities → blockade metric
```

The simulator is intentionally small so that the physics assumptions, Bloqade program construction, probability extraction, finite-shot interpretation, and performance interpretation remain easy to inspect.

---

## Repository Contents

```text
bloqade-rydberg-blockade-performance-demo/
├── README.md
├── requirements.txt
├── sim_bloqade-rydberg-blockade-performance-demo.ipynb
└── bloqade-rydberg-blockade-performance-demo.pdf
```

No generated figures are committed to the repository. The notebook generates plots when run.

---

## Files

### `sim_bloqade-rydberg-blockade-performance-demo.ipynb`

Jupyter notebook implementing the Bloqade Analog simulator. It is organized in three cells:

1. **Tools** — imports, physical constants, Bloqade program construction, interaction-scale conversion, count extraction, probability mapping, spacing sweep, selected-row extraction, and summary printing.
2. **User-facing controls** — an `ipywidgets` control panel for the spacing window, number of sampled distances, selected performance point, and number of shots. The Rabi amplitude, detuning, and pulse duration remain fixed in the current version to keep the physics model focused.
3. **Run, graphics, and summary** — a run button executes the Bloqade sweep, prints raw counts at the selected spacing, maps counts to state probabilities, generates the three-panel plot, prints a compact numerical summary, and displays the resulting dataframe.

### `bloqade-rydberg-blockade-performance-demo.pdf`

Technical brief describing the motivation, physical model, Bloqade workflow, mathematical details, interpretation, limitations, and planned extensions.

### `requirements.txt`

Python dependencies for running the notebook, including `ipywidgets` for the interactive controls.

---

## Scientific Idea

The demo considers two neutral atoms separated by distance \(R\). Each atom can be in a ground state \(|g\rangle\) or a Rydberg state \(|r\rangle\). The four two-atom basis states are:

```text
|gg>, |gr>, |rg>, |rr>
```

A uniform Rabi drive attempts to excite the atoms. When the atoms are close together, the Rydberg-Rydberg interaction shifts the doubly excited state \(|rr\rangle\) out of resonance. This suppresses simultaneous excitation, a phenomenon known as **Rydberg blockade**.

For blockade-based neutral-atom computing, the useful behavior is controlled access to the ground and single-excitation manifolds. Population in the double-excited state \(|rr\rangle\) is generally an unwanted leakage channel. Therefore, the notebook tracks:

```text
P(rr)
```

as a compact diagnostic for double-excitation leakage.

A simple blockade suppression metric is:

```text
1 - P(rr)
```

The notebook also reports:

```text
P(single) = P(gr) + P(rg)
```

because a successful fixed-duration pulse can transfer most final-shot probability into the single-excitation manifold while keeping \(P(rr)\) small.

---

## Interaction Scale

The Rydberg interaction is modeled as:

```text
V(R) = C6 / R^6
```

The notebook uses the rubidium Rydberg interaction scale commonly used in Bloqade/Aquila-style examples:

```text
C6 = 2π × 862690 rad · μm^6 / μs
```

The notebook reports the interaction as:

```text
V / 2π   in MHz
```

This makes the interaction scale directly comparable to Rabi-control frequency scales.

---

## Bloqade Analog Workflow

The simulator uses the Bloqade Analog API:

```python
from bloqade.analog import start
```

The core Bloqade program construction is:

```python
program = (
    start
    .add_position([(0.0, 0.0), (R_um, 0.0)])
    .rydberg
    .rabi.amplitude.uniform.constant(rabi_amplitude, duration)
    .detuning.uniform.constant(detuning, duration)
)

batch = program.bloqade.python().run(shots=shots)
report = batch.report()
```

The notebook uses `report.counts()` to extract sampled final-state outcomes.

---

## Measurement Convention

The notebook uses the Bloqade post-sequence convention:

```text
1 → g  ground / not-Rydberg
0 → r  Rydberg excitation
```

For two atoms, this gives:

```text
11 → gg
10 → gr
01 → rg
00 → rr
```

The final probabilities are estimated from sampled counts.

Because the notebook uses finite-shot sampling, observing zero `rr` outcomes should be interpreted as:

```text
P(rr) is below the current shot resolution
```

not as a mathematical proof that \(P(rr)=0\). Increasing the number of shots or extending the spacing range can reveal rare double-excitation events.

---

## Interactive Controls

The notebook now includes an `ipywidgets` control panel with:

```text
R min
R max
number of sampled distances
selected spacing / performance point
number of shots
```

The controls change the sampling and inspection window, not the underlying physical model. In the current version, the following physics parameters remain fixed in the notebook:

```text
rabi_amplitude = 2.0
detuning       = 0.0
duration       = 1.0
```

A run button executes the sweep after the controls are set. This avoids rerunning the Bloqade emulator on every slider movement.

The interactive layout is useful for validation. For example, a small spacing range may show \(P(rr)\approx 0\) everywhere because the atoms remain deep in the blockade regime. Extending `R max` allows the notebook to leave that regime, at which point \(P(rr)\) becomes nonzero and grows with distance. This demonstrates that suppressed \(P(rr)\) is a physical operating-window result, not a hard-coded zero.

---

## Expected Output

Running the notebook generates:

1. a plot of double-excitation probability `P(rr)` versus atom spacing,
2. a plot of the Rydberg interaction scale `V/2π` in MHz versus atom spacing,
3. a final-state probability distribution at the selected performance point,
4. raw Bloqade counts at the selected spacing,
5. mapped probabilities at the selected spacing,
6. a compact two-column numerical summary,
7. the resulting dataframe.

The selected spacing is also marked with a vertical guide line on the first two plots. This connects the spacing sweep to the final-state distribution shown in the third plot.

The core interpretation is:

```text
smaller atom spacing → stronger Rydberg interaction → stronger blockade → suppressed P(rr)
larger atom spacing  → weaker Rydberg interaction  → blockade leakage can become visible
```

For a well-timed fixed pulse in the blockade regime, the final distribution can be dominated by:

```text
P(gr) + P(rg)
```

with small \(P(gg)\) and suppressed \(P(rr)\). This indicates that the pulse ends near a collective single-excitation point.

---

## Installation

A clean conda environment is recommended.

```bash
conda create -n quera-bloqade python=3.11 -y
conda activate quera-bloqade

python -m pip install --upgrade pip
pip install -r requirements.txt
```

Register the environment as a Jupyter kernel:

```bash
python -m pip install ipykernel
python -m ipykernel install --user \
  --name quera-bloqade \
  --display-name "Python (quera-bloqade)"
```

Start JupyterLab:

```bash
jupyter lab
```

Then open:

```text
sim_bloqade-rydberg-blockade-performance-demo.ipynb
```

and select the kernel:

```text
Python (quera-bloqade)
```

### Widget note

`ipywidgets` must be installed in the same Python environment used by the notebook kernel. If the notebook reports:

```text
ModuleNotFoundError: No module named 'ipywidgets'
```

install widgets into the active environment:

```bash
conda activate quera-bloqade
python -m pip install ipywidgets
```

Then restart the notebook kernel.

---

## Portfolio Context

This repository is a compact portfolio artifact for neutral-atom performance modeling. It demonstrates the workflow pattern:

```text
hardware parameter → physical model → local emulation → probability-level output → performance interpretation
```

It is designed to show fluency with:

- Bloqade Analog programming,
- neutral-atom geometry,
- Rydberg interaction scaling,
- blockade physics,
- probability extraction from local emulation,
- finite-shot interpretation,
- interactive notebook design,
- compact performance-relevant metrics.

---

## Limitations

The current version is intentionally minimal. It does not yet include:

- atom loss,
- imperfect filling,
- readout error,
- laser noise,
- phase noise,
- finite-temperature effects,
- geometry jitter,
- calibration uncertainty,
- comparison with hardware data,
- runtime or time-to-solution estimation,
- time-resolved population dynamics.

The reported probabilities are final-shot measurement probabilities after a fixed pulse. They should not be interpreted as continuous-time populations.

---

## Planned Extensions

Natural next steps include:

1. adding finite-shot uncertainty bands,
2. adding spacing disorder or geometry jitter,
3. adding imperfect filling or atom-loss models,
4. exposing Rabi amplitude, detuning, and pulse duration as optional advanced controls,
5. comparing pulse-duration dependence,
6. adding time-resolved population diagnostics,
7. defining a threshold-based blockade success metric,
8. extending from two atoms to short chains or small graphs,
9. recording model assumptions and run metadata in a benchmark-style report.

A particularly useful next step is a geometry-jitter study:

```text
R → R + δR
```

followed by repeated emulation or resampling to estimate the distribution of `P(rr)` under geometry uncertainty.

---

## License

MIT License

Copyright (c) 2026 Boris Kiefer

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

This material was developed and/or adapted with support from the National
Science Foundation through the QCAP-NQVL-Pilot and QCAP-NQVL-Design efforts
under NSF Award Nos. OSI-2410813 and OSI-2531569.
Any opinions, findings, conclusions, or recommendations expressed in this
material are those of the author(s) and do not necessarily reflect the views
of the National Science Foundation.
---

## Author

**Dr. Boris Kiefer**  
New Mexico State University  
GitHub: [boriskiefer](https://github.com/boriskiefer)
