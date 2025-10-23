# RIF Troubleshooting Guide
*Common Issues and Solutions for New Mainframe Developers*

## 🚨 Emergency Response Quick Reference

### RIF Not Starting
**Symptoms:** START RIF command fails or task abends immediately

**Immediate Actions:**
1. Check console messages for abend codes
2. Verify PROC library contains RIF procedure
3. Check STEPLIB dataset availability
4. Confirm DB2 subsystem is active

**Common Causes & Solutions:**
```
S806 Abend → Load module not found
Solution: Verify HLQ.RIFLOAD contains RIFMAIN
Command: LISTDS 'HLQ.RIFLOAD' MEMBERS

S80A Abend → Authorization failure  
Solution: Check RACF permissions for datasets
Command: Contact security administrator

IEF236I → Dataset not found
Solution: Verify all DD statements in PROC
Action: Check dataset names match actual allocations
```

### RIF Running But No Data Collected
**Symptoms:** Task appears active but no database inserts

**Diagnostic Steps:**
1. Check SYSPRINT output for SQL errors
2. Verify DB2 connection status
3. Review RIFOPT configuration settings
4. Confirm IFI authorization

**Common Issues:**
```
SQLCODE -551 → Insufficient DB2 privileges
Solution: GRANT SELECT,INSERT on RIF tables to plan authorization ID

SQLCODE -805 → Package not bound
Solution: Run package bind JCL jobs

RIFOPT misconfiguration → No metrics enabled
Solution: Review gather_* options in RIFOPTS dataset
```

## 🔧 Compilation Issues

### PL/I Compilation Errors

#### Syntax Errors
```pli
/* ERROR: Undeclared variable */
IEW2456 IDENTIFIER 'VARIABLE_NAME' NOT DECLARED

Solution: Add declaration
dcl variable_name bin fixed(31);

/* ERROR: Missing semicolon */
IEW2308 SYNTAX ERROR

Solution: Check for missing semicolons, especially after END statements
do while(condition);
  /* statements */
end; /* <- Don't forget this semicolon */
```

#### Include/Copy Errors
```
ERROR: IEW2461 MEMBER 'RIFQWSA' NOT FOUND IN INCLUDE LIBRARY

Solution: Verify SYSLIB DD statement includes correct datasets
Check: INCDD DD DSN=HLQ.RIFSRC,DISP=SHR
```

#### SQL Preprocessing Errors
```
DSNH105E SQL STATEMENT NOT EXECUTABLE, SQLCODE = -204

Solution: Verify table exists and spelling is correct
Check DB2 catalog: SELECT * FROM SYSIBM.SYSTABLES WHERE NAME LIKE 'RIF%'
```

### Assembly Compilation Issues

#### Register Usage Errors
```assembly
ERROR: IEV009 UNDEFINED SYMBOL - R1

Solution: Ensure register equates are defined
R1 EQU 1
R2 EQU 2
```

#### Macro Expansion Issues  
```assembly
ERROR: IEV020 UNDEFINED OPERATION CODE - EXTRACT

Solution: Add required macro libraries to SYSLIB
//SYSLIB DD DSN=SYS1.MACLIB,DISP=SHR
//       DD DSN=SYS1.MODGEN,DISP=SHR
```

### Link-Edit Issues
```
ERROR: IEW2322 MEMBER RIFPOST NOT FOUND

Solution: Verify all assembly modules are compiled and available
Check: LISTDS 'HLQ.RIFLOAD' MEMBERS
```

## 🗄️ Database Issues

### Connection Problems

#### CAF Open Failures
**Error Pattern in SYSPRINT:**
```
RIF0001E - CAF retcode : 8
RIF0001E - CAF reascode : 00E30008
```

**Diagnosis:**
```
Retcode 8 → DB2 not available or plan issues
Reascode 00E30008 → Plan authorization failure

Solutions:
1. Verify DB2 subsystem is started: -DIS DB2
2. Check plan exists: SELECT * FROM SYSIBM.SYSPLAN WHERE NAME = 'RIF'
3. Verify plan authorization: GRANT EXECUTE ON PLAN RIF TO userid
```

#### SQL Execution Errors
```
Common SQLCODEs in RIF Context:

SQLCODE -204: Table not found
- Check table names in DDL match application
- Verify CREATE TABLE statements executed

SQLCODE -551: No privileges  
- GRANT INSERT, SELECT on all RIF tables
- Check package/plan authorization

SQLCODE -805: Package not found
- Run package bind jobs
- Verify CURRENT PACKAGE PATH setting

SQLCODE -911: Deadlock or timeout
- Review lock contention in application
- Check DB2 lock timeout settings
```

### Data Collection Issues

#### IFI Access Problems
```
Error: ADMIN_INFO_IFCID returns non-zero retcode

Diagnosis:
1. Check DB2 IFI authorization
2. Verify sufficient workfile database space  
3. Review DB2 system parameters

Solutions:
DB2 Command: -GRANT EXECUTE ON PROCEDURE ADMIN_INFO_IFCID TO userid
DB2 Parm: IRLM LOCK TIMEOUT = 60 (or appropriate value)
```

#### Performance Data Inconsistencies
```
Symptoms: Negative delta values, unrealistic metrics

Common Causes:
1. DB2 subsystem restart (counters reset)
2. IFI record format changes (DB2 upgrade)
3. First iteration not skipped properly

Solutions:
1. Monitor for DB2 start messages
2. Test after DB2 maintenance
3. Verify firstItteration logic
```

## 📊 Operational Issues

### Performance Problems

#### RIF Consuming Too Much CPU
**Symptoms:** High CPU usage by RIF task

**Investigation Steps:**
1. Check collection frequency (should be 1 minute)
2. Review enabled metrics in RIFOPT
3. Analyze SQL execution times
4. Monitor IFI response times

**Solutions:**
```
High Collection Frequency → Adjust timer intervals
Too Many Metrics → Disable unnecessary gather_* options  
SQL Performance → Add indexes, analyze access paths
IFI Delays → Check workfile database configuration
```

#### Memory Usage Issues
**Symptoms:** Storage abends (S80A, S878)

**Common Causes:**
```
S80A → Storage violation
- Check array bounds in PL/I code
- Review pointer arithmetic in based variables

S878 → Storage overlay
- Verify BASED variable alignment
- Check memory allocation/deallocation patterns
```

### Data Quality Issues

#### Missing Data Periods
**Symptoms:** Gaps in time-series data

**Investigation:**
1. Check RIF task status during gap period
2. Review SYSPRINT for error messages  
3. Analyze DB2 availability
4. Check storage space utilization

**Resolution:**
```
Task Stopped → Review operator actions, restart procedures
SQL Errors → Fix authorization, space, or connectivity issues
DB2 Unavailable → Coordinate with DB2 administration
Storage Full → Implement table space management
```

#### Incorrect Metric Values
**Symptoms:** Unrealistic performance numbers

**Common Issues:**
```
Negative Values → Counter rollover or restart not handled
Zero Values → Metric collection not configured
Huge Values → Unit conversion errors or data corruption

Investigation SQL:
SELECT MIN(metric), MAX(metric), AVG(metric)
FROM relevant_table 
WHERE INSERT_TS >= 'problem_timeframe';
```

## 🛠️ Development Environment Issues

### ISPF/TSO Problems

#### Dataset Access Issues
```
Error: DATASET NOT FOUND OR NOT ACCESSIBLE

Solutions:
1. Check dataset name spelling (case sensitive for non-standard names)
2. Verify dataset allocation: LISTDS 'dataset.name'
3. Check RACF permissions: Contact security team
4. Confirm dataset exists: LISTCAT ENTRIES('dataset.name')
```

#### Edit/Browse Limitations
```
Issue: Member appears empty or truncated

Causes:
1. Record format mismatch (FB vs VB)
2. Logical record length issues  
3. Dataset full or space issues

Solutions:
1. Check DCB attributes: LISTDS 'dataset.name' HISTORY
2. Allocate with correct attributes
3. Use IEBCOPY for member recovery if needed
```

### File Transfer Issues

#### ASCII/EBCDIC Conversion Problems
```
Symptoms: Special characters corrupted, compilation errors

Solutions:
1. Use proper transfer mode (ASCII for source, BINARY for load modules)
2. Configure translation tables correctly
3. Test with simple files first
4. Use IND$FILE or equivalent with correct options
```

## 🔍 Diagnostic Commands Reference

### z/OS System Commands
```bash
# Check task status  
D A,RIF                    # Display active RIF task
D J,RIF                    # Display job information

# System resources
D ASM                      # Display auxiliary storage manager
D SMS,STORGRP(groupname)   # Display storage group status
D TS,L=xx                  # Display tablespace status
```

### DB2 Commands  
```sql
-- Check DB2 status
-DISPLAY DB2                    -- DB2 subsystem status
-DISPLAY DATABASE(RIF)          -- RIF database status  
-DISPLAY THREAD(*)              -- Active threads
-DISPLAY PLAN(RIF)              -- Plan status

-- Performance monitoring
-DISPLAY BUFFERPOOL(BP2)        -- Buffer pool status
-DISPLAY LOCATION               -- Location information
-DISPLAY ARCHIVE               -- Archive log status
```

### JES Commands
```bash
# Job monitoring
$DJ,RIF                    # Display job in JES queue
$SI                        # System information
$HASP395                   # JES2 status messages
```

## 📋 Health Check Procedures

### Daily Verification Script
```sql
-- Verify RIF is collecting data
SELECT COUNT(*), MAX(INSERT_TS) 
FROM RIF.RIF_0001_CPU_DATA 
WHERE INSERT_TS >= CURRENT_TIMESTAMP - 2 HOURS;

-- Check for error patterns
SELECT COUNT(*) FROM RIF.APPLICATION_LOG 
WHERE MESSAGE_TEXT LIKE '%ERROR%' 
AND INSERT_TS >= CURRENT_TIMESTAMP - 24 HOURS;

-- Validate metric ranges
SELECT 'CPU' as METRIC, AVG(QWSAPSRB) as AVG_VALUE
FROM RIF.RIF_0001_CPU_DATA 
WHERE INSERT_TS >= CURRENT_TIMESTAMP - 1 HOUR
UNION
SELECT 'BUFFER_HITS', AVG(QBSTGET) 
FROM RIF.RIF_0002_BUFFER_POOL 
WHERE INSERT_TS >= CURRENT_TIMESTAMP - 1 HOUR;
```

### Weekly Maintenance Tasks
1. **Review SYSPRINT output** for accumulated messages
2. **Check dataset space utilization** for growth trends  
3. **Validate configuration changes** if any were made
4. **Test stop/restart procedures** during maintenance window
5. **Review performance trends** for capacity planning

---

## 🆘 Escalation Procedures

### Level 1: Self-Service (0-30 minutes)
- Check this troubleshooting guide
- Review recent changes
- Verify basic connectivity and authorization
- Attempt standard restart procedures

### Level 2: Team Support (30-120 minutes) 
- Engage mainframe systems team
- Involve DB2 administration if database-related
- Escalate to application development team
- Consider business impact assessment

### Level 3: Vendor Support (2+ hours)
- Open IBM PMR if DB2-related
- Contact hardware vendor if system-level
- Engage consulting resources if available
- Implement temporary workarounds

---

*Remember: Most RIF issues are environment-related (permissions, datasets, DB2 connectivity) rather than application logic problems. Start with the basics before diving into code analysis.*