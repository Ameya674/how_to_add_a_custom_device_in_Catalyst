# How to add a custom device in Catalyst (Pennylane's JIT compiler)

graph TD

  A[root:/catalyst]

  subgraph Catalyst Root
    A1[CITATION.cff]
    A2[LICENSE]
    A3[MANIFEST.in]
    A4[Makefile]
    A5[README.md]
    A6[pyproject.toml]
    A7[requirements.txt]
    A8[setup.py]
    A9[setup_dev_from_wheel.sh]
    A10[bin/]
    A11[benchmark/]
    A12[demos/]
    A13[doc/]
    A14[frontend/]
    A15[mlir/]
    A16[runtime/]
    A17[standalone_plugin_wheel/]
  end

  subgraph runtime/
    R1[CMakeLists.txt]
    R2[Makefile]
    R3[README.rst]
    R4[build/]
    R5[include/]
    R6[tests/]
    R7[lib/]
  end

  subgraph lib/
    L1[CMakeLists.txt]
    L2[OQDcapi/]
    L3[backend/]
    L4[capi/]
    L5[registry/]
  end

  subgraph backend/
    B1[CMakeLists.txt]
    B2[common/]
    B3[null_qubit/]
    B4[openqasm/]
    B5[oqd/]
    B6[custom_device/]
  end

  subgraph custom_device/
    CD1[CustomDevice.cpp]
    CD2[CustomDevice.hpp]
    CD3[CMakeLists.txt]
    CD4[custom_device.toml]
  end

  %% Relationships
  A --> A16
  A16 --> R7
  R7 --> L3
  L3 --> B6
  B6 --> CD1
  B6 --> CD2
  B6 --> CD3
  B6 --> CD4

  %% Styling
  classDef custom fill:#ffebcc,stroke:#ff9900,stroke-width:2px;
  class B6,CD1,CD2,CD3,CD4 custom;

