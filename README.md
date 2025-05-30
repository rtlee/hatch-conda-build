# hatch-conda-build

Conda package builder for [Hatch](https://hatch.pypa.io/latest/). Hatch is modern, extensible Python project manager.

## Usage

Add `hatch-conda-build` within the build-system.requires field in your pyproject.toml file.

```toml
[build-system]
requires = ["hatchling", "hatch-conda-build"]
build-backend = "hatchling.build"
```

Additionally `conda-build` must be in your current path when running a
hatch build.

`hatch-conda-build` uses [grayskull](https://github.com/conda/grayskull) to translate the requirements section of your `pyproject.toml`, which
specifies dependencies on PyPi, to appropriate conda package names.

### Building Conda Package

The [builder plugin](https://hatch.pypa.io/latest/plugins/builder/reference/) defines the `conda` target.

To start build process, run `hatch build -t conda`:

```shell
$ hatch build -t conda
[conda]
...
```

### Installing the local package

The ouptut package is written to the `dist/conda` directory. This directory is a fully-indexed
conda channel and can be used to install the package.

```shell
$ conda install -c ./dist/conda <package-name>
...
```

## Conda-build Options

Additional builder configuration can be set in the following TOML
header.

```toml
[tool.hatch.build.targets.conda]
...
```

Following table contains available customization of builder behavior.

| Option                 | Type      | Default         | Description                                      |
|:-----------------------|:----------|:----------------|:-------------------------------------------------|
| channels               | list[str] | ['conda-forge'] | Channels used for package build and testing      |
| default_numpy_version  | str       | None            | numpy version, otherwise use conda-build default |
| github_action_metadata | bool      | false           | Include extra metadata into recipe from Github runner environment variables|

When `github_action_metadata` is True and the [Github runner environment variables](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables#default-environment-variables) are present, the follow metadata is added to the 
`extra` section of the recipe:

| Variable(s)                          | Field         | Value           |
|:-------------------------------------|:--------------|:----------------|
| GITHUB_SHA                           | `sha`         | `${GITHUB_SHA}` |
| GITHUB_SERVER_URL<br>GITHUB_REPOSITORY | `remote_url`  | `${GITHUB_SERVER_URL}/${GITHUB_REPOSITORY}` |
| GITHUB_RUN_ID                        | `flow_run_id` | `${GITHUB_RUN_ID}` |

### Modifying the generated recipe

You can specify [recipe metadata](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html) keys in following TOML header

```tml
[tool.hatch.build.targets.conda.recipe]
...
```

Recipe metadata is defined in TOML syntax. The metadata is merged using [deepmerge](https://deepmerge.readthedocs.io/en/latest/index.html) with the following strategies

```python
recipe_merger = Merger(
    type_strategies=[
        (dict, ["merge"]),
        (list, ["append"])
    ],
    fallback_strategies=["override"],
    type_conflict_strategies=["override"]
)
```

Most importantly, this will append list values if the key already exists. For example you can use nested key TOML
syntax:

```toml
[tool.hatch.build.targets.conda.recipe]
requirements.run = ["anaconda-anon-usage"]
test.imports = ["my_package"]
```

will *append* the package name `anaconda-anon-usage` to the autogenerated run requirements and inject the `test`
section into the recipe before it is built.


## License

Plugin hatch-conda-build is distributed under the terms of the [MIT](https://spdx.org/licenses/MIT.html) license.
