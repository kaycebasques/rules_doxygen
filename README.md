# Doxygen rules for Bazel

[![CI](https://github.com/TendTo/rules_doxygen/actions/workflows/ci.yml/badge.svg)](https://github.com/TendTo/rules_doxygen/actions/workflows/ci.yml)

This repository contains a Starlark implementation of Doxygen rules in Bazel.

## Setup as a module dependency (bzlmod)

Add the following to your _MODULE.bazel_:

```bzl
bazel_dep(name = "rules_doxygen", version = "1.1.0", dev_dependency = True)
```

If you don't want to depend on the [Bazel package registry](https://bazel.build/external/bazelbuild/rules_pkg) or you want to use a not-yet-published version of this module, you can use an archive override by adding the following lines below the `bazel_dep` rule in your _MODULE.bazel_ file:

```bzl
bazel_dep(name = "rules_doxygen", version = "1.1.0", dev_dependency = True)
archive_override(
    module_name = "rules_doxygen",
    urls = "https://github.com/TendTo/rules_doxygen/archive/refs/heads/main.tar.gz",
    strip_prefix = "rules_doxygen-main",
    # The SHA256 checksum of the archive file, based on the rules' version, for reproducibility
    # integrity = "sha256-0SCaZuAerluoDs6HXMb0Bj9FttZVieM4+Dpd9gnMM+o=", # Example
)
```

### Doxygen version selection

To select a doxygen version to use, use the `doxygen_extension` module extension below the `bazel_dep` rule in your MODULE.bazel file.

```bzl
doxygen_extension = use_extension("@rules_doxygen//:extensions.bzl", "doxygen_extension")
use_repo(doxygen_extension, "doxygen")
```

By default, version `1.11.0` of Doxygen is used. To select a different version, indicate it in the `version` module:

```bzl
doxygen_extension = use_extension("@rules_doxygen//:extensions.bzl", "doxygen_extension")
# Using the 1.10.0 version of Doxygen on Windows instead of the default 1.11.0
doxygen_extension.version(version = "1.10.0", sha256 = "2135c1d5bdd6e067b3d0c40a4daac5d63d0fee1b3f4d6ef1e4f092db0d632d5b")
use_repo(doxygen_extension, "doxygen")
```

If you don't know the SHA256 of the Doxygen binary, just leave it empty.
The build will fail with an error message containing the correct SHA256.

```bash
Download from https://github.com/doxygen/doxygen/releases/download/Release_1_10_0/doxygen-1.10.0.windows.x64.bin.zip failed: class com.google.devtools.build.lib.bazel.repository.downloader.UnrecoverableHttpException Checksum was 2135c1d5bdd6e067b3d0c40a4daac5d63d0fee1b3f4d6ef1e4f092db0d632d5b but wanted 0000000000000000000000000000000000000000000000000000000000000000
```

## Use

The main macro exposed by this module is `doxygen`.
It generates a Doxygen documentation target from a list of sources.
Only the sources are required, the rest of the parameters are optional.

```bzl
# My BUILD.bazel file
doxygen(
    name = "doxygen",   # Name of the rule, can be anything
    srcs = glob([       # List of sources to document.
        "*.h",          # Usually includes the source files and the markdown files.
        "*.cpp",
    ]) + ["README.md"],
    project_brief = "Example project for doxygen",  # Brief description of the project
    project_name = "base",                          # Name of the project
    configurations = [                              # Additional configurations to add to the Doxyfile
        "GENERATE_HTML = YES",                      # They are the same as the Doxyfile options,
        "GENERATE_LATEX = NO",                      # and will override the default values
        "USE_MDFILE_AS_MAINPAGE = README.md",
    ]
)
```

Use the [Doxygen documentation](https://www.doxygen.nl/manual/config.html) or generate a brand new _Doxyfile_ with `doxygen -g` to see all the available options to put in the `configurations` list.
They will simply be appended at the end of the file, overriding the default values.

> [!Note]
> See the [examples](examples) directory for examples of how to use the rules.

## Build

To build the documentation, run the following command:

```bash
bazel build //path/to:doxygen_target
```

For example, if the _BUILD.bazel_ file is in the root of the repository, and the target is named `doxygen`

```bzl
# BUILD.bazel file in the root of the repository
doxygen(
    name = "doxygen",
    srcs = glob([
        "*.h",
        "*.cpp",
    ]),
    project_name = "base",
)
```

the build command would be:

```bash
bazel build //:doxygen
```

The generated documentation will be in the `bazel-bin/<subpackage>` directory.
If the _BUILD.bazel_ file was in the root of the repository, the `<subpackage>` would be an empty string.

The documentation can be viewed by opening the `index.html` file in a browser using any web server:

```bash
# Using Python 3
cd bazel-bin/<subpackage>/html
python3 -m http.server 8000
```

Lastly, you may want to compress the documentation to share it with others:

```bash
tar -czvf doxygen.tar.gz bazel-bin/<subpackage>/html
```

## TODO

- [ ] Add support for macos (I cant't be bothered :D)
- [ ] Add more easy-to-use common configuration for the Doxyfile
