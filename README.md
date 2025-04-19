# mp_typecheck
Static Type checking for MicroPython modules and applications 

Use this action to check your micropython code for type errors and other issues.

The action can use either the `mypy` or `pyright` type checkers. The default is `pyright`.
In addition to the type checker you can configure which type stubs to use for the type checking.

Example usage:
```yaml
on:
  push:
  workflow_dispatch:

jobs:
  CI:
    runs-on: ubuntu-latest
    name: MicroPython - CI
    steps:
      - uses: actions/checkout@v4
      - name: Validate code
        uses: josverl/mp_typecheck@latest
        with:
          typechecker: pyright, mypy
          mp-stubs: micropython-rp2-pico_w-stubs

```

To use the same configuration in CI and locally the recommendation is to use the `pyproject.toml` file. 

### Action
Below is an example of a `pyproject.toml` file that can be used to configure the action and local type checker, or your IDE of choise.
```toml
[project]
name = "myprojec"
description = "My MicroPython project"
version = "0.1.0"
classifiers = [
  "Programming Language :: Python :: 3",
  "Programming Language :: Python :: MicroPython",
  "License :: OSI Approved :: MIT License"
]

### Configuration 

[project.optional-dependencies]
# install to folder typings
# uv pip install -r pyproject.toml --extra stubs --target typings 
stubs = [
  "micropython-rp2-pico_w-stubs"
]

# ###################################################################
# pyright options for MicroPython
# ###################################################################
[tool.pyright]
# places to check
include = ["src"]
extraPaths = ["src/lib"]
# how to check
typeCheckingMode = "standard"
pythonPlatform = "Linux"
reportMissingModuleSource = "none"
reportOptionalMemberAccess = "information"
reportOptionalCall = "information"
# what to use 
stubPath = "typings"
typeshedPath = "typings"
# things not to check
ignore = ["**/typings"]
exclude = [
    ".*", 
    "__*", 
    "**/typings"
]

# ###################################################################
# mypy options for MicroPython
# ###################################################################
[tool.mypy]
# places to check
files = "src/*.py"
mypy_path = "src/lib,typings"
# how to check
platform = "linux"
follow_imports = "silent"
follow_imports_for_stubs = true
# what to use
custom_typeshed_dir = "typings" # allow mypy to use micropython-stdlib
# things not to check
exclude = [
    ".*",  
    "__*",  
    "typings[\\/].*", # TOML basic string 
]
```

### Using multiple configurations

Actions allow you to validate your code against the type stubs of multiple ports, boards and versions of MicroPython.
Below is an example of a matrix test against multipl boards and versions.
![image](https://github.com/user-attachments/assets/bb2f0c72-c573-454a-8835-58ef9df72855)

```YAML
on:
  push:
  workflow_dispatch:
jobs:
  many_ports:
    strategy:
      fail-fast: false
      matrix:
        port: [rp2-pico_w, esp32, esp8266, stm32, samd]
        version: ["1.24.1", "1.25.0"]
    runs-on: ubuntu-latest
    name: Check ${{ matrix.port }} - ${{ matrix.version }}
    steps:
      - uses: actions/checkout@v4
      - name: Validate code
        uses: josverl/typecheck_mp@latest
        with:
          mp-stubs: micropython-${{ matrix.port }}-stubs~=${{ matrix.version }}
          typechecker: pyright
``
Note: You will need to validate that type-subs are available for the specified boards and versions. If not the action will fail.
If you need additional type-stubs published, please [raise an issue](https://github.com/Josverl/micropython-stubs/issues) with the details of the board.



