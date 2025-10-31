## **Overview**

This document explains how to integrate **Weka (machine learning toolkit)** with **Microsoft SQL Server (SSMS)** using a **JDBC driver**.

It enables loading data from SQL tables into Weka and exporting datasets back to SQL.

---

## **1. Repository Contents**

| File | Description |
| --- | --- |
| `RunWeka.ini` | Weka configuration file for external Java and JDBC classpath. |
| `DatabaseUtils.props` | Weka database configuration (driver, URL, credentials). |
| `sqljdbc_13.2.1.0_enu.zip` | Microsoft JDBC driver archive (contains JRE8 `.jar`). |

---

## **2. Prerequisites**

| Component | Version | Location / Notes |
| --- | --- | --- |
| **Weka** | 3.9.6 | `C:\Weka-3-9-6` |
| **SQL Server** | 2022 | `localhost`, port 1433 |
| **Java (JDK)** | 8u472 | `C:\Program Files\jdk8u472-b08` |
| **JDBC Driver** | 13.2.1 | `C:\jdbc\mssql-jdbc.jar` |

---

## **3. Java Setup**

Installed **OpenJDK 8u472 (Temurin)**

Path:

```
C:\Program Files\jdk8u472-b08

```

**Verify installation:**

```bash
"C:\Program Files\jdk8u472-b08\bin\java.exe" -version

```

**Expected output:**

```
openjdk version "1.8.0_472"
OpenJDK Runtime Environment (Temurin)

```

---

## **4. Configure Java Environment Variables**

Set up environment variables to make Java accessible globally.

### **Step 1: Set JAVA_HOME**

1. Press **Win + R**, type `sysdm.cpl`, and open **Environment Variables**.
2. Under **System variables**, click **New**:
    
    ```
    Variable name: JAVA_HOME
    Variable value: C:\Program Files\jdk8u472-b08
    
    ```
    

### **Step 2: Add to PATH**

1. Under **System variables**, select `Path` → **Edit** → **New**.
2. Add:
    
    ```
    %JAVA_HOME%\bin
    
    ```
    

### **Step 3: Verify**

```bash
java -version
javac -version

```

Both should return version `1.8.0_472`.

---

## **5. JDBC Driver Setup**

1. Extract the provided file:
    
    `sqljdbc_13.2.1.0_enu.zip`
    
2. Locate:
    
    ```
    mssql-jdbc-13.2.1.jre8.jar
    
    ```
    
3. Rename and move it to:
    
    ```
    C:\jdbc\mssql-jdbc.jar
    
    ```
    

---

## **6. Verify SQL Server Connectivity**

Check SQL Server port:

```bash
Test-NetConnection -ComputerName localhost -Port 1433

```

Expected output:

```
TcpTestSucceeded : True

```

---

## **7. Enable SQL Authentication in SSMS**

1. In **SSMS → Server Properties → Security**, enable
    
    **SQL Server and Windows Authentication mode**.
    
2. Restart SQL Server Service.
3. Run the following in a new query window:

```sql
ALTER LOGIN sa ENABLE;
ALTER LOGIN sa WITH PASSWORD = 'Admin@123';

```

---

## **8. Configure Weka to Use External Java (JDK8)**

Edit file:

```
C:\Weka-3-9-6\weka.ini

```

Add:

```
java_home=C:\Program Files\jdk8u472-b08

```

---

## **9. Launch Weka with JDBC Classpath**

Start Weka manually using:

```bash
& "C:\Program Files\jdk8u472-b08\bin\java.exe" -cp "C:\Weka-3-9-6\weka.jar;C:\jdbc\mssql-jdbc.jar" weka.gui.GUIChooser

```

---

## **10. Create Database and Sample Table**

Run the following in **SSMS**:

```sql
CREATE DATABASE WekaLab;
USE WekaLab;

CREATE TABLE Students (
    StudentID INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(50),
    Age INT,
    Marks FLOAT,
    Grade CHAR(1)
);

INSERT INTO Students (Name, Age, Marks, Grade) VALUES
('Alice', 20, 85.5, 'A'),
('Bob', 22, 73.0, 'B'),
('Charlie', 21, 68.5, 'C'),
('David', 23, 90.0, 'A'),
('Eve', 19, 55.0, 'D');

```

---

## **11. Configure `DatabaseUtils.props` for Weka**

File path:

```
C:\Users\<YourUser>\wekafiles\props\DatabaseUtils.props

```

Contents:

```
jdbcDriver=com.microsoft.sqlserver.jdbc.SQLServerDriver
jdbcURL=jdbc:sqlserver://localhost:1433;databaseName=WekaLab;encrypt=false;trustServerCertificate=true
userName=sa
password=Admin@123
nominalNull=?
stringNull=?
numericPrecision=10

```

---

## **12. Connect from Weka**

In **Weka Explorer → Preprocess → Open DB → Connect**

Use:

```
Database URL: jdbc:sqlserver://localhost:1433;databaseName=WekaLab;encrypt=false;trustServerCertificate=true
Username: sa
Password: Admin@123

```

Click **Connect** → should display “Connection successful”.

---

## **13. Load Data into Weka**

In **Weka Explorer → Preprocess → Open DB → Query**:

```sql
SELECT * FROM Students;

```

The dataset will appear in Weka’s viewer.

---

## **14. Summary of Fixes**

| Issue | Fix |
| --- | --- |
| Driver not detected | Added `mssql-jdbc.jar` to classpath |
| Wrong Java version | Forced Weka to use JDK8 |
| SSL/PKIX error | Added `encrypt=false;trustServerCertificate=true` |
| Login failed | Enabled mixed authentication and set SA password |
| Missing props | Created `DatabaseUtils.props` under `wekafiles/props` |
| Environment not set | Added `JAVA_HOME` and updated PATH |

---

## **15. Verification Checklist**

| Step | Status |
| --- | --- |
| Java 8 recognized | ✅ |
| PATH and JAVA_HOME configured | ✅ |
| JDBC driver loaded | ✅ |
| SQL Server reachable (port 1433) | ✅ |
| Weka connected successfully | ✅ |
| Data loaded from SQL | ✅ |
