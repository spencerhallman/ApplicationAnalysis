# Application Analysis
## RIF (Resource Interface Facility) - DB2 Statistics Collection System

---

## Table of Contents

1. [Application Overview](#section-1-application-overview)
   - [Application Metadata](#11-application-metadata)
   - [Application Components Inventory](#12-application-components-inventory)
2. [Program-Level Analysis](#section-2-program-level-analysis)
   - [RIFMAIN - Main Control Program](#21-rifmain---main-control-program)
   - [Assembler Modules](#22-assembler-modules)
   - [External Dependencies](#23-external-dependencies)
   - [Complexity Metrics](#24-complexity-metrics)
   - [Code Quality Issues](#25-code-quality-issues)
3. [Application-Level Analysis](#section-3-application-level-analysis)
   - [Application Architecture](#31-application-architecture)
   - [Component Relationship Matrix](#32-component-relationship-matrix)
   - [Data Flow Across Application](#33-data-flow-across-application)
   - [Database Schema Analysis](#34-database-schema-analysis)
4. [Impact Analysis](#section-4-impact-analysis)
   - [Change Impact Assessment Example](#41-change-impact-assessment-example)
   - [Dependency Risk Analysis](#42-dependency-risk-analysis)
5. [Operational Analysis](#section-5-operational-analysis)
   - [Runtime Characteristics](#51-runtime-characteristics)
   - [Startup/Shutdown Process](#52-startupshutdown-process)
   - [Error Handling](#53-error-handling)
   - [Configuration Management](#54-configuration-management)
6. [Maintenance & Modernization](#section-6-maintenance--modernization-recommendations)
   - [Critical Issues](#61-critical-issues-priority-1)
   - [Enhancement Opportunities](#62-enhancement-opportunities)
   - [Modernization Roadmap](#63-modernization-roadmap)
   - [Technical Debt Assessment](#64-technical-debt-assessment)
7. [Build & Deployment](#section-7-build--deployment)
   - [Build Process](#71-build-process)
   - [Installation Steps](#72-installation-steps)
   - [Migration Considerations](#73-migration-considerations)
8. [Documentation Quality](#section-8-documentation-quality)
   - [Strengths](#81-strengths)
   - [Recommendations](#82-recommendations)
9. [Appendices](#appendix-a-statistics-collected)
   - [Appendix A: Statistics Collected](#appendix-a-statistics-collected)
   - [Appendix B: Performance Considerations](#appendix-b-performance-considerations)
   - [Appendix C: Glossary](#appendix-c-glossary)

---

## SECTION 1: APPLICATION OVERVIEW

### 1.1 Application Metadata
- **Application Name:** RIF (Resource Interface Facility)
- **Primary Language(s):** PL/I (Primary), Assembler (Utilities), JCL (Build/Deploy), SQL (DDL)
- **Total Programs:** 27 source modules (23 PL/I, 4 Assembler)
- **Total Lines of Code:** Approximately 15,000+ lines
- **Last Modified Date:** 2016-2020 (based on source comments)
- **Application Type:** Started Task (STC) / Background Service
- **Runtime Environment:** z/OS Batch with DB2 CAF connectivity
- **Purpose:** Real-time DB2 subsystem statistics collection and storage for performance monitoring and analysis

### 1.2 Application Components Inventory

| Component Type | Name | Lines of Code | Purpose | Status |
|----------------|------|---------------|---------|--------|
| PL/I Main Program | RIFMAIN | ~470 | Main driver and control logic | Active |
| PL/I Program | RIFSTIM | ~23 | Timer/delay service | Active |
| PL/I Module | RIFCMND | ~72 | Console command interface | Active |
| PL/I Module | RIFDB2 | ~109 | DB2 CAF connectivity | Active |
| PL/I Module | RIFOPTS | ~326 | Options/configuration parser | Active |
| PL/I Module | SYSWAIT | ~111 | ECB wait/post utilities | Active |
| PL/I Module | DSNQWS0 | ~64 | IFCID 1 index mapping | Active |
| PL/I Module | DSNQWS1 | ~56 | IFCID 2 index mapping | Active |
| PL/I Module | DSNQW03 | ~45 | IFCID 225 index mapping | Active |
| PL/I Module | RIFQWSA | ~171 | CPU usage processor | Active |
| PL/I Module | RIFQJST | ~342 | Log manager processor | Active |
| PL/I Module | RIFQWOS | ~134 | z/OS statistics processor | Active |
| PL/I Module | RIFQ8ST | ~670 | IDAA statistics processor | Active |
| PL/I Module | RIFQBST | ~458 | Buffer pool processor | Active |
| PL/I Module | RIFQIST | ~680+ | Data manager processor | Active |
| PL/I Module | RIFQBGL | ~680+ | Group buffer pool processor | Active |
| PL/I Module | RIFQTXA | ~680+ | Local locking processor | Active |
| PL/I Module | RIFQTGS | ~680+ | Global locking processor | Active |
| PL/I Module | RIFQTST | ~680+ | Service controller processor | Active |
| PL/I Module | RIFQXST | ~680+ | SQL data processor | Active |
| PL/I Module | RIFQ2251 | ~235 | Address space summary processor | Active |
| PL/I Module | INSTEMPL | ~43 | Template for new processors | Template |
| Assembler | RIFATCH | ~84 | ATTACH wrapper | Active |
| Assembler | RIFCMND | ~131 | Console command listener | Active |
| Assembler | RIFPOST | ~81 | POST SVC wrapper | Active |
| Assembler | RIFWAIT | ~83 | WAIT SVC wrapper | Active |
| Configuration | RIFOPTS.opts | ~32 | Runtime configuration | Active |
| DB2 DDL | RIF database schema.sql | ~1338 | Database schema definition | Active |
| JCL | ASM job 1.jcl | ~30 | Compile RIFATCH | Active |
| JCL | ASM job 2.jcl | ~30 | Compile RIFCMND | Active |
| JCL | ASM job 3.jcl | ~30 | Compile RIFWAIT | Active |
| JCL | ASM job 4.jcl | ~30 | Compile RIFPOST | Active |
| JCL | RIFMAIN compile job.jcl | ~78 | Compile RIFMAIN | Active |
| JCL | RIFSTIM compile job.jcl | ~78 | Compile RIFSTIM | Active |
| JCL | RIF package binds.jcl | ~21 | Bind DB2 package | Active |
| JCL | RIF plan binds.jcl | ~21 | Bind DB2 plan | Active |
| JCL | STC start procedure.jcl | ~15 | Started task procedure | Active |

---

## SECTION 2: PROGRAM-LEVEL ANALYSIS

### 2.1 RIFMAIN - Main Control Program

#### Program Overview
- **Program Name:** RIFMAIN
- **Program Type:** Main Entry Point
- **Language:** Enterprise PL/I
- **Total Lines:** ~470
- **Executable Lines:** ~350
- **Comment Lines:** ~120
- **Purpose:** Primary driver for DB2 statistics collection. Controls timing, IFCID collection, data processing, and storage.

#### Program Structure

```
RIFMAIN (Main Procedure)
├── Initialization
│   ├── Include Modules (RIFCMND, SYSWAIT, RIFDB2, RIFOPTS)
│   ├── Include IFCID Mappings (DSNQWS0, DSNQWS1, DSNQW03)
│   └── Include Processors (RIFQWSA, RIFQJST, RIFQWOS, etc.)
├── Main Processing Logic
│   ├── mainproc
│   │   ├── readOptions()
│   │   ├── opencaf()
│   │   ├── getLocalDb2SSID()
│   │   ├── listenForStop()
│   │   ├── saveStartTime()
│   │   └── Main Work Loop (while not shutdown)
│   │       ├── getCurrentUnixTime()
│   │       ├── processIFCID0001()
│   │       ├── processIFCID0002()
│   │       ├── processIFCID0225()
│   │       ├── processDisplayBP()
│   │       └── waitForNextTrigger()
│   │   └── closecaf()
│   ├── processIFCID0001
│   │   ├── getIFI(1)
│   │   ├── processQWSA() - CPU statistics
│   │   ├── processQJST() - Log manager
│   │   ├── processQWOS() - z/OS statistics
│   │   └── COMMIT
│   ├── processIFCID0002
│   │   ├── getIFI(2)
│   │   ├── processQ8ST() - IDAA data
│   │   ├── processQIST() - Data manager
│   │   ├── processQBST() - Buffer pools
│   │   ├── processQBGL() - Group buffer pools
│   │   ├── processQTXA() - Local locking
│   │   ├── processQTGS() - Global locking
│   │   ├── processQTST() - Service controller
│   │   ├── processQXST() - SQL activity
│   │   └── COMMIT
│   ├── processIFCID0225
│   │   ├── getIFI(225)
│   │   ├── processQW02251() - Address space info
│   │   └── COMMIT
│   ├── getIFI(ifcid_nr)
│   ├── getCurrentUnixTime()
│   ├── getLocalDb2SSID()
│   ├── saveStartTime()
│   ├── waitForNextTrigger()
│   └── endpli()
└── Error Handling
    └── SQL error checking via checkSQL()
```

#### Control Flow Analysis

**Logic Flow Diagram:**
```
START (RIFMAIN)
  │
  ├─→ Read Configuration (readOptions)
  │     │
  │     ├─→ Parse RIFOPTS file
  │     └─→ Set gather flags
  │
  ├─→ Open DB2 Connection (opencaf via CAF)
  │     │
  │     ├─→ Success? ─No─→ ERROR ─→ STOP
  │     │
  │     Yes
  │     │
  ├─→ Get DB2 SSID (getLocalDb2SSID)
  │
  ├─→ Start Command Listener (listenForStop)
  │     │
  │     └─→ ATTACH RIFATCH subtask
  │
  ├─→ Initialize Timestamp (saveStartTime)
  │
  ├─→ MAIN WORK LOOP (while shutdown='N')
  │     │
  │     ├─→ Get Current Unix Time
  │     │
  │     ├─→ Process IFCID 1 (System Stats)
  │     │     ├─→ Call ADMIN_INFO_IFCID SP
  │     │     ├─→ Parse QWSA (CPU data)
  │     │     ├─→ Parse QJST (Log manager)
  │     │     ├─→ Parse QWOS (z/OS stats)
  │     │     ├─→ INSERT into DB2 tables
  │     │     └─→ COMMIT
  │     │
  │     ├─→ Process IFCID 2 (Database Stats)
  │     │     ├─→ Call ADMIN_INFO_IFCID SP
  │     │     ├─→ Parse multiple subsections
  │     │     ├─→ INSERT into DB2 tables
  │     │     └─→ COMMIT
  │     │
  │     ├─→ Process IFCID 225 (Address Space)
  │     │     ├─→ Call ADMIN_INFO_IFCID SP
  │     │     ├─→ Parse QW02251
  │     │     ├─→ INSERT into DB2 tables
  │     │     └─→ COMMIT
  │     │
  │     ├─→ Wait for Next Minute
  │     │     ├─→ ATTACH RIFSTIM (timer thread)
  │     │     ├─→ WAIT on ECB list
  │     │     │     ├─→ Timer ECB posted? → Continue
  │     │     │     └─→ Stop ECB posted? → Exit loop
  │     │     └─→ DETACH timer thread
  │     │
  │     └─→ Loop back
  │
  ├─→ Close DB2 Connection (closecaf)
  │
  └─→ STOP
```

#### Data Structure Analysis

**Key Data Structures:**

| Data Item | Level | Type | Size | Usage | Defined In |
|-----------|-------|------|------|-------|------------|
| admin_info_ssid | 01 | Structure | ~1350 | SP parameters | RIFMAIN Line 42 |
| admin_info_ifcid | 01 | Structure | ~1350 | SP parameters | RIFMAIN Line 53 |
| admin_info_ifcid_rs | 01 | Structure | 32004 | Result set | RIFMAIN Line 61 |
| rs_loc1 | - | RESULT_SET_LOCATOR | 4 | Cursor locator | RIFMAIN Line 65 |
| current_unix_time | - | BIN FIXED(63) | 8 | Timestamp | RIFMAIN Line 81 |
| RIFSTIMParmlist | 01 | Structure | 12 | Timer parameters | RIFMAIN Line 90 |
| STOPECB | - | ECB (UNION) | 4 | Stop signal | RIFMAIN Line 98 |
| ECBPtrList | Array | POINTER(2) | 8 | Wait list | RIFMAIN Line 116 |
| rif_options | 01 | Structure | ~32 | Configuration | RIFOPTS Line 4 |
| comnd_commarea | 01 | UNION | 17 | Command area | RIFCMND Line 35 |

**Data Flow Analysis - Current Unix Time:**
```
getCurrentUnixTime() procedure (Line 366)
    ↓
SQL: SELECT calculation FROM SYSDUMMY1
    ↓
Store in :current_unix_time (BIN FIXED(63))
    ↓
Used in all INSERT statements as UNIX_TIME column
    ↓
Written to all statistics tables
```

### 2.2 Assembler Modules

#### RIFWAIT - ECB Wait Service
- **Lines:** 83
- **Purpose:** Issue WAIT SVC on ECB list
- **Key Operations:**
  - Accepts pointer to ECB list
  - Issues SVC 1 (WAIT) with LONG=YES
  - Returns when any ECB is posted
- **Register Usage:** R0 (ECB count), R1 (ECB list pointer), R12 (base)
- **Linkage:** LE-compliant, callable from PL/I

#### RIFPOST - ECB Post Service
- **Lines:** 81
- **Purpose:** Issue POST SVC on single ECB
- **Key Operations:**
  - Accepts pointer to ECB
  - Issues SVC 2 (POST)
  - Clears R0 before POST
- **Register Usage:** R0 (cleared), R1 (ECB pointer), R12 (base)
- **Linkage:** LE-compliant, callable from PL/I

#### RIFCMND - Console Command Listener
- **Lines:** 131
- **Purpose:** Listen for MVS STOP command from console
- **Key Operations:**
  - EXTRACT communication ECB
  - QEDIT to clear initial CIB
  - WAIT on console ECB
  - Check for STOP command (X'40')
  - POST stop ECB to main task
  - SET shutdown flag to 'Y'
- **Complexity:** Medium - uses MVS communication services

#### RIFATCH - Attach Wrapper
- **Lines:** 84
- **Purpose:** ATTACH RIFCMND as subtask
- **Key Operations:**
  - WTO messages for status
  - ATTACH EP=RIFCMND
  - Abend on failure (DC XL8'00000000')
- **Risk:** ⚠️ Hardcoded abend instead of proper error handling

### 2.3 External Dependencies

#### Called Programs/Services

| Program/Service | Call Type | Parameters | Return Values | Used By |
|-----------------|-----------|------------|---------------|---------|
| SYSPROC.ADMIN_INFO_SSID | CALL (SP) | subsystem_name, retcd, errmsg | SSID name | RIFMAIN Line 403 |
| SYSPROC.ADMIN_INFO_IFCID | CALL (SP) | ifcid_nr, db2mem, retcd, errmsg | Result set with IFCID data | RIFMAIN Line 277 |
| RIFSTIM | ATTACH | Parmlist pointer | - | RIFMAIN Line 450 |
| RIFATCH | CALL | Commarea pointer | - | RIFCMND Line 68 |
| RIFWAIT | CALL | ECB list pointer | - | RIFMAIN Line 454 |
| RIFPOST | CALL | ECB pointer | - | RIFSTIM Line 21 |
| DSNALI | CALL (CAF) | Function, SSID, Plan, retcode, reascode | DB2 connection | RIFDB2 Lines 34, 65 |

#### Included Modules

| Module Name | Replaced By | Used In | Contents |
|-------------|-------------|---------|----------|
| RIFCMND | %include | RIFMAIN Line 29 | Console command structures |
| SYSWAIT | %include | RIFMAIN Line 30 | ECB wait/post definitions |
| RIFDB2 | %include | RIFMAIN Line 36 | DB2 CAF connectivity |
| RIFOPTS | %include | RIFMAIN Line 125 | Options structures |
| DSNQWS0 | %include | RIFMAIN Line 130 | IFCID 1 index |
| DSNQWS1 | %include | RIFMAIN Line 131 | IFCID 2 index |
| DSNQW03 | %include | RIFMAIN Line 132 | IFCID 225 index |
| RIFQWSA through RIFQ2251 | %include | RIFMAIN Lines 135-153 | IFCID processors |

#### Database Access

**DB2 Tables Accessed:**

| Table Name | Operations | Key Columns | Purpose |
|------------|------------|-------------|---------|
| RIF.RIF_0001_CPU_DATA | INSERT | INSERT_TS, SUBSYSTEM, QWSAPROC | CPU usage by address space |
| RIF.RIF_0001_LOG_MANAGER | INSERT | INSERT_TS, SUBSYSTEM | Log manager statistics |
| RIF.RIF_0001_ZOS_STATISTICS | INSERT | INSERT_TS, SUBSYSTEM | z/OS system statistics |
| RIF.RIF_0002_BUFFER_POOL | INSERT | INSERT_TS, SUBSYSTEM, QBSTPID | Buffer pool activity |
| RIF.RIF_0002_DATA_MANAGER | INSERT | INSERT_TS, SUBSYSTEM | Data manager statistics |
| RIF.RIF_0002_GLOBAL_LOCKING_DATA | INSERT | INSERT_TS, SUBSYSTEM | Global locking (IRLM) |
| RIF.RIF_0002_GROUP_BUFFER_POOL | INSERT | INSERT_TS, SUBSYSTEM, QBGLGN | GBP statistics |
| RIF.RIF_0002_IDAA | INSERT | INSERT_TS, SUBSYSTEM, Q8STNAME | IDAA accelerator stats |
| RIF.RIF_0002_LOCAL_LOCKING_DATA | INSERT | INSERT_TS, SUBSYSTEM | Local lock statistics |
| RIF.RIF_0002_SERVICE_CONTROLLER | INSERT | INSERT_TS, SUBSYSTEM | Service controller data |
| RIF.RIF_0002_SQL_DATA | INSERT | INSERT_TS, SUBSYSTEM | SQL activity counters |
| RIF.RIF_0225_ADDRESS_SPACE_SUMMARY | INSERT | INSERT_TS, SUBSYSTEM, QW0225AN | Address space memory |

**SQL Statements:**

```sql
-- Line 367: Get Unix Timestamp
SELECT (86400*(CAST(DAYS(CURRENT TIMESTAMP - CURRENT TIMEZONE) AS BIGINT)
       - DAYS('1970-01-01'))
       + MIDNIGHT_SECONDS(CURRENT TIMESTAMP - CURRENT TIMEZONE)) * 1000
  INTO :current_unix_time
  FROM SYSIBM.SYSDUMMY1;

-- Line 423: Calculate next interval
SELECT TIMESTAMP(:save_timestamp) + 1 MINUTE
      ,CAST(((TIMESTAMP(:save_timestamp) - CURRENT_TIMESTAMP) * 1000) AS INT)
  INTO :save_timestamp, :waitTime
  FROM SYSIBM.SYSDUMMY1;

-- Example INSERT (QWSA processor, Line 57)
INSERT INTO RIF.RIF_0001_CPU_DATA
VALUES(CURRENT TIMESTAMP
      ,:current_unix_time
      ,:admin_info_ssid.subsystem_name
      ,:ins_QWSA.qwsaproc
      ,:ins_QWSA.qwsapsrb
      ,:ins_QWSA.qwsaejst
      ,:ins_QWSA.qwsasrbt
      ,:ins_QWSA.qwsaiipt
      ,:ins_QWSA.qwsapsrb_ziip);
```

### 2.4 Complexity Metrics

#### RIFMAIN Complexity

**McCabe Cyclomatic Complexity:**
- **Main Procedure Complexity:** ~15 (Medium)
- **Decision Points:** 12
  - IF checks for firstItteration (multiple)
  - IF checks for gather flags (multiple)
  - WHILE loop (shutdown check)
  - IF checks for SQL errors
  - IF checks for debug mode
- **Independent Paths:** ~15
- **Complexity Rating:** Medium (10-20)

**Code Quality Indicators:**

| Metric | Value | Threshold | Status |
|--------|-------|-----------|--------|
| Average procedure length | 45 lines | <100 | ✓ Pass |
| Maximum nesting depth | 4 | <5 | ✓ Pass |
| GO TO count | 1 | <5 | ✓ Pass |
| Comment ratio | 25% | >10% | ✓ Pass |
| Dead code lines | 0 | 0 | ✓ Pass |
| Modular design | High | - | ✓ Pass |

### 2.5 Code Quality Issues

#### Potential Issues

| Line/Module | Issue Type | Description | Severity | Recommendation |
|-------------|------------|-------------|----------|----------------|
| RIFMAIN:465 | GO TO usage | Uses GOTO for error termination | Low | Acceptable for error handling |
| RIFATCH:65 | Forced abend | DC XL8'00000000' forces abend | Medium | Add proper error recovery |
| RIFQWSA:13 | Logic error | Checks gather_QBGL instead of gather_QWSA | High | Fix to check correct flag |
| Multiple | Hard-coded values | Table names hard-coded | Low | Consider using configuration |
| RIFOPTS:5 | Default SSID | 'DERP' as default | Low | Ensure changed at install |

#### Strengths

| Aspect | Description |
|--------|-------------|
| Modular Design | Excellent separation of concerns with include modules |
| Error Handling | Comprehensive SQL error checking throughout |
| Documentation | Good inline comments explaining logic |
| Data Validation | Checks for first iteration to ensure delta calculations |
| Configurability | Flexible option-driven data collection |
| Resource Management | Proper CAF open/close, COMMIT points |
| Asynchronous Design | Clean use of ATTACH/WAIT/POST for parallelism |

---

## SECTION 3: APPLICATION-LEVEL ANALYSIS

### 3.1 Application Architecture

#### System Structure

```
RIF APPLICATION ARCHITECTURE
│
├── STARTED TASK COMPONENTS
│   │
│   ├── RIFMAIN (Main Task - TCB 1)
│   │   ├── Responsibilities:
│   │   │   ├── Configuration management
│   │   │   ├── DB2 CAF connection management
│   │   │   ├── IFCID data collection orchestration
│   │   │   ├── Data processing coordination
│   │   │   ├── DB2 INSERT operations
│   │   │   └── Timing/scheduling
│   │   │
│   │   └── Calls:
│   │       ├── ADMIN_INFO_SSID (DB2 SP)
│   │       ├── ADMIN_INFO_IFCID (DB2 SP)
│   │       ├── RIFSTIM (ATTACH)
│   │       ├── RIFATCH (CALL)
│   │       └── RIFWAIT (CALL)
│   │
│   ├── RIFCMND Subtask (TCB 2 - via RIFATCH)
│   │   ├── Responsibilities:
│   │   │   ├── Console command monitoring
│   │   │   ├── STOP command detection
│   │   │   └── Shutdown signaling
│   │   │
│   │   └── Uses:
│   │       ├── EXTRACT (MVS communication)
│   │       ├── QEDIT (CIB management)
│   │       ├── WAIT (console ECB)
│   │       └── POST (stop ECB)
│   │
│   └── RIFSTIM Thread (TCB 3 - per iteration)
│       ├── Responsibilities:
│       │   ├── Delay for specified milliseconds
│       │   └── POST timer ECB when elapsed
│       │
│       └── Uses:
│           ├── DELAY (PL/I built-in)
│           └── RIFPOST (POST SVC)
│
├── PROCESSING MODULES (Included in RIFMAIN)
│   │
│   ├── IFCID 1 Processors (System Statistics)
│   │   ├── RIFQWSA → RIF_0001_CPU_DATA
│   │   ├── RIFQJST → RIF_0001_LOG_MANAGER
│   │   └── RIFQWOS → RIF_0001_ZOS_STATISTICS
│   │
│   ├── IFCID 2 Processors (Database Statistics)
│   │   ├── RIFQ8ST → RIF_0002_IDAA
│   │   ├── RIFQBST → RIF_0002_BUFFER_POOL
│   │   ├── RIFQIST → RIF_0002_DATA_MANAGER
│   │   ├── RIFQBGL → RIF_0002_GROUP_BUFFER_POOL
│   │   ├── RIFQTXA → RIF_0002_LOCAL_LOCKING_DATA
│   │   ├── RIFQTGS → RIF_0002_GLOBAL_LOCKING_DATA
│   │   ├── RIFQTST → RIF_0002_SERVICE_CONTROLLER
│   │   └── RIFQXST → RIF_0002_SQL_DATA
│   │
│   └── IFCID 225 Processors (Address Space)
│       └── RIFQ2251 → RIF_0225_ADDRESS_SPACE_SUMMARY
│
├── UTILITY MODULES
│   ├── RIFDB2 - DB2 CAF connection management
│   ├── RIFOPTS - Configuration file parsing
│   ├── SYSWAIT - ECB wait/post definitions
│   └── DSNQWS0/1/03 - IFCID structure indexes
│
├── DATABASE (DB2)
│   ├── Database: RIF
│   ├── Tablespaces: 12 partitioned (19 partitions each)
│   ├── Tables: 12 (statistics storage)
│   ├── Indexes: 24 (12 PK + 12 timestamp)
│   └── Stogroup: RIF
│
└── EXTERNAL INTERFACES
    ├── DB2 Stored Procedures
    │   ├── SYSPROC.ADMIN_INFO_SSID
    │   └── SYSPROC.ADMIN_INFO_IFCID
    │
    ├── MVS Services
    │   ├── Console commands (MODIFY, STOP)
    │   ├── WTO (operator messages)
    │   └── ATTACH/DETACH (subtask management)
    │
    └── z/OS Services
        ├── SVC 1 (WAIT)
        ├── SVC 2 (POST)
        ├── EXTRACT (communication)
        └── QEDIT (CIB management)
```

### 3.2 Component Relationship Matrix

| Component | Calls | Called By | Reads | Writes | DB2 Tables | Type |
|-----------|-------|-----------|-------|--------|------------|------|
| RIFMAIN | RIFSTIM, RIFATCH, RIFWAIT, RIFPOST, DSNALI, SP | - | RIFOPT DD | SYSPRINT | All 12 | Main |
| RIFSTIM | RIFPOST | RIFMAIN (ATTACH) | - | - | - | Service |
| RIFATCH | RIFCMND | RIFMAIN | - | - | - | Wrapper |
| RIFCMND | RIFPOST | RIFATCH | - | - | - | Service |
| RIFWAIT | - | RIFMAIN | - | - | - | Utility |
| RIFPOST | - | RIFSTIM, RIFCMND | - | - | - | Utility |
| DSNALI | - | RIFMAIN (via RIFDB2) | - | - | - | DB2 CAF |

### 3.3 Data Flow Across Application

#### Critical Data Flows

```
TIMING & CONTROL FLOW:
z/OS Console → RIFCMND (STOP command) → POST STOPECB → RIFMAIN (shutdown)
                                              ↑
RIFMAIN → RIFSTIM (timer) ───────────────────┘ (POST timer ECB every minute)


STATISTICS COLLECTION FLOW:
DB2 Subsystem (running, generating IFCIDs)
    ↓
RIFMAIN calls ADMIN_INFO_IFCID(ifcid_nr=1)
    ↓
DB2 returns IFCID data in result set
    ↓
RIFMAIN: allocate cursor, fetch row into :ifirec (VARCHAR(32000))
    ↓
Parse IFCID using based structures (QWS0, QWSA, QJST, QWOS)
    ↓
Calculate deltas (current - saved_previous)
    ↓
Convert units (e.g., CPU time / 4096000000 for seconds)
    ↓
INSERT into DB2 tables (RIF_0001_* tables)
    ↓
COMMIT
    ↓
Save current values for next iteration delta
    ↓
[Repeat for IFCID 2, IFCID 225]
    ↓
WAIT for next minute (RIFSTIM + RIFWAIT)
    ↓
Loop back


DATA PERSISTENCE:
In-Memory (PL/I structures)
    ↓
DB2 Tables (partitioned by INSERT_TS)
    ↓
Analytics/Reporting Tools (external, not in scope)
```

### 3.4 Database Schema Analysis

#### Database Objects

**Database:** RIF
- **Bufferpool:** BP2 (tables), BP20 (indexes)
- **Stogroup:** RIF (uses volume list with VCAT)
- **CCSID:** UNICODE (tables), EBCDIC (database)

**Tablespaces:**

| Tablespace | Tables | Partitions | DSSIZE | BUFFERPOOL | Purpose |
|------------|--------|------------|--------|------------|---------|
| RIFQWSA | RIF_0001_CPU_DATA | 19 | 4G | BP2 | CPU usage statistics |
| RIFQJST | RIF_0001_LOG_MANAGER | 19 | 4G | BP2 | Log manager statistics |
| RIFQWOS | RIF_0001_ZOS_STATISTICS | 19 | 4G | BP2 | z/OS system statistics |
| RIFQBST | RIF_0002_BUFFER_POOL | 19 | 4G | BP2 | Buffer pool statistics |
| RIFQIST | RIF_0002_DATA_MANAGER | 19 | 4G | BP2 | Data manager statistics |
| RIFQBGL | RIF_0002_GROUP_BUFFER_POOL | 19 | 4G | BP2 | Group BP statistics |
| RIFQTGS | RIF_0002_GLOBAL_LOCKING_DATA | 19 | 4G | BP2 | Global lock statistics |
| RIFQTXA | RIF_0002_LOCAL_LOCKING_DATA | 19 | 4G | BP2 | Local lock statistics |
| RIFQTST | RIF_0002_SERVICE_CONTROLLER | 19 | 4G | BP2 | Service controller stats |
| RIFQXST | RIF_0002_SQL_DATA | 19 | 4G | BP2 | SQL activity statistics |
| RIFQ8ST | RIF_0002_IDAA | 19 | 4G | BP2 | IDAA accelerator stats |
| RIFQ2251 | RIF_0225_ADDRESS_SPACE_SUMMARY | 19 | 4G | BP2 | Address space memory |

**Partitioning Strategy:**
- All tables partitioned by `INSERT_TS` (TIMESTAMP)
- 19 partitions per table (configurable)
- Each partition represents one day of data
- Partition limit keys: 2020-10-28 through 2020-11-15 **(example dates from DDL - must be updated to current/future dates during installation)**
- **⚠️ Critical:** Partition dates must be rotated daily or ALTER'd to future dates

**Indexes:**

| Index Type | Naming Pattern | Purpose | BUFFERPOOL |
|------------|----------------|---------|------------|
| Primary Key | PK_RIF_* | Uniqueness, query optimization | Default |
| Timestamp | IX_RIF_*__INSERT_TS | Time-range queries | BP20 |

---

## SECTION 4: IMPACT ANALYSIS

### 4.1 Change Impact Assessment Example

**Change Description:** "Add new field QWOSZIIP (zIIP usage) to QWOS processor and RIF_0001_ZOS_STATISTICS table"

#### Direct Impacts

| Entity Type | Entity Name | Impact Type | Required Action |
|-------------|-------------|-------------|-----------------|
| DB2 Table | RIF_0001_ZOS_STATISTICS | Schema Change | ALTER TABLE ADD COLUMN QWOSZIIP REAL |
| PL/I Include | RIFQWOS.pli | Code Change | Add field to qwos, ins_qwos structures |
| PL/I Include | RIFQWOS.pli | Code Change | Add field to INSERT statement |

#### Indirect Impacts

| Entity Type | Entity Name | Impact Reason | Analysis Required | Recompile? | Retest? |
|-------------|-------------|---------------|-------------------|------------|---------|
| Program | RIFMAIN | Includes RIFQWOS | No (transparent) | Yes | Yes |
| JCL | RIFMAIN compile job | Recompile needed | No | No | No |
| JCL | RIF package binds | DBRM changes | No | No | No |
| Configuration | RIFOPTS.opts | No change | No | No | No |

#### Estimated Impact
- **Programs Requiring Change:** 1 (RIFQWOS.pli)
- **Programs Requiring Recompile:** 2 (RIFMAIN, RIFSTIM potentially)
- **Programs Requiring Testing:** 1 (RIFMAIN)
- **DB2 Objects Modified:** 1 (table)
- **Estimated Effort:** 4-8 hours
- **Risk Level:** Low

### 4.2 Dependency Risk Analysis

| Dependency Type | Entity | Risk Level | Reason | Mitigation |
|-----------------|--------|------------|--------|------------|
| Single Point of Failure | RIFMAIN | High | All processing in one task | Add restart capability, logging |
| External Dependency | DB2 Subsystem | High | Application non-functional without DB2 | Add connection retry, queue data |
| External Dependency | ADMIN_INFO_IFCID | High | Core data collection depends on SP | No direct mitigation, IBM-supplied |
| Configuration | RIFOPTS file | Medium | Invalid config causes abend | Add validation, defaults |
| Partition Management | All tablespaces | Medium | Full partitions cause INSERT failures | Automate partition rotation |
| Resource Constraint | DB2 CGTT space | Medium | ADMIN_INFO_IFCID fails with no WORKFILE | Monitor, skip iteration on failure |
| Hard-coded Values | Table names | Low | Schema changes require code changes | Consider dynamic SQL, metadata |

---

## SECTION 5: OPERATIONAL ANALYSIS

### 5.1 Runtime Characteristics

**Performance Profile:**
- **Execution Frequency:** Continuous (STC)
- **Collection Interval:** Every 1 minute (configurable)
- **Data Volume:** ~12 rows/minute (1 per table if all gather flags enabled)
- **Daily Data Volume:** ~17,280 rows/day
- **Storage Growth:** ~200-500 MB/day (estimated, depends on partitioning)

**Resource Requirements:**
- **CPU:** Low (~2-5% of 1 CPU)
- **Memory:** Medium (~50-100 MB REGION)
- **I/O:** Medium (DB2 INSERT operations, log writes)
- **DB2 Threads:** 1 (CAF connection)
- **DB2 Packages:** 1 (RIF.RIFMAIN)
- **DB2 Plan:** 1 (RIF)

### 5.2 Startup/Shutdown Process

**Startup:**
```
1. MVS Operator issues: S RIFA,optmember=<member>
2. JES allocates STEPLIB, RIFOPT DD statements
3. RIFMAIN begins execution
4. Read options from RIFOPT DD
5. Open CAF connection to DB2
6. Get local DB2 SSID
7. ATTACH RIFCMND subtask (console listener)
8. Enter main loop (start collecting data)
9. Display "RIF0099I - RIF IS STARTED"
```

**Shutdown:**
```
1. MVS Operator issues: P RIFA  (or STOP command)
2. RIFCMND detects STOP command
3. Set shutdown flag = 'Y'
4. POST STOPECB
5. RIFMAIN exits main loop
6. Close CAF connection
7. Terminate (all subtasks cleaned up by LE)
```

### 5.3 Error Handling

**Error Categories:**

| Error Type | Handling | Recovery | Logging |
|------------|----------|----------|---------|
| SQL Error (Warning) | Log, continue | Skip current iteration | SYSPRINT, WTO |
| SQL Error (Critical) | Log, set firstItteration | Restart next iteration | SYSPRINT, WTO |
| SQL Error (Fatal) | GOTO quit, terminate | None (operator restart) | SYSPRINT, WTO |
| CAF Open Failure | Display error, GOTO quit | None (operator restart) | SYSPRINT, WTO |
| CAF Close Failure | Display error, continue | Continue shutdown | SYSPRINT, WTO |
| SP Return Code != 0 | Display error, terminate | None (operator restart) | SYSPRINT, WTO |
| ATTACH Failure | Abend | None | WTO, Abend dump |

**⚠️ Risk:** Limited error recovery capability - most errors cause termination

### 5.4 Configuration Management

**Configuration File (RIFOPTS):**

Key options:
```
subsystem_id                    <DB2 subsystem name>
debug_mode                      YES/NO
gather_QWSA                     YES/NO
gather_QJST                     YES/NO
gather_QWOS                     YES/NO
gather_Q8ST                     YES/NO
gather_QBST                     YES/NO
gather_QIST                     YES/NO
gather_QISE                     YES/NO
gather_QBGL                     YES/NO
gather_QTXA                     YES/NO
gather_QTGS                     YES/NO
gather_QXCA                     YES/NO
gather_QTST                     YES/NO
gather_QXST                     YES/NO
gather_QW02251                  YES/NO
[... additional IFCID 225 subsections]
```

**Flexibility:**
- ✓ Can enable/disable individual IFCID processors
- ✓ Can enable debug mode for detailed logging
- ✓ Can target different DB2 subsystems
- ✓ Can use different option sets via STC parameter

---

## SECTION 6: MAINTENANCE & MODERNIZATION RECOMMENDATIONS

### 6.1 Critical Issues (Priority 1)

| Priority | Issue | Location | Effort | Benefit | Recommended Action |
|----------|-------|----------|--------|---------|-------------------|
| 1 | Logic error - wrong flag check | RIFQWSA:13 | 5 min | High | Change gather_QBGL to gather_QWSA |
| 1 | Partition rotation not automated | DDL schema | 1 day | Critical | Create automated partition rotation procedure |
| 1 | Hardcoded abend | RIFATCH:65 | 2 hours | Medium | Replace with proper error return and message |
| 2 | Limited error recovery | RIFMAIN | 1 week | High | Add restart logic, queuing for transient DB2 errors |
| 2 | No monitoring/alerting | - | 3 days | High | Add health check procedure, alerts for failures |

### 6.2 Enhancement Opportunities

**Opportunity 1: Enhanced Error Resilience**
- **Current State:** Most errors cause application termination
- **Recommendation:** 
  - Add retry logic for transient DB2 errors
  - Queue failed INSERTs for later retry
  - Continue processing on SP errors (log and skip iteration)
- **Effort:** 2-3 weeks
- **Benefit:** Improved availability, reduced operator intervention

**Opportunity 2: Dynamic Table Management**
- **Current State:** Table names hard-coded in each processor
- **Recommendation:** 
  - Create metadata table mapping IFCID → table name
  - Use dynamic SQL for INSERTs
  - Enable table name changes without code changes
- **Effort:** 1 week
- **Benefit:** Improved maintainability, flexibility

**Opportunity 3: Performance Optimization**
- **Current State:** 12 separate INSERT + COMMIT cycles per iteration
- **Recommendation:** 
  - Batch all INSERTs in single COMMIT
  - Use multi-row INSERT where applicable
  - Consider asynchronous INSERT processing
- **Effort:** 1 week
- **Benefit:** Reduced log I/O, improved throughput

**Opportunity 4: Automated Partition Management**
- **Current State:** Manual partition rotation required
- **Recommendation:** 
  - Create DB2 stored procedure to ALTER partitions
  - Call daily via scheduled job or within RIF
  - Add alerts for approaching partition limits
- **Effort:** 3-5 days
- **Benefit:** Reduced operational overhead, avoided outages

### 6.3 Modernization Roadmap

#### Phase 1: Stabilization (1-2 months)
- **Week 1-2:**
  - Fix critical bug (RIFQWSA flag check)
  - Add comprehensive error logging
  - Create partition rotation procedure
  - Document recovery procedures
- **Week 3-4:**
  - Add retry logic for transient errors
  - Implement health check procedure
  - Create monitoring/alerting
- **Week 5-8:**
  - Enhance error messages with diagnostic data
  - Add performance instrumentation
  - Create operator runbook

#### Phase 2: Enhancement (3-4 months)
- **Month 3:**
  - Implement dynamic table management
  - Optimize batch INSERT performance
  - Add automated partition management
- **Month 4:**
  - Add data retention/archive capability
  - Implement data quality checks
  - Create administrative utilities

#### Phase 3: Evolution (6-12 months)
- **Optional Future Enhancements:**
  - REST API for query access to statistics
  - Real-time dashboard integration
  - Predictive alerting based on trends
  - Multi-subsystem collection aggregation
  - Cloud-based analytics integration

### 6.4 Technical Debt Assessment

| Debt Type | Severity | Impact | Estimated Remediation | Priority |
|-----------|----------|--------|----------------------|----------|
| Partition management | High | Production outage risk | 40 hours | 1 |
| Limited error recovery | High | Availability | 80 hours | 1 |
| Hard-coded table names | Medium | Maintainability | 40 hours | 2 |
| Forced abend (RIFATCH) | Medium | Diagnostics | 4 hours | 2 |
| Single-threaded INSERTs | Low | Performance | 40 hours | 3 |
| No automated testing | Medium | Change risk | 80 hours | 3 |
| No health monitoring | Medium | Observability | 24 hours | 2 |

**Total Technical Debt:** ~308 hours (7.7 weeks)

---

## SECTION 7: BUILD & DEPLOYMENT

### 7.1 Build Process

**Assembler Modules (4 jobs):**
```
Step 1: ASM
  - Program: ASMA90
  - Options: DECK, NOOBJECT, LIST, XREF(SHORT), NORENT
  - Input: RIFASM(member)
  - Output: &&OBJSET (deck)

Step 2: LKED
  - Program: IEWL
  - Options: XREF, LET, LIST, MAP, AC=1, NORENT, AMODE=31, RMODE=ANY
  - Input: &&OBJSET
  - Output: RIFLOAD(member)
```

**PL/I Programs (2 jobs):**
```
Step 1: PLICOMP
  - Program: IBMZPLI
  - Options: PP(SQL('ATTACH(CAF)')), ARCH(12), OPTIMIZE(TIME)
  - Preprocessor: DB2 (automatic via PP option)
  - Input: RIFSRC(member)
  - Output: &&OBJ (object), RIFDBRM(member) (DBRM)

Step 2: LKED
  - Program: HEWL
  - Options: LIST, XREF, LET, MAP, AMODE=31, RMODE=ANY, NORENT, AC(00)
  - Input: &&OBJ
  - Output: RIFLOAD(member)
  - Entry: CEESTART (LE initialization)
```

**DB2 Binds (2 jobs):**
```
BIND PACKAGE:
  - Collection: RIF
  - Member: RIFMAIN
  - Qualifier: RIF
  - Isolation: CS
  - Validate: RUN
  - Encoding: EBCDIC

BIND PLAN:
  - Plan: RIF
  - PKLIST: RIF.*
  - Validate: BIND
  - Isolation: CS
  - RETAIN: Yes
```

### 7.2 Installation Steps

1. **Allocate Datasets:**
   ```
   <RIFHLQ>.RIFASM   - PDS, RECFM=FB, LRECL=80 (assembler source)
   <RIFHLQ>.RIFSRC   - PDS, RECFM=FB, LRECL=80 (PL/I source) 
   <RIFHLQ>.RIFOPTS  - PDS, RECFM=FB, LRECL=80 (config members)
   <RIFHLQ>.RIFLOAD  - PDSE, RECFM=U (load modules)
   <RIFHLQ>.RIFDBRM  - PDS, RECFM=FB, LRECL=80 (DBRMs)
   ```

2. **Customize Installation:**
   - Edit DDL: Replace `????` with VCAT name, BUFFERPOOL names
   - Edit JCL: Replace `<RIFHLQ>` with installation HLQ
   - Edit JCL: Replace `<DB2HLQ>` with DB2 installation HLQ
   - Edit DDL: Update partition limit keys to current/future dates
   - Edit RIFOPTS: Set subsystem_id, gather flags
   - Edit STC proc: Replace `<RIFHLQ>`, `<DB2HLQ>`, set RIFOPT member

3. **Create DB2 Objects:**
   ```
   Execute: RIF database schema.sql via SPUFI or DSNTEP2
   ```

4. **Compile Programs:**
   ```
   Submit: ASM job 1.jcl (RIFATCH)
   Submit: ASM job 2.jcl (RIFCMND)
   Submit: ASM job 3.jcl (RIFWAIT)
   Submit: ASM job 4.jcl (RIFPOST)
   Submit: RIFMAIN compile job.jcl
   Submit: RIFSTIM compile job.jcl
   ```

5. **Bind DB2 Objects:**
   ```
   Submit: RIF package binds.jcl
   Submit: RIF plan binds.jcl
   ```

6. **Install STC Procedure:**
   ```
   Copy: STC start procedure.jcl → PROCLIB(RIFA)
   ```

7. **Grant DB2 Permissions:**
   ```sql
   GRANT EXECUTE ON PACKAGE RIF.RIFMAIN TO <plan_owner>;
   GRANT EXECUTE ON PLAN RIF TO <execution_id>;
   GRANT INSERT ON RIF.* TO <execution_id>;
   GRANT SELECT ON SYSIBM.SYSDUMMY1 TO <execution_id>;
   GRANT EXECUTE ON PROCEDURE SYSPROC.ADMIN_INFO_SSID TO <execution_id>;
   GRANT EXECUTE ON PROCEDURE SYSPROC.ADMIN_INFO_IFCID TO <execution_id>;
   ```

8. **Start RIF:**
   ```
   S RIFA,optmember=<member>
   ```

### 7.3 Migration Considerations

**Prerequisites:**
- z/OS 2.1+ (for QWOSREAL field support)
- DB2 V11+ (for ADMIN_INFO_* stored procedures)
- Enterprise PL/I V4.5+
- HLASM 1.6+
- LE runtime

**Compatibility:**
- ✓ DB2 V11, V12, V13
- ✓ z/OS 2.1, 2.2, 2.3, 2.4, 2.5
- ⚠️ IFCID structures may change across DB2 versions - validate with IBM documentation

---

## SECTION 8: DOCUMENTATION QUALITY

### 8.1 Strengths

| Aspect | Quality | Notes |
|--------|---------|-------|
| Inline Code Comments | Good | Procedures well-documented, logic explained |
| External Documentation | Good | README, guides, glossary present |
| DDL Documentation | Good | Instructions in SQL comments |
| JCL Documentation | Fair | Basic comments, could be enhanced |
| Structure Definitions | Excellent | IBM IFCID mappings well-documented |
| Configuration | Fair | Options file has examples but limited explanation |

### 8.2 Recommendations

1. **Add API Documentation:**
   - Document each processor module's inputs/outputs
   - Create data dictionary for all IFCID fields
   - Document table schema with business meaning of each column

2. **Enhance Operational Documentation:**
   - Create troubleshooting guide with common error scenarios
   - Document recovery procedures
   - Create performance tuning guide

3. **Add Development Documentation:**
   - Create template for adding new IFCID processors (INSTEMPL exists but document its use)
   - Document build/test procedures
   - Create contribution guide

---

## APPENDIX A: STATISTICS COLLECTED

### IFCID 1 - System Statistics (Every Minute)

**QWSA - Address Space CPU Usage:**
- QWSAPSRB: Preemptable SRB CPU time (CP)
- QWSAEJST: Job step TCB CPU time
- QWSASRBT: SRB CPU time
- QWSAIIPT: I/O interrupt processing CPU time
- QWSAPSRB_ZIIP: Preemptable SRB CPU time (zIIP)

**QJST - Log Manager:**
- Write requests (wait, nowait, force)
- Buffer waits, log reads (buffer, active, archive)
- BSDS access, CI offloads
- Archive operations, log write I/O

**QWOS - z/OS Statistics:**
- CPU count, utilization (LPAR, DB2, MSTR, DBM1)
- Page-in rates
- Storage (real, virtual, free, used)

### IFCID 2 - Database Statistics (Every Minute)

**Q8ST - IDAA (IBM Db2 Analytics Accelerator):**
- Connection, request, timeout, failure counts
- Bytes/messages/blocks/rows sent/received
- CPU times, elapsed times
- Query counts by type

**QBST - Buffer Pool Activity:**
- Getpages, read I/O, write I/O
- Hit ratios (data, index)
- Prefetch, sequential detection
- Deferred writes, castouts
- Group buffer pool activity

**QIST - Data Manager:**
- RID pool usage
- Sort pool usage  
- Work file usage
- Index page splits
- Image copy activity

**QBGL - Group Buffer Pool:**
- Cross-invalidations
- Page writes/reads to CF
- XI requests, notify operations
- Directory entries

**QTXA - Local Lock Manager:**
- Lock/unlock requests
- Deadlocks, timeouts
- Suspensions, escalations
- Lock changes

**QTGS - Global Lock Manager (IRLM):**
- False contentions
- Global lock requests
- Propagation events
- Notify operations

**QTST - Service Controller:**
- Allocation requests
- Authorization checks
- Dataset operations

**QXST - SQL Activity:**
- SELECT, INSERT, UPDATE, DELETE counts
- PREPARE, OPEN, CLOSE, FETCH counts
- DDL operations (CREATE, DROP, ALTER)
- COMMIT, ROLLBACK
- Parallel degree statistics

### IFCID 225 - Address Space Memory (Every Minute)

**QW02251 - Address Space Summary:**
- Virtual storage (allocated, free, getmain)
- Real storage (pages in real, aux slots)
- Storage limits (region, extended)
- Private area storage

---

## APPENDIX B: PERFORMANCE CONSIDERATIONS

### Data Volume Estimates

**Per Minute (all gather flags enabled):**
- 12 INSERT operations
- ~50-100 KB of data
- 12 COMMIT operations

**Per Hour:**
- 720 INSERT operations
- 3-6 MB of data

**Per Day:**
- 17,280 INSERT operations
- 72-144 MB of data

**Storage Planning:**
- 19-day retention (based on default partitions): ~1.5-3 GB
- Recommend: Automate partition rotation daily
- Archive old partitions to offline storage

### DB2 Workload Impact

**Low Impact:**
- 1 CAF connection (minimal thread overhead)
- ADMIN_INFO_IFCID SP executes in DB2 (low overhead - reads SMF buffers)
- CS isolation (minimal locking)

**Recommendations:**
- Schedule during low-activity periods if DB2 is resource-constrained
- Monitor CGTT space in WORKFILE database (ADMIN_INFO_IFCID usage)
- Consider separate DB2 member for reporting queries (data sharing)

---

## APPENDIX C: GLOSSARY

### Mainframe & z/OS Terms

**AMODE (Addressing Mode)**: Specifies whether a program uses 24-bit or 31-bit addressing (AMODE 24 or AMODE 31).

**ASID (Address Space Identifier)**: Unique identifier assigned to each address space by z/OS.

**ATTACH**: z/OS service to create a new subtask (TCB) within an address space.

**CAF (Call Attach Facility)**: DB2 interface that allows batch programs to connect to DB2 without using TSO or CICS.

**CIB (Command Input Buffer)**: MVS control block containing console commands directed to an address space.

**DETACH**: z/OS service to terminate a subtask previously created with ATTACH.

**DSA (Dynamic Save Area)**: Language Environment storage area for saving register contents.

**ECB (Event Control Block)**: 4-byte control block used for task synchronization via WAIT/POST.

**EXTRACT**: MVS macro to obtain system information, including communication area addresses.

**IFCID (Instrumentation Facility Component Identifier)**: DB2 trace record type containing performance/statistics data.

**LE (Language Environment)**: IBM common runtime environment for multiple HLLs (COBOL, PL/I, C/C++).

**LPAR (Logical Partition)**: Subdivision of mainframe hardware resources into separate virtual systems.

**POST**: MVS service (SVC 2) to signal completion of an event by updating an ECB.

**QEDIT**: MVS macro for managing Command Input Buffers (CIBs).

**RMODE (Residency Mode)**: Specifies whether a load module must reside below or above 16MB line.

**SRB (Service Request Block)**: z/OS dispatchable unit that runs in supervisor state.

**STC (Started Task)**: z/OS job initiated via START command, runs continuously until stopped.

**SVC (Supervisor Call)**: Hardware instruction that invokes z/OS system services.

**TCB (Task Control Block)**: z/OS control block representing a dispatchable unit of work.

**WAIT**: MVS service (SVC 1) to suspend task execution until an ECB is posted.

**WTO (Write To Operator)**: MVS macro to send messages to the system console.

### DB2 Terms

**CGTT (Created Global Temporary Table)**: Work table created in WORKFILE database for query processing.

**DBRM (Database Request Module)**: Output from DB2 precompiler containing SQL statements and bind information.

**GBP (Group Buffer Pool)**: Shared buffer pool in coupling facility for DB2 data sharing.

**IDAA (IBM Db2 Analytics Accelerator)**: Hardware accelerator for complex DB2 queries using Netezza technology.

**IRLM (Internal Resource Lock Manager)**: Component managing locks in DB2, especially for data sharing.

**SP (Stored Procedure)**: Precompiled SQL program stored in DB2 and executed via CALL statement.

**SSID (Subsystem Identifier)**: 4-character name identifying a DB2 subsystem (e.g., DB2A, DB2P).

### Application-Specific Terms

**Delta Calculation**: Computing the difference between current and previous metric values to get interval statistics.

**Gather Flag**: Configuration option controlling whether a specific IFCID processor is active.

**Processor Module**: PL/I include file that parses and stores a specific IFCID subrecord type.

**RIF**: Resource Interface Facility - the application name for this DB2 statistics collector.

**Shutdown Flag**: Single-byte indicator set by command listener to signal main task to terminate.

### Performance & Statistics Terms

**Buffer Pool**: Memory area in DB2 for caching data and index pages.

**Hit Ratio**: Percentage of page requests satisfied from buffer pool vs. requiring I/O.

**Log Manager**: DB2 component managing write-ahead logging and recovery.

**Partition**: Subdivision of a table based on range of values, enabling data lifecycle management.

**Prefetch**: DB2 optimization reading multiple pages in anticipation of sequential access.

**zIIP (z Integrated Information Processor)**: Specialized processor for eligible DB2 and Java workloads.

---

**Analysis Generated:** Based on source code inspection of RIF application  
**Analysis Version:** 1.0  
**Document Date:** 2025-11-05  
**Analyst Note:** This analysis follows the structure and depth of BMC AMI DevX DevEnterprise reporting style, providing comprehensive static analysis of the RIF mainframe application.
