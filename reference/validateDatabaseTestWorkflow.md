# Validate a static CI matrix against a declaration

Validate a static CI matrix against a declaration

## Usage

``` r
validateDatabaseTestWorkflow(config, workflow, ci)
```

## Arguments

- config:

  A database test configuration.

- workflow:

  Path to a GitHub Actions YAML workflow or an already parsed workflow
  mapping.

- ci:

  CI tier represented by the static matrix.

## Value

`TRUE`, invisibly, or a `panoptes_workflow_drift` error.
