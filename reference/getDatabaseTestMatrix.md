# Derive a CI matrix from a database test declaration

Derive a CI matrix from a database test declaration

## Usage

``` r
getDatabaseTestMatrix(config, ci)
```

## Arguments

- config:

  A database test configuration.

- ci:

  One CI tier: `pull-request`, `push`, `scheduled`, `manual`, or `none`.

## Value

A data frame suitable for serializing as a GitHub Actions matrix.
