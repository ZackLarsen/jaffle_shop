# jaffle_shop

This is a fork of the jaffle_shop test repo provided by dbt. The purpose of this repo is to use the jaffle_shop example models with the dbt-duckdb extension to experiment with using dbt locally using a DuckDB database instead of a cloud data warehouse.

## Setup

Define your python environment in an environment.yaml file. It must have duckdb and dbt-duckdb installed to work properly. It can look something like this:

```yaml
name: dbt_jaffle_shop
channels:
  - defaults
  - conda-forge
dependencies:
  - python=3.10
  - pip
  - pip:
    - pyarrow
    - polars
    - duckdb
    - dbt-duckdb
    - ipykernel
    - jupyter
    - deltalake
    - seaborn
    - matplotlib
```

Install the DuckDB adapter for dbt and other necessary dependencies by running the following mamba command in your terminal:

```bash
mamba env create -f environment.yaml
```

To update your environment, run the following mamba command in your terminal:

```bash
mamba env update --file environment.yaml --prune
```

To activate the environment, run the following mamba command in your terminal:

```bash
mamba activate dbt_jaffle_shop
```

## Instructions

Once your environment has been defined and you have used mamba (or conda) to create it and install the necessary packages, you can fork the [jaffle_shop repo](https://github.com/dbt-labs/jaffle_shop), clone it locally, and then run dbt commands against it.

```bash
git clone https://github.com/ZackLarsen/jaffle_shop.git
```

Change into the jaffle_shop directory from the command line:
```bash
cd jaffle_shop
```

To create a new dbt project, dbt instructs us to run this command in the terminal, but it seems to fail, so I skipped this step and just created a profiles.yml file manually (see the next step for details on the contents).

```bash
dbt init jaffle_shop
```

Instead of running **dbt init**, I simply created my profiles.yml file using guidance from Josh Wills in his dbt-duckdb GitHub repository and the corresponding page on the dbt docs site.
- https://github.com/jwills/dbt-duckdb
- https://docs.getdbt.com/docs/core/connect-data-platform/duckdb-setup

For example, to use DuckDB, your profiles.yml file should look like this:

```yaml
jaffle_shop:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: '/Users/zacklarsen/Documents/Documents - Zack’s Mac mini/Projects/jaffle_shop/jaffle_shop.duckdb'
      extensions:
        - httpfs
        - parquet
```

Ensure your profile is setup correctly from the command line:

```bash
dbt debug
```

Load the CSVs with the demo data set. This materializes the CSVs as tables in your target schema. Note that a typical dbt project **does not require this step** since dbt assumes your raw data is already in your warehouse.

```bash
dbt seed
```

Run dbt run in your terminal to compile and run your dbt project. This will create a compiled SQL file for your example_model and execute it against your DuckDB database.

```bash
dbt run
```

> **NOTE:** If this steps fails, it might mean that you need to make small changes to the SQL in the models folder to adjust for the flavor of SQL of your target database. Definitely consider this if you are using a community-contributed adapter.

Test the output of the models:

```bash
dbt test
```

Generate documentation for the project:

```bash
dbt docs generate
```

View the documentation for the project:

```bash
dbt docs serve
```

Navigate to your browser and go to http://localhost:8080 to view the documentation. Click on the lower right corner to view the data lineage graph for your project, which shows a DAG of the source tables and transformations.

## Query newly created tables from seeds using DuckDB:

At the terminal, issue these two commands. The first initializes a DuckDB session and the second opens our saved jaffle_shop duckdb database file created when we ran the dbt models.

```bash
duckdb

.open jaffle_shop.duckdb
```

In the duckdb session that opens, you can query the newly created tables by issuing SQL commands.:

```sql
SELECT * FROM customers LIMIT 10;
```

## Resources

- https://github.com/dbt-labs/jaffle_shop
- https://docs.getdbt.com/docs/core/connect-data-platform/duckdb-setup
- https://docs.getdbt.com/docs/core/connect-data-platform/duckdb-setup
- https://www.getdbt.com/analytics-engineering/modular-data-modeling-technique/
- https://github.com/dbt-labs/dbt-learn-demo

## Lineage Graph

![Lineage Graph](/etc/lineage.png)

### Seeds
This repo contains [seeds](https://docs.getdbt.com/docs/building-a-dbt-project/seeds) that includes some (fake) raw data from a fictional app.

The raw data consists of customers, orders, and payments, with the following entity-relationship diagram:

![Jaffle Shop ERD](/etc/jaffle_shop_erd.png)
