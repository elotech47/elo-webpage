---
title: "QSS Integrator: Python Wrapper for Stiff Chemical Kinetics"
date: 2025-11-11
draft: false
tags: ["Python", "C++", "Numerical Methods", "Combustion", "Scientific Computing"]
categories: ["Research", "Software Development"]
description: "A Python package wrapping the Quasi-Steady State method for efficient integration of stiff ODE systems in combustion chemistry"
---

## Overview

During my PhD research in computational combustion, I frequently encountered the challenge of integrating stiff chemical kinetics systems. While the Quasi-Steady State (QSS) method existed in various C++ implementations, there wasn't a readily accessible Python package that could interface with modern combustion libraries like Cantera.

I developed [qss-integrator](https://pypi.org/project/qss-integrator/), a Python wrapper around a C++ implementation of the QSS method, to bridge this gap. The package is available on PyPI and has become a core component of my research on reinforcement learning-based adaptive solver selection.

## The Quasi-Steady State Method

The QSS method is designed for stiff ODE systems commonly found in chemical kinetics. Unlike traditional methods that treat all species uniformly, QSS exploits timescale separation by identifying fast (quasi-steady) and slow species automatically.

The method splits ODEs into production and destruction terms:
```
dy/dt = q(y) - d(y)
```
where q represents production rates and d represents destruction rates. This formulation allows larger timesteps while maintaining accuracy, particularly effective for systems with extreme stiffness ratios.

The algorithm is based on the work by Mott, Oran, & van Leer (2000): *A Quasi-Steady-State Solver for the Stiff Ordinary Differential Equations of Reaction Kinetics*.

## Implementation

### Core Architecture

The package consists of:
- **C++ Backend**: Core QSS integration algorithm (`qss_integrator.cpp/h`)
- **Python Bindings**: pybind11 interface (`qss_python.cpp`)
- **Utility Layer**: Helper functions for Cantera integration (`utils.py`)

```python
from qss_integrator import QssIntegrator, PyQssOde

# Define ODE splitting
def chemistry_ode(t, y, corrector=False):
    # Split into production and destruction terms
    q = compute_production_rates(y)
    d = compute_destruction_rates(y)
    return q, d

# Configure integrator
integrator = QssIntegrator()
ode = PyQssOde(chemistry_ode)
integrator.setOde(ode)
integrator.initialize(n_species)

# Set parameters
integrator.epsmin = 1e-2
integrator.epsmax = 20.0
integrator.dtmin = 1e-15
integrator.dtmax = 1e-6

# Integrate
integrator.setState(y0, t0=0.0)
integrator.integrateToTime(tf)
```

### Build System

The package uses modern Python packaging tools:
- **setuptools + pybind11** for building C++ extensions
- **cibuildwheel** for cross-platform wheel generation
- **GitHub Actions** for automated testing and deployment

Supports:
- Python 3.8-3.12
- Linux (x86_64), macOS (x86_64, arm64), Windows (x86_64)
- Automatic PyPI publishing on release

## Application in My Research

I use this package extensively in my PhD work on reinforcement learning for adaptive solver selection in combustion simulations. The RL agent learns to switch between CVODE (a traditional stiff solver) and QSS based on system characteristics.

## Technical Details

### Configuration Parameters

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `epsmin` | 1e-2 | Accuracy parameter for timestep selection |
| `epsmax` | 20.0 | Correction tolerance threshold |
| `dtmin` | 1e-15 | Minimum allowed timestep |
| `dtmax` | 1e-6 | Maximum allowed timestep |
| `itermax` | 2 | Number of corrector iterations |
| `abstol` | 1e-8 | Absolute tolerance for convergence |

### Performance Characteristics

The QSS method performs best when:
- System exhibits strong stiffness (timescale separation > 10³)
- Fast species are identifiable (high destruction/production ratio)
- Accuracy requirements are moderate (1e-4 to 1e-8)

Less effective for:
- Mildly stiff systems (timescale separation < 10²)
- Systems requiring very tight tolerances (< 1e-10)

## Installation and Usage

```bash
# Install from PyPI
pip install qss-integrator

# With optional dependencies for combustion examples
pip install qss-integrator[examples]
```

Example with Cantera:
```python
import cantera as ct
from qss_integrator.utils import create_qss_solver

gas = ct.Solution('gri30.yaml')
gas.TPX = 1000, ct.one_atm, 'CH4:1, O2:2, N2:7.52'

config = {
    'epsmin': 1e-2,
    'epsmax': 10.0,
    'dtmin': 1e-15,
    'dtmax': 1e-6
}

solver = create_qss_solver(gas, ct.one_atm, config)
y0 = [gas.T] + list(gas.Y)
solver.setState(y0, 0.0)
solver.integrateToTime(1e-3)  # 1 ms
```

## Links

- **PyPI**: [qss-integrator](https://pypi.org/project/qss-integrator/)
- **GitHub**: [elotech47/pyQSS](https://github.com/elotech47/pyQSS)
- **Documentation**: [README](https://github.com/elotech47/pyQSS/blob/main/README.md)

## Technical Stack

- **Languages**: C++17, Python 3.8+
- **Core Libraries**: pybind11, NumPy
- **Optional Dependencies**: Cantera (for combustion applications)
- **Build Tools**: setuptools, cibuildwheel
- **CI/CD**: GitHub Actions with multi-platform testing

## Citation

```bibtex
@software{qss_integrator,
  title={QSS Integrator: Quasi-Steady State Method for Stiff ODE Systems},
  author={Ikponmwoba, Eloghosa},
  year={2024},
  url={https://github.com/elotech47/pyQSS}
}
```

---

*Developed as part of PhD research at Louisiana State University, focusing on reinforcement learning applications for computational combustion.*

