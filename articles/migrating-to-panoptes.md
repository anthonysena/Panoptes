# Migrating copied database helpers to Panoptes

## Migration boundary

The CohortGenerator prototype mixed reusable infrastructure with
package-owned behavior. Migration moves declaration parsing, selection,
environment policy, adapters, context assembly, resource lifecycle, and
CI drift detection to Panoptes. It keeps cohort fixtures, generation
calls, assertions, cohort tables, and cleanup in CohortGenerator.

## Helper mapping

| Copied helper concept | Panoptes replacement |
|----|----|
| `getDatabaseTestConfig()` | [`readDatabaseTestConfig()`](https://ohdsi.github.io/Panoptes/reference/readDatabaseTestConfig.md) |
| [`getSelectedTestDbms()`](https://ohdsi.github.io/Panoptes/reference/getSelectedTestDbms.md) | `getSelectedTestDbms(config)` |
| [`getDatabasePlatformConfig()`](https://ohdsi.github.io/Panoptes/reference/getDatabasePlatformConfig.md) | `getDatabasePlatformConfig(config)` |
| per-DBMS environment functions | [`resolveDatabaseTestEnvironment()`](https://ohdsi.github.io/Panoptes/reference/resolveDatabaseTestEnvironment.md) plus aliases |
| [`validateDatabaseTestEnvironment()`](https://ohdsi.github.io/Panoptes/reference/validateDatabaseTestEnvironment.md) | same public name with required/optional policy |
| per-DBMS connection functions | internal Panoptes adapters |
| `getDatabaseTestContext()` | [`createDatabaseTestContext()`](https://ohdsi.github.io/Panoptes/reference/createDatabaseTestContext.md) |
| `getPlatformConnectionDetails()` | [`withDatabaseTestContext()`](https://ohdsi.github.io/Panoptes/reference/withDatabaseTestContext.md) |
| manual unique cohort table suffix | `context$objectPrefix` or [`createUniqueDatabaseObjectName()`](https://ohdsi.github.io/Panoptes/reference/createUniqueDatabaseObjectName.md) |
| global SqlRender option assignment | scoped automatically by [`withDatabaseTestContext()`](https://ohdsi.github.io/Panoptes/reference/withDatabaseTestContext.md) |
| BigQuery temporary file | scoped automatically by [`withDatabaseTestContext()`](https://ohdsi.github.io/Panoptes/reference/withDatabaseTestContext.md) |
| hard-coded workflow matrix | [`getDatabaseTestMatrix()`](https://ohdsi.github.io/Panoptes/reference/getDatabaseTestMatrix.md) and [`validateDatabaseTestWorkflow()`](https://ohdsi.github.io/Panoptes/reference/validateDatabaseTestWorkflow.md) |

The generic tests from `test-database-platforms.R` move to Panoptes.
Keep a small consumer integration test proving that the installed
package declaration is found and that the package behavior test receives
the expected context.

## Update the declaration

The prototype’s `environmentPrefix` was descriptive and could not
represent exceptions such as SQL Server’s repeated `SERVER`, Snowflake’s
`CDM53_SCHEMA`, or BigQuery’s prefix. Panoptes rejects that field.
Remove it and map existing secrets to normalized `HADES_*` variables in
the workflow, or pass explicit aliases while migrating.

Add these executable fields to every platform:

- `runner`: the GitHub runner label;
- `os`: `any`, `linux`, `windows`, or `macos`; and
- `githubEnvironment`: the protected GitHub environment name.

The template puts BigQuery on `windows-latest`, resolving the
prototype’s Windows-only helper versus Ubuntu-only matrix conflict.

## Replace orchestration

Replace this ordering from the prototype:

``` text
construct context -> validate environment -> download drivers -> set option
```

with:

``` r

config <- Panoptes::readDatabaseTestConfig(package = "CohortGenerator")
Panoptes::skipIfNoLiveDatabase(config)

Panoptes::withDatabaseTestContext(
  config,
  legacyAliases = cohortGeneratorAliases,
  verify = TRUE,
  {
    # Existing CohortGenerator behavior and on.exit() table cleanup.
  }
)
```

Panoptes validates before it calls DatabaseConnector and does not
download JDBC drivers. Driver installation belongs in CI setup or a
developer’s configured driver folder, avoiding network activity during
context validation.

## Update CI

Map existing secrets to the normalized variables inside each matrix job.
Split the current all-platform pull-request matrix into the declared
tiers:

- PostgreSQL, SQL Server, and Oracle on `pull-request`;
- Redshift, Spark, BigQuery, Snowflake, and IRIS on `scheduled`;
- BigQuery on its declared Windows runner.

Add workflow-validation tests for each static matrix. Keep secrets out
of the ordinary check workflow. Pin Panoptes to a released tag before
broad adoption.

## Remove copied code only after the integration passes

Once the package behavior test passes through Panoptes:

1.  remove copied configuration, environment, and adapter helpers;
2.  remove generic helper unit tests now covered by Panoptes;
3.  retain the consumer behavior test and cleanup;
4.  retain a declaration-discovery integration test; and
5.  run at least one required and one scheduled external platform before
    the migration is considered complete.
