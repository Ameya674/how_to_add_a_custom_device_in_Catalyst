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
    B2[custom_device/]
  end

  subgraph custom_device/
    CD1[CustomDevice.cpp]
    CD2[CustomDevice.hpp]
    CD3[CMakeLists.txt]
    CD4[custom_device.toml]
  end

  %% Relationships
  A --> A1
  A1 --> R1
  A1 --> R2
  A1 --> R3
  R3 --> L1
  R3 --> L3
  L3 --> B1
  L3 --> B2
  B2 --> CD1
  B2 --> CD2
  B2 --> CD3
  B2 --> CD4

  %% Unified yellow style for specific files only
  classDef yellow fill:#ffff99,stroke:#333,stroke-width:2px,color:#000;

  class R2,B1,CD3 yellow;

```

Add files the files and folders colored in yellow, edit if they already exist. 

- CustomDevice.hpp: Header file for the custom device. Contains all the function signatures. 
- CustomDevice.cpp: Contains the implementations for all the functions.
- custom_device.toml: Configuration file for Pennylane to use the device, defines what gates and measurements are supported.
- CMakeLists.txt: To build the new device.

## Basic boilerplate code

#### CustomDevice.hpp
```
#pragma once
#include <QuantumDevice.hpp>
#include <vector>
#include <string>
#include <optional>

namespace Catalyst::Runtime::Devices {

struct CustomDevice final : public QuantumDevice {
    explicit CustomDevice(const std::string &kwargs);

    auto AllocateQubit() -> QubitIdType override;
    auto AllocateQubits(size_t num_qubits) -> std::vector<QubitIdType> override;
    void ReleaseQubit(QubitIdType) override;
    void ReleaseAllQubits() override;

    auto GetNumQubits() const -> size_t override;
    void SetDeviceShots(size_t shots) override;
    auto GetDeviceShots() const -> size_t override;

    void NamedOperation(const std::string &name,
                        const std::vector<double> &params,
                        const std::vector<QubitIdType> &wires,
                        bool inverse,
                        const std::vector<QubitIdType> &ctrl_wires,
                        const std::vector<bool> &ctrl_values) override;

    auto Measure(QubitIdType wire, std::optional<int32_t> postselect) -> Result override;

    void StartTapeRecording() override;
    void StopTapeRecording() override;

    void State(DataView<std::complex<double>, 1> &state) override;


};

} // namespace Catalyst::Runtime::Devices

```

#### CustomDevice.cpp
```
#include "CustomDevice.hpp"
#include <complex>
#include <vector>
#include <string>
#include <optional>
#include <iostream>

namespace Catalyst::Runtime::Devices {

CustomDevice::CustomDevice([[maybe_unused]] const std::string &kwargs) {
    std::cout << "Constructor: CustomDevice" << std::endl;
}

auto CustomDevice::AllocateQubit() -> QubitIdType {
    std::cout << "Called: AllocateQubit" << std::endl;
    return 0;
}

auto CustomDevice::AllocateQubits(size_t num_qubits) -> std::vector<QubitIdType> {
    std::cout << "Called: AllocateQubits" << std::endl;
    return std::vector<QubitIdType>(num_qubits, 0);
}

void CustomDevice::ReleaseQubit(QubitIdType) {
    std::cout << "Called: ReleaseQubit" << std::endl;
}

void CustomDevice::ReleaseAllQubits() {
    std::cout << "Called: ReleaseAllQubits" << std::endl;
}

auto CustomDevice::GetNumQubits() const -> size_t {
    std::cout << "Called: GetNumQubits" << std::endl;
    return 1; // Assume 1 qubit for simplicity
}

void CustomDevice::SetDeviceShots(size_t shots) {
    std::cout << "Called: SetDeviceShots with shots: " << shots << std::endl;
}

auto CustomDevice::GetDeviceShots() const -> size_t {
    std::cout << "Called: GetDeviceShots" << std::endl;
    return 1;
}

void CustomDevice::NamedOperation(const std::string &name,
                                 const std::vector<double> &params,
                                 const std::vector<QubitIdType> &wires,
                                 bool inverse,
                                 const std::vector<QubitIdType> &ctrl_wires,
                                 const std::vector<bool> &ctrl_values) {
    std::cout << "Called: NamedOperation with name: " << name << ", wires: ";
    for (const auto &wire : wires) {
        std::cout << wire << " ";
    }
    std::cout << ", inverse: " << inverse << std::endl;

    if (name == "Hadamard" && wires.size() == 1 && params.empty() && ctrl_wires.empty()) {
        std::cout << "Applying Hadamard gate on wire " << wires[0] << std::endl;
        // Simulate Hadamard application (e.g., update a state vector)
        // For demo, just log; in a real backend, apply the gate
    } else {
        std::cerr << "Unsupported operation: " << name << std::endl;
        throw std::runtime_error("Unsupported operation: " + name);
    }
}

auto CustomDevice::Measure(QubitIdType wire, std::optional<int32_t> postselect) -> Result {
    std::cout << "Called: Measure on wire " << wire << std::endl;
    bool *result = new bool(true); // Dummy result for |0>
    return result;
}

void CustomDevice::StartTapeRecording() {
    std::cout << "Called: StartTapeRecording" << std::endl;
}

void CustomDevice::StopTapeRecording() {
    std::cout << "Called: StopTapeRecording" << std::endl;
}

void CustomDevice::State(DataView<std::complex<double>, 1> &state) {
    std::cout << "Called: State" << std::endl;
}

} // namespace Catalyst::Runtime::Devices

GENERATE_DEVICE_FACTORY(CustomDevice, Catalyst::Runtime::Devices::CustomDevice)
```
#### custom_device.toml

```
schema = 3

[operators.gates]
Hadamard = { properties = ["invertible"] }  # Support Hadamard

[operators.observables]
PauliX = { }
PauliY = { }
PauliZ = { }
Hamiltonian = { conditions = ["terms-commute"] }
Sum = { conditions = ["terms-commute"] }
SProd = { }
Prod = { }

[measurement_processes]
ExpectationMP = { }
SampleMP = { }
CountsMP = { conditions = ["finiteshots"] }
StateMP = { conditions = ["analytic"] }

[compilation]
qjit_compatible = false
runtime_code_generation = false
dynamic_qubit_management = false
overlapping_observables = true
non_commuting_observables = false
initial_state_prep = false
supported_mcm_methods = []
```

## How to build?

### Add/edit lines in green

#### /catalyst/runtime/lib/backend/custom_device/CMakeLists.txt
```
cmake_minimum_required(VERSION 3.20)

project(rtd_custom_device LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(rtd_custom_device SHARED CustomDevice.cpp)

target_include_directories(rtd_custom_device
    PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    PRIVATE
    ${runtime_includes}
    ${backend_utils_includes}
)

set_property(TARGET rtd_custom_device PROPERTY POSITION_INDEPENDENT_CODE ON)
```

#### /catalyst/runtime/lib/backend/CMakeLists.txt

```diff
 add_subdirectory(null_qubit)
 configure_file(null_qubit/null_qubit.toml null_qubit.toml)

+add_subdirectory(custom_device)
+configure_file(custom_device/custom_device.toml custom_device.toml)

 if(ENABLE_OQD)
 add_subdirectory(oqd)
 configure_file(oqd/oqd.toml oqd.toml)
 endif()

 if(ENABLE_OPENQASM)
 add_subdirectory(openqasm)
 configure_file(openqasm/braket_local_qubit.toml braket_local_qubit.toml)
 configure_file(openqasm/braket_aws_qubit.toml braket_aws_qubit.toml)
 endif()
```

#### /catalyst/runtime/Makefile

```diff
PYTHON ?= $(shell which python3)
PYTHON_PREFIX := $(shell $(PYTHON) -c "import sys; print(sys.prefix)")
PYTHON_VERSION := $(shell $(PYTHON) -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")
C_COMPILER ?= $(shell which clang)
CXX_COMPILER ?= $(shell which clang++)
COMPILER_LAUNCHER ?= $(shell which ccache)
NPROC ?= $(shell python3 -c "import os; print(os.cpu_count())")

MK_ABSPATH := $(abspath $(lastword $(MAKEFILE_LIST)))
MK_DIR := $(dir $(MK_ABSPATH))
RT_BUILD_DIR ?= $(MK_DIR)/build
CODE_COVERAGE ?= OFF
BUILD_TYPE ?= RelWithDebInfo
ENABLE_OPENQASM ?= ON
ENABLE_OQD ?= OFF
ENABLE_ASAN ?= OFF
STRICT_WARNINGS ?= ON
LLVM_DIR ?= $(MK_DIR)/../mlir/llvm-project/

PLATFORM := $(shell uname -s)

# Since we build executables for all test targets in this file, there is no reason to preload.
# Preloading while the ASAN RT is already embedded in the executable can lead to the "incompatible
# ASAN runtimes detected" error, at least on Linux (macOS seemed okay with it).
# If necessary, add ASAN flags here to fine-tune behaviour (e.g. ASAN_OPTIONS="...").
ASAN_COMMAND :=

# Note: ASAN replaces dlopen calls, which means that when we open other libraries via dlopen that
#       relied on the parent's library's RPATH, these libraries are no longer found.
#         e.g. `dlopen(openqasm_python_module.so)` from OpenQASMRunner.hpp
#       We can fix this using LD_LIBRARY_PATH (for dev builds).
ifeq ($(PLATFORM), Linux)
ASAN_FLAGS = LD_LIBRARY_PATH="$(RT_BUILD_DIR)/lib:$(LD_LIBRARY_PATH)"
# Python tests can generally produce leaks, here observed on Linux. OpenQASM tests call into Python.
PY_ASAN_OPTIONS := ASAN_OPTIONS="detect_leaks=0"
endif

# Deferred assignment since RT_BUILD_DIR can be modified by the targets.
ifeq ($(ENABLE_ASAN), ON)
ASAN_COMMAND = $(ASAN_FLAGS)
endif

+BUILD_TARGETS := rt_capi rtd_null_qubit rtd_custom_device
TEST_TARGETS := runner_tests_qir_runtime runner_tests_mbqc_runtime

ifeq ($(ENABLE_OPENQASM), ON)
	BUILD_TARGETS += rtd_openqasm
	TEST_TARGETS += runner_tests_openqasm
endif

ifeq ($(ENABLE_OQD), ON)
	BUILD_TARGETS += rt_OQD_capi rtd_oqd_device
	TEST_TARGETS += runner_tests_oqd
endif

.PHONY: help
help:
	@echo "Please use \`make <target>' where <target> is one of"
	@echo "  all                to build Catalyst Runtime"
	@echo "  coverage           to generate a coverage report using lcov"
	@echo "  clean              to delete all temporary, cache, and build files"
	@echo "  test               to run the Catalyst runtime test suite"
	@echo "  format [check=1]   to apply C++ formatter; use with 'check=1' to check instead of modify (requires clang-format)"
	@echo "  format [version=?] to apply C++ formatter; use with 'version={version}' to run clang-format-{version} instead of clang-format"
	@echo "  check-tidy         to build Catalyst Runtime with RUNTIME_CLANG_TIDY=ON (requires clang-tidy)"

.PHONY: configure
configure:
	@echo "Configure Catalyst Runtime"

	cmake -G Ninja -B $(RT_BUILD_DIR) . \
		-DCMAKE_BUILD_TYPE=$(BUILD_TYPE) \
		-DCMAKE_LIBRARY_OUTPUT_DIRECTORY=$(RT_BUILD_DIR)/lib \
		-DCMAKE_C_COMPILER=$(C_COMPILER) \
		-DMLIR_INCLUDE_DIRS=$(LLVM_DIR)/mlir/include \
		-DCMAKE_CXX_COMPILER=$(CXX_COMPILER) \
		-DCMAKE_C_COMPILER_LAUNCHER=$(COMPILER_LAUNCHER) \
		-DCMAKE_CXX_COMPILER_LAUNCHER=$(COMPILER_LAUNCHER) \
		-DENABLE_OPENQASM=$(ENABLE_OPENQASM) \
		-DENABLE_OQD=$(ENABLE_OQD) \
		-DENABLE_CODE_COVERAGE=$(CODE_COVERAGE) \
		-DPython_EXECUTABLE=$(PYTHON) \
		-DENABLE_ADDRESS_SANITIZER=$(ENABLE_ASAN) \
		-DRUNTIME_ENABLE_WARNINGS=$(STRICT_WARNINGS)

.PHONY: runtime
runtime: configure
	cmake --build $(RT_BUILD_DIR) --target $(BUILD_TARGETS) -j$(NPROC)

.PHONY: test_runner
test_runner: configure
	cmake --build $(RT_BUILD_DIR) --target $(TEST_TARGETS) -j$(NPROC)

.PHONY: test
test: CODE_COVERAGE=OFF
test: BUILD_TYPE?=RelWithDebInfo
test: test_runner
	@echo "Catalyst runtime test suite - NullQubit"
	$(ASAN_COMMAND) $(RT_BUILD_DIR)/tests/runner_tests_qir_runtime
	@echo "Catalyst MBQC runtime test suite"
	$(ASAN_COMMAND) $(RT_BUILD_DIR)/tests/runner_tests_mbqc_runtime
ifeq ($(ENABLE_OPENQASM), ON)
	# Test the OpenQasm devices C++ tests
	$(ASAN_COMMAND) $(PY_ASAN_OPTIONS) $(RT_BUILD_DIR)/tests/runner_tests_openqasm
endif
ifeq ($(ENABLE_OQD), ON)
	# Test the OQD devices C++ tests
	$(ASAN_COMMAND) $(RT_BUILD_DIR)/tests/runner_tests_oqd
endif

.PHONY: coverage
coverage: RT_BUILD_DIR := $(RT_BUILD_DIR)_cov
coverage: CODE_COVERAGE=ON
coverage: BUILD_TYPE=Debug
coverage: C_COMPILER=$(shell which gcc)
coverage: CXX_COMPILER=$(shell which g++)
coverage: export LLVM_PROFILE_FILE := $(RT_BUILD_DIR)/tests/%m.profraw
coverage: test_runner
	@echo "check C++ code coverage"
	$(ASAN_COMMAND) $(RT_BUILD_DIR)/tests/runner_tests_qir_runtime
	$(ASAN_COMMAND) $(RT_BUILD_DIR)/tests/runner_tests_mbqc_runtime
ifeq ($(ENABLE_OPENQASM), ON)
	$(ASAN_COMMAND) $(PY_ASAN_OPTIONS) $(RT_BUILD_DIR)/tests/runner_tests_openqasm
endif
ifeq ($(ENABLE_OQD), ON)
	$(ASAN_COMMAND) $(RT_BUILD_DIR)/tests/runner_tests_oqd
endif
ifeq ($(PLATFORM),Linux)
	lcov --directory $(RT_BUILD_DIR) -b $(MK_DIR)/lib --capture --output-file $(RT_BUILD_DIR)/coverage.info
	lcov --remove $(RT_BUILD_DIR)/coverage.info '/usr/*' '*/_deps/*' '*/envs/*' '*/mlir/*' --output-file $(RT_BUILD_DIR)/coverage.info -q
	genhtml $(RT_BUILD_DIR)/coverage.info --output-directory $(RT_BUILD_DIR)/cov -t "Catalyst Runtime C++ Coverage" --num-spaces 4  -q
else
	xcrun llvm-profdata merge $(RT_BUILD_DIR)/tests/*.profraw -o $(RT_BUILD_DIR)/tests/rt_test_coverage.profdata
	xcrun llvm-cov show -instr-profile $(RT_BUILD_DIR)/tests/rt_test_coverage.profdata \
		-object $(RT_BUILD_DIR)/tests/runner_tests_openqasm \
		$(RT_BUILD_DIR)/tests/runner_tests_qir_runtime \
		$(RT_BUILD_DIR)/tests/runner_tests_mbqc_runtime \
		-format=html -output-dir=$(RT_BUILD_DIR)/coverage_html \
		$(MK_DIR)/include $(MK_DIR)/lib $(MK_DIR)/tests
endif

.PHONY: clean
clean:
	@echo "clean build files"
	rm -rf $(RT_BUILD_DIR) $(RT_BUILD_DIR)_cov cov coverage.info $(MK_DIR)/BuildTidy

.PHONY: format
format:
ifdef check
	$(PYTHON) ../bin/format.py --check $(if $(version:-=),--cfversion $(version)) .
else
	$(PYTHON) ../bin/format.py $(if $(version:-=),--cfversion $(version)) .
endif

.PHONY: check-tidy
check-tidy:
	@echo "build Catalyst Runtime with RUNTIME_CLANG_TIDY=ON"
	cmake -G Ninja -B $(MK_DIR)/BuildTidy . \
		-DCMAKE_BUILD_TYPE=$(BUILD_TYPE) \
		-DCMAKE_C_COMPILER=$(C_COMPILER) \
		-DCMAKE_CXX_COMPILER=$(CXX_COMPILER) \
		-DRUNTIME_CLANG_TIDY=ON

	cmake --build $(MK_DIR)/BuildTidy --target rt_capi -j$(NPROC)

```

## How to use this new device with Pennylane?

#### Boilerplate code for the circuit (circuit.py)

```
import pennylane as qml
from pennylane import numpy as np
from pennylane import qjit
import pathlib

class CustomDevice(qml.devices.Device):
    config_filepath = pathlib.Path("/catalyst/runtime/lib/backend/custom_device/custom_device.toml")

    @staticmethod
    def get_c_interface():
        return "CustomDevice", "/catalyst/runtime/build/lib/librtd_custom_device.so"

    def __init__(self, shots=None, wires=None):
        super().__init__(wires=wires, shots=shots)

    @property
    def operations(self):
        return {"Hadamard"}  # Declare Hadamard as supported

    @property
    def observables(self):
        return {"State"}  # Support state measurement

    def execute(self, circuits, config):
        # Placeholder: Implement circuit execution
        # This should call the C++ backend to execute the circuit
        return np.array([1.0, 0.0], dtype=np.complex128)  # Dummy state for |0>

@qjit(keep_intermediate=True)
@qml.qnode(CustomDevice(wires=1))
def circuit():
    qml.Hadamard(wires=0)
    qml.Hadamard(wires=0)
    return qml.state()

print(circuit())
```


