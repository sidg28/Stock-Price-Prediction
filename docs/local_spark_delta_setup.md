# Local PySpark + Delta Lake Setup on Windows

This document records the local setup used for Project Phoenix so the environment can be recreated on a new Windows machine.

---

## 1. Install Python

Python used during development:

```text
Python 3.11
```

Verify:

```powershell
python --version
```

---

## 2. Install Java / JDK

Apache Spark requires Java.

JDK used:

```text
Java 17
Eclipse Adoptium / Temurin JDK
```

Verify:

```powershell
java -version
```

Java worked directly after installation; no manual `JAVA_HOME` configuration was required during this setup.

---

## 3. Install PySpark

For the original Spark practice environment:

```powershell
pip install pyspark
```

Verify:

```powershell
python -c "import pyspark; print(pyspark.__version__)"
```

A basic Spark session can be created with:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("Phoenix")
    .master("local[*]")
    .getOrCreate()
)
```

Test:

```python
spark.range(10).show()
```

---

## 4. Windows Hadoop / winutils Setup

Local Spark on Windows required Hadoop Windows utilities.

Create:

```text
C:\hadoop\bin
```

Place the appropriate `winutils.exe` inside:

```text
C:\hadoop\bin\winutils.exe
```

Set the environment variable:

```text
HADOOP_HOME=C:\hadoop
```

Add this to `PATH`:

```text
C:\hadoop\bin
```

Verify in PowerShell:

```powershell
echo $env:HADOOP_HOME
where.exe winutils
```

Expected:

```text
C:\hadoop
C:\hadoop\bin\winutils.exe
```

> Note: Even with `winutils.exe`, the PySpark 4.2.0 local environment encountered a Hadoop `NativeIO$Windows.access0` error while writing Parquet. Because of this, local Windows filesystem behaviour may still differ from Databricks/Linux Spark environments.

---

# Delta Lake Environment

A separate virtual environment was created for Delta Lake instead of modifying the existing Spark environment.

This avoids dependency/version conflicts.

## 5. Create Delta Virtual Environment

From the project directory:

```powershell
python -m venv .venv-delta
```

This creates:

```text
.venv-delta/
```

---

## 6. Activate the Virtual Environment

PowerShell may block `.ps1` scripts because of its execution policy.

For the current PowerShell process only:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Then:

```powershell
.\.venv-delta\Scripts\Activate.ps1
```

Expected:

```text
(.venv-delta) PS D:\github>
```

Alternatively, using Command Prompt:

```cmd
.venv-delta\Scripts\activate.bat
```

---

## 7. Install Compatible Spark + Delta Versions

The Delta environment used:

```text
PySpark 4.0.0
Delta Lake 4.0.0
```

Install:

```powershell
pip install pyspark==4.0.0
pip install delta-spark==4.0.0
pip install jupyter ipykernel
```

Verify:

```powershell
python -c "import pyspark; print(pyspark.__version__)"
```

Expected:

```text
4.0.0
```

---

## 8. Register the Environment as a Jupyter Kernel

Run:

```powershell
python -m ipykernel install --user --name phoenix-delta --display-name "Phoenix Delta"
```

This makes the virtual environment selectable from Jupyter / VS Code notebooks.

---

## 9. Select the Kernel in VS Code

For `.ipynb` notebooks:

```text
Select Kernel
    ↓
Jupyter Kernel
    ↓
Phoenix Delta
```

Verify that the notebook is actually using the virtual environment:

```python
import sys
print(sys.executable)
```

Expected:

```text
D:\Github\.venv-delta\Scripts\python.exe
```

Important:

> Selecting a Python interpreter in VS Code and selecting a Jupyter notebook kernel are separate things. Make sure the notebook itself uses `Phoenix Delta`.

---

## 10. Verify Delta Installation

Inside the notebook:

```python
import pyspark
import delta

print("Spark:", pyspark.__version__)
print("Delta imported successfully")
```

Expected:

```text
Spark: 4.0.0
Delta imported successfully
```

---

## 11. Create a Delta-enabled SparkSession

```python
import pyspark
from delta import configure_spark_with_delta_pip

builder = (
    pyspark.sql.SparkSession.builder
    .appName("PhoenixDelta")
    .master("local[*]")
    .config(
        "spark.sql.extensions",
        "io.delta.sql.DeltaSparkSessionExtension"
    )
    .config(
        "spark.sql.catalog.spark_catalog",
        "org.apache.spark.sql.delta.catalog.DeltaCatalog"
    )
)

spark = configure_spark_with_delta_pip(builder).getOrCreate()
```

---

## 12. Test Delta Lake Locally

Create some sample data:

```python
data = [
    (1, "AAPL"),
    (2, "MSFT")
]

df = spark.createDataFrame(data, ["id", "symbol"])
```

Write it as Delta:

```python
df.write \
    .format("delta") \
    .mode("overwrite") \
    .save("output/test_delta")
```

Read it back:

```python
delta_df = (
    spark.read
    .format("delta")
    .load("output/test_delta")
)

delta_df.show()
```

A successful Delta table directory contains roughly:

```text
output/test_delta/
│
├── _delta_log/
│
└── part-xxxxx.snappy.parquet
```

This demonstrates the basic Delta Lake storage model:

```text
Delta Table
    =
Parquet data files
    +
Delta transaction log (_delta_log)
```

---

# Final Local Environment

```text
Windows
│
├── Python 3.11
│
├── Java 17
│
├── Hadoop Windows utilities
│   └── C:\hadoop\bin\winutils.exe
│
└── Project
    │
    └── .venv-delta
        ├── PySpark 4.0.0
        ├── Delta Lake 4.0.0
        ├── Jupyter
        └── IPython Kernel
             └── "Phoenix Delta"
```

## Useful Verification Commands

```powershell
python --version
java -version
echo $env:HADOOP_HOME
where.exe winutils
```

Inside the notebook:

```python
import sys
import pyspark
import delta

print(sys.executable)
print(pyspark.__version__)
```

---

## Important Lessons from Setup

1. Spark requires a working Java installation.
2. Windows local Spark may require additional Hadoop/winutils configuration.
3. PySpark and Delta Lake versions must be compatible.
4. Use a separate virtual environment to avoid breaking an existing Spark installation.
5. VS Code's Python interpreter and Jupyter kernel are not necessarily the same.
6. A Delta-enabled SparkSession requires Delta-specific Spark configuration.
7. Delta tables still store the underlying data in Parquet files while `_delta_log` provides Delta's transactional table layer.