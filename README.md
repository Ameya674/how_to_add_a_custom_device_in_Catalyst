# How to add a Custom Device in Catalyst

```mermaid
graph TD
  A[Start] --> B[Process]
  B --> C[End]
<!-- 
```mermaid
graph TD
  A[catalyst/] --> A1[CITATION.cff]
  A --> A2[LICENSE]
  A --> A3[MANIFEST.in]
  A --> A4[Makefile]
  A --> A5[README.md]
  A --> A6[benchmark/]
  A --> A7[bin/]
  A --> A8[demos/]
  A --> A9[doc/]
  A --> A10[frontend/]
  A --> A11[mlir/]
  A --> A12[pyproject.toml]
  A --> A13[requirements.txt]
  A --> A14[runtime/]
  A --> A15[setup.py]
  A --> A16[setup_dev_from_wheel.sh]
  A --> A17[standalone_plugin_wheel/]

  A14 --> B1[CMakeLists.txt]
  A14 --> B2[Makefile]
  A14 --> B3[README.rst]
  A14 --> B4[build/]
  A14 --> B5[include/]
  A14 --> B6[tests/]
  A14 --> B7[lib/]

  B7 --> C1[CMakeLists.txt]
  B7 --> C2[OQDcapi/]
  B7 --> C3[backend/]

  C3 --> D1[CMakeLists.txt]
  C3 --> D2[common/]
  C3 --> D3[null_qubit/]
  C3 --> D4[openqasm/]
  C3 --> D5[oqd/]
  C3 --> D6[custom_device/]

  D6 --> E1[CustomDevice.cpp]
  D6 --> E2[CustomDevice.hpp]
  D6 --> E3[CMakeLists.txt]
  D6 --> E4[custom_device.toml]

  %% Highlight custom_device branch
  class D6,E1,E2,E3,E4 custom;

  classDef custom fill=#ffebcc,stroke=#ff9900,stroke-width=2;

``` -->
