# SQLChains Prototype
## 1. Overview

This repository contains the executable prototype of SQLChains, the corresponding dataset, and specific methods for reproducing the experimental results in our VLDB submission.

## 2. Directory Structure

The prototype requires the following fixed directory structure (already configured in the provided ZIP package). Do NOT modify the structure to ensure successful execution:

      SQLChains-VLDB/

      ├── lib/                 # Dependencies (no action required)

      ├── chains.db            # 1GB dataset/metadata file

      ├── README.md            # This documentation

      └── SQLChains.jar        # Executable prototype (JDK8 compiled)

## 3. Environment Requirements

JDK Version: JDK 8

## 4. Quick Run

Follow these steps to run the prototype. No additional configuration or data preparation is needed.

### Step 1: Extract the ZIP Package

Extract the provided `SQLChains-VLDB.zip` file to any directory (e.g., `D:\SQLChains-VLDB` on Windows).

### Step 2: Run the Prototype

Navigate to the extracted directory and run:

```bash
cd D:\SQLChains-VLDB
java -Xmx4g -Xms2g -jar SQLChains.jar
```

> **Note:** If you prefer to run from a different directory, make sure `chains.db` is in your current working directory and provide the full path to the JAR file:
> ```bash
> java -Xmx4g -Xms2g -jar /path/to/SQLChains-VLDB/SQLChains.jar
> ```

### Step 3: Verify Execution

A successful startup shows the following prompt:
```bash
Starting SQLChains prototype...
CFP>
```

Type `help` to view all supported commands (detailed below).


## 5. Core Commands

All commands are case-insensitive. You can use `list` to view query chain definitions or output the full SQL statements, and `exec` to execute a query chain or subchain. Execution supports both **step-by-step** mode and **pipelined** mode.

### 5.1 list [chain_name] [options]

List query chain details or SQL statements for specified chains. Usage examples are provided below.

      list             #List all query chains
      list tpc-c1      #List each query in the tpc-c1 chain
      list tpc-c1-v1   #List each query in the tpc-c1-v1 variant chain
      list tpc-c1.q14  #List full nested SQL of tpc-c1 (ending at q14)

#### Optional Parameters

- `--size=N`: Specify dataset scale (default: 1; e.g., --size=3 for 3x scale)

- `--ClickHouse`: Output ClickHouse-compatible SQL (default: DuckDB)

- `--nosort`: Remove ORDER BY clauses (except for the input query)

- `--cte`: Output SQL in CTE format (default: nested SQL for better performance)

### 5.2 exec chain_name [options]

Execute a full or sub query chain and output execution cost for key steps. Usage examples are provided below.

      exec tpc-c1                      #Execute the full tpc-c1 chain
      exec tpc-c1.q8                   #Execute the tpc-c1 sub-chain ending at q8
      exec tpc-c1 --size=3 --pipe      #Execute tpc-c1 on 3x scale dataset with pipeline execution
      exec tpc-c1 --limit=20           #Execute tpc-c1 and output top 20 results

#### Optional Parameters

- `--pipe`: Enable pipeline execution

- `--size=N`: Execute on Nx scale dataset (default: 1)

- `--nosort`: Execute without sorting

- `--limit=N`: Output top N results (default: 10)


## 6. Reproducibility

To fully reproduce the experimental results in our paper, follow the steps below.

### 6.1 Prerequisites

Before running the experiments, install the following:
- **DuckDB**       — Our experiments were conducted on Windows 11 using **DuckDB 1.2**
- **ClickHouse**   — We used the official Docker image of **ClickHouse 24.8** on Windows 11
> The experiments in this paper were performed on **Windows 11** with the specific versions noted above. Other versions may work but have not been tested.

### 6.2 Obtaining SQL Statements for DuckDB and ClickHouse

Use `list` in SQLChains to generate SQL for DuckDB and ClickHouse, e.g., `list tpc-c1.v14`. For ClickHouse, the `--ClickHouse` parameter is only needed for S-C1 queries; other queries do not require it.

SQLChains can output SQL in either **nested** or **CTE** format. In our experiments, we observed that **nested SQL performs better** than CTE on both DuckDB and ClickHouse. Therefore, all experiments in this paper use the **nested SQL** format.

### 6.3 Execution and Measurement

1. Execute each query **5 times**.
2. Take the **median value** as the final execution cost.

> **Note for DuckDB:** During our experiments, we observed that DuckDB's performance may degrade slightly with continuous execution. To ensure fair and consistent measurements, we recommend exiting DuckDB after each query and restarting the process for the next query. 

### 6.4 Run SQLChains Prototype (Part 1)

Follow the steps in the previous sections to launch the SQLChains prototype. Then use the `exec` command with appropriate parameters to execute the specified query chains.

### 6.5 Run Experiments with DuckDB (Part 2)
1. Launch DuckDB and open the dataset:
   ```sql
   .open chains.db
   .timer on
   PRAGMA threads=1;
2. Paste the nested SQL obtained from SQLChains and execute.

### 6.6 Run Experiments with ClickHouse (Part 3)
#### Create Data Tables
Use DuckDB to export three tables (`lineitem`, `stocks`, `retail`) from `chains.db` to CSV files, then import them into ClickHouse:

```sql
CREATE TABLE default.lineitem ENGINE = MergeTree ORDER BY tuple() AS
SELECT * FROM file('lineitem.csv', CSVWithNames);

CREATE TABLE default.retail ENGINE = MergeTree ORDER BY tuple() AS
SELECT * FROM file('retail.csv', CSVWithNames);

CREATE TABLE default.stocks ENGINE = MergeTree ORDER BY tuple() AS
SELECT Open, Close, High, Low, Volume, Amount, Price,
       toString(Code) as Code, Name, Date,
       toTime(toDateTime(concat(Date, ' ', Time))) as Time
FROM file('stocks.csv', CSVWithNames);
```

#### Execute Queries
Set single thread and execute the nested SQL obtained from SQLChains:
```sql
SET max_threads=1;
[Paste the nested SQL here]
```

## 7. Query Chains and Datasets

#### Basic Query Chains

- **TPC-C1** (ends with Q14)
- **TPC-C2** (ends with Q9)
- **S-C1** (ends with Q12)
- **S-C2** (ends with Q14)
- **OR-C1** (ends with Q10)
- **OR-C2** (ends with Q9)

#### Query Chain Variants

- TPC-C2-V1, TPC-C2-V2, TPC-C2-V3
- S-C1-V2, S-C1-V3
- S-C2-V4
- OR-C1-V3

#### Datasets

- **TPC-H**: scale 1x and 3x
- **Stocks**: scale 1x and 3x
- **Online Retail**: scale 1x

## 8. Acknowledgments

We thank the contributors of the following datasets used in this paper:

- **TPC-H**: Transaction Processing Performance Council (TPC)
- **Online Retail**: UCI Machine Learning Repository

The **Stocks** dataset was compiled by the authors.

## 9. Contact

For any issues related to prototype execution or reproducibility, please contact the corresponding author of the paper.

---

Last Updated: VLDB Submission Period 2026

