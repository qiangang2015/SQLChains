# CubeFlows Prototype
## 1. Overview

This README provides instructions for running the CubeFlows prototype ([`Prototype.rar`](https://github.com/gnqn/CubeFlow/releases/tag/V1.0.0)), including system setup, execution commands, and the experimental methods used to reproduce the results in the paper.

## 2. Directory Structure

The prototype requires the following fixed directory structure (already configured in the provided ZIP package). Do NOT modify the structure to ensure successful execution:

      Prototype/

      ├── lib/                 # Dependencies (no action required)

      ├── chains.db            # 1GB dataset/metadata file

      └── SQLChains.jar        # Executable prototype (JDK8 compiled)

## 3. Environment Requirements

JDK Version: JDK 8

## 4. Quick Run

Follow these steps to run the prototype. No additional configuration or data preparation is needed.

### Step 1: Extract the ZIP Package

Extract the provided `Prototype.rar` file to any directory (e.g., `D:\Prototype` on Windows).

### Step 2: Run the Prototype

Navigate to the extracted directory and run:

```bash
cd D:\Prototype
java -Xmx4g -Xms2g -jar SQLChains.jar
```

> **Note:** If you prefer to run from a different directory, make sure `chains.db` is in your current working directory and provide the full path to the JAR file:
> ```bash
> java -Xmx4g -Xms2g -jar /path/to/Prototype/SQLChains.jar
> ```

### Step 3: Verify Execution

A successful startup shows the following prompt:
```bash
Starting CubeFLows prototype...
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
> The experiments in the paper were performed on **Windows 11** with the specific versions noted above. Other versions may work but have not been tested.

### 6.2 Obtaining SQL Statements for DuckDB and ClickHouse

Use `list` in CubeFlows to generate SQL for DuckDB and ClickHouse, e.g., `list tpc-c1.v14`. For ClickHouse, the `--ClickHouse` parameter is only needed for S-C1 queries; other queries do not require it.

CubeFlows can output SQL in either **nested** or **CTE** format. In our experiments, we observed that **nested SQL performs better** than CTE on both DuckDB and ClickHouse. Therefore, all experiments in the paper use the **nested SQL** format.

### 6.3 Execution and Measurement

1. Execute each query **5 times**.
2. Take the **median value** as the final execution cost.

> **Note for DuckDB:** During our experiments, we observed that DuckDB's performance may degrade slightly with continuous execution. To ensure fair and consistent measurements, we recommend exiting DuckDB after each query and restarting the process for the next query. 

> **Note on single-thread execution:** All experiments were conducted in single-threaded mode. For DuckDB and ClickHouse, this is enforced via `PRAGMA threads=1` and `SET max_threads=1`, respectively. The CubeFlows prototype uses DuckDB as its input engine, with the same `PRAGMA threads=1` setting already enforced in the source code. Although the prototype itself does not support multi-threading, we observed that adding the JVM flag `-XX:ActiveProcessorCount=1` helps stabilize performance in our tests with JDK 11.

### 6.4 Run Experiments with CubeFlows

Follow the steps in the previous sections to launch the CubeFlows prototype. Then use the `exec` command with appropriate parameters to execute the specified query chains.

### 6.5 Run Experiments with DuckDB
1. Launch DuckDB and open the dataset:
   ```sql
   .open chains.db
   .timer on
   PRAGMA threads=1;
2. Paste the nested SQL obtained from CubeFlows and execute.

### 6.6 Run Experiments with ClickHouse
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
Set single thread and execute the nested SQL obtained from CubeFlows:
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


## Runtime Screenshots

### Initial Startup of the Prototype
<img width="862" height="34" alt="image" src="https://github.com/user-attachments/assets/8293c89e-c7c6-4956-8df2-28342343221f" />

### Execute TPC-C2
<img width="858" height="352" alt="image" src="https://github.com/user-attachments/assets/c40a1649-37cd-4cb7-b62e-30524267f084" />

### Nested SQL Query for TPC-C2
<img width="854" height="298" alt="image" src="https://github.com/user-attachments/assets/6e82253f-3915-4547-8926-4c84ccfde980" />

### Single-threaded Baseline Execution with DuckDB (Nested SQL)
<img width="854" height="407" alt="image" src="https://github.com/user-attachments/assets/c26d279c-40b6-42f3-92f3-a1e9f9cec149" />



## Visual Flowchart of the Experimental Query Chain

### TPC-C1
<img width="966" height="417" alt="TPC-C1" src="https://github.com/user-attachments/assets/98851111-70ae-4b8e-ac2c-2ce6d6d2891f" />

### TPC-C2
<img width="1047" height="238" alt="TPC-C2" src="https://github.com/user-attachments/assets/fa59fc54-50be-479c-9e37-d3cec6fb1aae" />

### S-C1
<img width="1157" height="299" alt="S-C1" src="https://github.com/user-attachments/assets/bb202a30-a234-410b-9707-9a83f43d2c49" />

### S-C2
<img width="1162" height="341" alt="S-C2" src="https://github.com/user-attachments/assets/e3ba72b6-1c19-48e9-963a-4619a6d1ed73" />

### OR-C1
<img width="869" height="403" alt="OR-C1" src="https://github.com/user-attachments/assets/11fcf356-7a45-4710-9bbf-88773ec1ddfa" />

### OR-C2
<img width="935" height="368" alt="OR-C2" src="https://github.com/user-attachments/assets/06d69b03-43e3-4f14-9153-6780f6afa5e8" />
