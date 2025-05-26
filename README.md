# How to add a custom device in Catalyst (Pennylane's JIT compiler)

```mermaid
graph TD

  A[root:/catalyst]

  subgraph catalyst/
    A1[runtime/]
  end

  subgraph runtime/
    R1[CMakeLists.txt]
    R2[Makefile]
    R3[lib/]
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
  A --> A1
  A1 --> R3
  A1 --> R1
  A1 --> R2
  R3 --> L3
  L3 --> B6
  B6 --> CD1
  B6 --> CD2
  B6 --> CD3
  B6 --> CD4

  %% Styling
  classDef custom fill:#ffebcc,stroke:#ff9900,stroke-width:2px;
  class B6,CD1,CD2,CD3,CD4 custom;

```
