# RIF Developer Onboarding Guide
*Essential Knowledge for Modern Developers New to Mainframe/DB2/PL/I*

## 🎯 Purpose
This guide helps developers with modern programming backgrounds (Java, Python, cloud platforms) understand the RIF application and mainframe development environment.

## 📋 Prerequisites Check
**Before starting RIF development, ensure you have:**

### System Access
- [ ] TSO/ISPF logon capability
- [ ] Access to RIF source datasets (HLQ.RIFSRC, etc.)
- [ ] DB2 subsystem access for testing
- [ ] Authority to submit batch jobs
- [ ] SDSF access for job monitoring

### Development Tools
- [ ] ISPF/PDF for editing (or modern editor with upload capability)
- [ ] File transfer capability (IND$FILE, FTP, or similar)
- [ ] DB2 query tool (SPUFI, QMF, or similar)
- [ ] JCL knowledge or reference materials

## 🏗️ Development Environment Setup

### 1. Understanding the File System
```
Modern Equivalent          Mainframe Reality
├── Git Repository    →    PDS (Partitioned Dataset)
├── src/              →    HLQ.RIFSRC (source members)
├── build/            →    Temporary datasets
├── target/           →    HLQ.RIFLOAD (executables)
└── config/           →    HLQ.RIFOPTS (configuration)
```

### 2. Source Code Organization
```
Dataset.Member              Purpose
├── HLQ.RIFSRC(RIFMAIN)    → Main program controller
├── HLQ.RIFSRC(RIFSTIM)    → Timer thread handler
├── HLQ.RIFSRC(RIFDB2)     → DB2 interface routines
├── HLQ.RIFSRC(RIFCMND)    → Command processing
├── HLQ.RIFSRC(RIFOPTS)    → Configuration management
├── HLQ.RIFSRC(RIFQ*)      → Data processors (one per metric type)
├── HLQ.RIFASM(RIF*)       → Assembly language modules
└── HLQ.RIFJCL(*)          → Build and execution JCL
```

### 3. Build Process Walkthrough
```bash
# Modern Build (what you're used to)
mvn clean compile package

# Mainframe Build (what you need to learn)
# Step 1: Submit PL/I compilation JCL
# Step 2: Submit Assembly compilation JCL  
# Step 3: Submit Link-edit JCL
# Step 4: Update runtime PROC
# Step 5: Test with START command
```

## 🔧 Essential Mainframe Concepts

### Event Control Blocks (ECBs)
**Modern Equivalent:** Semaphores/Events
```pli
/* PL/I ECB Usage in RIF */
dcl TIMER_ECB type ECB;              // Event object
call clearECB(TIMER_ECB);           // Reset event
call WAIT(addr(TIMER_ECB));         // Wait for event
call POST(addr(TIMER_ECB));         // Signal event
```

### ATTACH/DETACH (Threading)
**Modern Equivalent:** Thread creation
```pli
/* Start a new thread */
attach RIFSTIM(addr(parameters)) thread(timer_handle);
/* Wait and clean up */
detach thread(timer_handle);
```

### Dataset I/O
**Modern Equivalent:** File operations
```pli
/* File I/O in PL/I */
dcl config_file file record input;
open file(config_file);
read file(config_file) into(record_structure);
close file(config_file);
```

## 🗄️ DB2 z/OS Fundamentals

### CAF (Call Attach Facility) vs. JDBC
```pli
/* CAF Connection (what RIF uses) */
dcl 1 caf_parms,
  3 function char(12) init('OPEN'),
  3 ssid     char(4),  
  3 plan     char(8)  init('RIF'),
  3 retcode  bin fixed(31),
  3 reason   char(4);

call dsnali(caf_parms);  // Connect to DB2

/* Compare to JDBC (what you know) */
Connection conn = DriverManager.getConnection(url, user, pass);
```

### IFI Data Access Pattern
```pli
/* Get performance data via stored procedure */
exec sql call SYSPROC.ADMIN_INFO_IFCID(:ifcid_number);

/* Process binary data records */
base_ptr = addr(raw_data) + offset;
cpu_time = base_ptr->qwsa.qwsapsrb - saved_values.qwsapsrb;
```

## 🎮 Hands-On Exercise: Making Your First Change

### Exercise 1: Add Debug Output
**Goal:** Understand the development cycle

1. **Edit RIFMAIN source:**
```pli
/* Add after line ~180 in RIFMAIN */
if (debug_mode = '1'b) then do;
  put skip list(time() !! ' DEBUG: Starting main work loop');
end;
```

2. **Submit compilation JCL:**
```jcl
//MYCOMP JOB ...
// EXEC procedure from RIFMAIN-compile-job.jcl
```

3. **Check for errors in job output**

4. **Test with debug mode ON in RIFOPTS**

### Exercise 2: Modify Configuration Option
**Goal:** Understand configuration processing

1. **Add new option to RIFOPTS.pli**
2. **Update configuration file**
3. **Test the change**

## 🐛 Common Debugging Scenarios

### Problem: Compilation Errors
```
SOLUTION PATTERN:
1. Check SYSPRINT for error messages
2. Verify INCLUDE/COPY member availability  
3. Check syntax against PL/I reference
4. Ensure all DCL statements are properly terminated
```

### Problem: Runtime Abends
```
SOLUTION PATTERN:
1. Check job output for abend codes (S0C1, S0C4, etc.)
2. Look for SQL error codes in application output
3. Verify dataset allocation in job step
4. Check storage usage and limits
```

### Problem: Data Not Collecting
```
SOLUTION PATTERN:
1. Verify DB2 subsystem is active
2. Check IFI authorization 
3. Confirm RIFOPT settings
4. Review SQL error handling logic
```

## 📖 Essential Reference Materials

### Quick Reference Cards
- **PL/I Language:** Built-in functions, syntax patterns
- **SQL/DB2:** Error codes, performance SQL
- **z/OS Commands:** START/STOP/DISPLAY commands
- **JCL:** Common job control patterns

### Online Resources
- **IBM Knowledge Center:** Official documentation
- **IBM Redbooks:** Practical implementation guides
- **SHARE Conference:** User group presentations
- **Stack Overflow:** mainframe-specific tags

## 🚀 Next Steps After Onboarding

### Week 1-2: Environment Mastery
- [ ] Navigate ISPF comfortably
- [ ] Edit and compile a simple program
- [ ] Submit and monitor batch jobs
- [ ] Browse datasets and member lists

### Month 1: RIF Understanding  
- [ ] Trace through main processing loop
- [ ] Understand each data processor module
- [ ] Modify configuration options
- [ ] Add simple debug statements

### Month 2-3: Enhancement Capability
- [ ] Add new metric collection
- [ ] Modify database schema
- [ ] Enhance error handling
- [ ] Performance tuning awareness

### Month 4-6: Modernization Planning
- [ ] Identify modernization candidates
- [ ] Prototype API interfaces
- [ ] Design cloud migration strategy
- [ ] Document business requirements

## ⚠️ Critical Gotchas for Modern Developers

### 1. Case Sensitivity
- **Mainframe:** Generally uppercase, member names are case-sensitive
- **Modern:** Usually case-sensitive throughout

### 2. Character Sets
- **Mainframe:** EBCDIC character encoding
- **Modern:** ASCII/UTF-8 encoding
- **Impact:** File transfers require conversion

### 3. Memory Management
- **PL/I:** Manual storage management with CONTROLLED/BASED variables
- **Modern:** Automatic garbage collection
- **Impact:** Memory leaks are possible and dangerous

### 4. Error Handling
- **PL/I:** ON condition handlers (not try/catch)
- **Modern:** Exception-based error handling
- **Impact:** Different debugging approaches needed

### 5. Build Dependencies
- **Mainframe:** JCL-based, sequential job steps
- **Modern:** Parallel builds with dependency management
- **Impact:** Longer build cycles, more complex troubleshooting

## 🏆 Success Metrics

**You'll know you're succeeding when you can:**
- [ ] Navigate the RIF codebase confidently
- [ ] Make configuration changes without fear
- [ ] Debug compilation and runtime issues
- [ ] Explain the business value of each metric
- [ ] Design enhancements that fit the architecture
- [ ] Estimate effort for modernization initiatives

---
*Remember: The mainframe is different, not inferior. Many patterns that seem archaic exist for good reasons related to reliability, performance, and backwards compatibility.*