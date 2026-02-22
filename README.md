# SQLChains Prototype - VLDB Supplemental Material
## 1. Overview

This repository contains the executable prototype ofSQLChains, the corresponding dataset, and specific methods for reproducing the experimental results in our VLDB submission.

## 2. Directory Structure

The prototype requires the following fixed directory structure (already configured in the provided ZIP package). Do NOT modify the structure to ensure successful execution:

SQLChains-VLDB/

├── SQLChains.jar       # Executable prototype (JDK8 compiled)

├── chains.db           # 1GB dataset/metadata file

├── lib/                # Dependencies (no action required)

└── README.md           # This documentation

## 3. Environment Requirements

JDK Version: JDK 8

## 4. Quick Run

Follow these steps to run the prototype. No additional configuration or data preparation is needed.

### Step 1: Extract the ZIP Package

Extract the provided `SQLChains-VLDB.zip` file to any directory (e.g., `D:\SQLChains-VLDB` on Windows).

### Step 2: Run the Prototype

Run the following command from the extracted directory (SQLChains-VLDB):

      cd D:\SQLChains-VLDB
      java -Xmx4g -Xms2g -jar SQLChains.jar

Note: If you're not in the extracted directory, make sure chains.db is in your current working directory and update the jar path accordingly:

      java -Xmx4g -Xms2g -jar /path/to/SQLChains-VLDB/SQLChains.jar

### Step 3: Verify Execution

After launching the prototype, you will see the following prompt (indicating successful startup):

      Starting SQLChains prototype...
      CFP>

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

## 6. Experimental Notes

### SQL Format Selection

Our prototype supports both nested SQL (default) and CTE format (via the `--cte` parameter). All experimental results in the paper use nested SQL, as our evaluations show that CTE format introduces extra materialization overhead in DuckDB and ClickHouse (due to limited CTE inlining support). Nested SQL can be fully optimized into an end-to-end execution plan, ensuring fair performance comparison.

### Query Chains Overview

The prototype includes 13 preloaded query chains, covering 6 TPC-C benchmarks and 7 variants:

- - Benchmark chains: tpc-c1, tpc-c2, tpc-c3, tpc-c4, tpc-c5, tpc-c6

- - Variant chains: tpc-c1.v1, tpc-c2.v1, tpc-c3.v1, tpc-c4.v1, tpc-c5.v1, tpc-c6.v1, tpc-c6.v2

## Reproducibility

To fully reproduce the experimental results in our paper, follow the three parts below:

### Part 1: Run SQLChains Prototype

1. 1. Run the prototype with JDK 8 (1.8.0_200+).

2. 2. Use the `exec` command with default parameters (nested SQL, no --pipe, size=1) for baseline results.

3. 3. For scale experiments, use `--size=3` and `--size=5`.

### Part 2: Run Experiments with DuckDB

Prerequisite: Download DuckDB version 1.2 first.

1. 1. Run SQLChains prototype, execute `list tpc-c1.v14` to get the corresponding nested SQL statements. (The same method applies to other queries)

2. 2. Launch DuckDB and execute the following commands in sequence:
      
`# Open chains.db dataset`
      
`.open chains.db`
      
`# Enable timer to record execution cost`
      
`.timer on`
     
`# Set single thread as experimental benchmark`
      
`PRAGMA threads=1;`
      
`# Paste the nested SQL obtained from SQLChains and execute`
`[Paste the nested SQL here]`

3. 3. Execute each query 5 times and take the median value as the final execution cost.

4. 4. Note: DuckDB's performance may decrease slightly with continuous execution. Our approach is to exit DuckDB after each query execution, and restart the above steps for the next query.

### Part 3: Run Experiments with ClickHouse

Prerequisites: 1. Download ClickHouse version 24.8 first. 2. Create data tables in advance: Use DuckDB to export three tables (lineitem, stocks, retail) from chains.db to corresponding CSV files respectively, then import the CSV files into ClickHouse with the following SQL statements:

-- Create lineitem table
CREATE TABLE default.lineitem
ENGINE = MergeTree
ORDER BY tuple()
AS SELECT * 
FROM file('lineitem.csv', CSVWithNames);

-- Create retail table
CREATE TABLE default.retail
ENGINE = MergeTree
ORDER BY tuple()
AS SELECT * 
FROM file('retail.csv', CSVWithNames);

-- Create stocks table
CREATE TABLE default.stocks
ENGINE = MergeTree
ORDER BY tuple()
AS
SELECT 
    Open,
    Close, 
    High,
    Low,
    Volume,
    Amount,
    Price,
    toString(Code) as Code,
    Name,
    Date,
    toTime(toDateTime(concat(Date, ' ', Time))) as Time
FROM file('stocks.csv', CSVWithNames);

1. 1. Obtain SQL statements: For S-C1, use `list S-C1 --ClickHouse` to generate ClickHouse-compatible SQL; no need to add `--ClickHouse` parameter for other queries.

2. 2. Prepare for execution: Set single thread and execute the SQL statement:
      
`-- Set single thread`
      
`SET max_threads=1;`
      
`-- Paste the ClickHouse-compatible SQL obtained from SQLChains and execute`
      
`[Paste the ClickHouse-compatible SQL here]`

## Contact

For any issues related to prototype execution or reproducibility, please contact the corresponding author of the paper.

---

Last Updated: VLDB Submission Period 2026

