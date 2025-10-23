# RIF Terminology Glossary
*Essential Terms for Modern Developers New to Mainframe/DB2/PL/I*

## 🎯 How to Use This Glossary
**Format:** Term | Modern Equivalent | Definition | RIF Context

## 🖥️ Mainframe/z/OS Terms

### System Architecture
**Address Space** | Process | A virtual memory environment where programs execute | RIF runs in its own address space

**ASID (Address Space Identifier)** | Process ID | Unique identifier for each address space | Used in CPU metrics to identify processes

**ASCB (Address Space Control Block)** | Process Control Block | Control structure containing address space information | Referenced in QWSA CPU data

**ECB (Event Control Block)** | Semaphore/Event | Synchronization mechanism for task coordination | Used for timer events and stop commands

**JES (Job Entry Subsystem)** | Job Scheduler | Manages batch job execution and started tasks | Controls RIF started task lifecycle

**LPAR (Logical Partition)** | Virtual Machine | Hardware partitioning of mainframe resources | RIF runs within specific LPAR

**MVS (Multiple Virtual Storage)** | Operating System Kernel | Core of z/OS operating system | Provides services RIF depends on

**SMF (System Management Facilities)** | System Logging | z/OS system event and performance logging | Source of some performance data

**STC (Started Task)** | System Service/Daemon | Long-running system or application process | RIF runs as started task

**TCB (Task Control Block)** | Thread Control Block | Control structure for dispatchable work | CPU time measured at TCB level

**WLM (Workload Manager)** | Resource Manager | z/OS component that manages system resources | Affects RIF performance characteristics

**zIIP (z Integrated Information Processor)** | Specialized CPU | Special processor for eligible DB2/Java workloads | RIF measures zIIP CPU usage

### Storage and I/O
**DASD (Direct Access Storage Device)** | Disk Drive | Mainframe disk storage | Where DB2 databases reside

**Dataset** | File | Named collection of data records | Source code stored in datasets

**DSORG (Dataset Organization)** | File System Type | How data is organized within dataset | PDS for source, PS for data

**PDS (Partitioned Dataset)** | Directory/Folder | Dataset containing multiple named members | Contains RIF source code modules

**PDSE (Partitioned Dataset Extended)** | Advanced Directory | Enhanced version of PDS with better performance | Modern replacement for PDS

**RACF (Resource Access Control Facility)** | Access Control System | Mainframe security and authorization system | Controls RIF dataset/DB2 access

**VSAM (Virtual Storage Access Method)** | Indexed File System | High-performance data access method | Used by DB2 for data storage

### Job Control and Operations
**DD (Data Definition)** | File Handle/Path | JCL statement defining dataset allocation | Maps logical names to physical datasets

**JCL (Job Control Language)** | Shell Script/Makefile | Language for defining batch jobs and procedures | Used to build and run RIF

**PROC (Procedure)** | Startup Script | Predefined JCL for common operations | RIF startup procedure

**STEPLIB** | Library Path/LD_LIBRARY_PATH | Search path for executable load modules | Tells system where to find RIF programs

## 🗄️ DB2 Terms

### Architecture and Components  
**CAF (Call Attach Facility)** | Database Connection Pool | Interface for program-to-DB2 connection | RIF's method to connect to DB2

**DB2 Subsystem** | Database Instance | Complete DB2 environment | Target of RIF monitoring

**DBRM (Database Request Module)** | Compiled SQL | Output of SQL precompiler containing SQL statements | Created during RIF build process

**IFI (Instrumentation Facility Interface)** | Performance API | Interface for accessing DB2 performance data | Primary data source for RIF

**IFCID (Instrumentation Facility Component Identifier)** | Performance Event Type | Specific type of performance record | RIF processes IFCIDs 1, 2, and 225

**Package** | Prepared Statement Set | Executable form of SQL statements | RIF SQL packaged for execution

**Plan** | Execution Context | Access path and authorization for SQL execution | RIF plan grants database access

**SPUFI (SQL Processor Using File Input)** | SQL Query Tool | Interactive SQL execution utility | Used for testing RIF database queries

**SSID (Subsystem Identifier)** | Database Name | 4-character name identifying DB2 subsystem | Configured in RIFOPT

### Performance and Monitoring
**Buffer Pool** | Database Cache | Memory area for caching database pages | Key performance metric collected by RIF

**IRLM (IMS Resource Lock Manager)** | Lock Manager | Manages locking for data integrity | Source of locking statistics

**QMF (Query Management Facility)** | Report Writer | DB2 query and reporting tool | Alternative to SPUFI for data analysis

**RID (Record Identifier)** | Row Pointer | Physical pointer to database record | Used in access path optimization

**Tablespace** | Table Container | Physical storage container for tables | RIF tables stored in RIF tablespace

## 🔧 PL/I Language Terms

### Data Types and Structures
**BASED Variable** | Pointer-based Variable | Variable mapped to memory location via pointer | Used for parsing IFI records

**BIN FIXED** | Integer | Binary fixed-point number | Standard integer type in RIF

**CHAR VARYING** | String | Variable-length character string | Used for text data in RIF

**CONTROLLED Storage** | Managed Memory | Explicitly allocated/freed memory | Manual memory management

**DCL (Declare)** | Variable Declaration | Statement to define variables and structures | All RIF variables declared with DCL

**PICTURE** | Formatted Number | Numeric variable with specific format | Used for decimal calculations

**UNION** | Variant Record | Multiple variable layouts sharing memory | Used for data type conversions

### Control Flow and Procedures  
**ATTACH** | Thread Creation | Start a new concurrent task | Creates timer thread in RIF

**DO WHILE** | While Loop | Iterative control structure | Main processing loop in RIF

**ON Condition** | Exception Handler | Error handling mechanism | SQL error handling in RIF

**PROC (Procedure)** | Function/Method | Reusable code module | RIF organized into procedures

**SELECT/WHEN** | Switch/Case | Multi-way branching statement | Configuration processing in RIF

### Built-in Functions
**ADDR** | Address-of Operator | Returns memory address of variable | Used extensively for pointers

**HEX** | Hex Conversion | Convert data to hexadecimal representation | Debugging output in RIF

**LENGTH** | String Length | Returns length of string variable | Used in data processing

**SUBSTR** | Substring | Extract portion of string | Text processing in RIF

**TRIM** | String Trim | Remove trailing spaces | Output formatting in RIF

## ⚙️ Assembly Language Terms

### Registers and Instructions
**General Registers (R0-R15)** | CPU Registers | Hardware registers for computation | Parameter passing and computation

**Base Register** | Address Base | Register used for address calculation | Typically R12 in RIF modules

**CSECT (Control Section)** | Program Module | Independently relocatable program unit | Each assembly module is a CSECT

**DSECT (Dummy Section)** | Data Template | Template for mapping data structures | Parameter list definitions

**USING** | Address Resolution | Establish addressability for labels | Tell assembler which register maps to addresses

### System Services
**ATTACH** | Process Creation | Create new address space or task | System service wrapped by RIF

**EXTRACT** | Get System Information | Retrieve system control blocks | Used for console communication

**POST** | Signal Event | Set ECB to signaled state | Wake up waiting tasks

**QEDIT** | Queue Management | Manage system command queues | Console command processing

**WAIT** | Wait for Event | Suspend until ECB is posted | Synchronization primitive

**WTO (Write to Operator)** | Console Message | Send message to system console | Error reporting and status

## 📊 Performance Metrics Terminology

### CPU Measurements  
**CPU Time Units** | Time Measurement | CPU time measured in timer units (1/4096 second) | Converted to seconds in RIF

**EJST Time** | Job Step Time | CPU time consumed by job step TCB | Application CPU usage

**PSRB Time** | Preemptable SRB Time | CPU time for preemptable service requests | General system CPU

**SRB (Service Request Block)** | Kernel Thread | Non-preemptable system service routine | System overhead measurement

**zIIP Time** | Specialty Engine Time | CPU time on z Integrated Information Processor | Cost-saving processor usage

### Storage Measurements
**Frame** | Memory Page | 4KB unit of real storage | Physical memory measurement

**Page** | Virtual Memory Page | 4KB unit of virtual storage | Virtual memory usage

**Slot** | Swap Space Unit | Unit of auxiliary storage allocation | Paging space measurement

### Database Measurements
**Buffer Pool Hit Ratio** | Cache Hit Rate | Percentage of page requests satisfied from memory | Key performance indicator

**GETPAGE** | Page Request | Request for database page | Buffer pool activity metric

**Physical I/O** | Disk Read/Write | Actual disk I/O operation | Performance bottleneck indicator

**Logical I/O** | Buffer Access | Access to page in buffer pool | Application activity measure

## 🔄 Operational Terms

### Job Management
**Abend** | Abnormal Termination | Program crash or failure | Error condition requiring investigation

**Completion Code** | Return Code | Numeric result of program execution | Success/failure indicator

**Return Code** | Exit Status | Program completion status | 0 = success, others = error

**Step** | Job Phase | Individual execution unit within job | Compile, link, execute phases

### System Management  
**IPL (Initial Program Load)** | System Boot | Process of starting z/OS system | Affects system-wide statistics

**Quiesce** | Graceful Shutdown | Orderly shutdown of system component | Preferred shutdown method

**SDSF (System Display and Search Facility)** | Job Monitor | Interface for viewing job status and output | Monitor RIF execution

**VTAM (Virtual Telecommunications Access Method)** | Network Manager | z/OS networking component | Affects system performance

## 🛠️ Development Tools

### Editors and Utilities
**ISPF (Interactive System Productivity Facility)** | IDE/Editor | Main development interface for mainframe | Primary tool for RIF development

**PDF (Program Development Facility)** | Development Environment | ISPF application for program development | Edit, compile, test environment

**SCLM (Software Configuration and Library Manager)** | Version Control | IBM's source control system | Alternative to modern Git

### Compilers and Processing
**HLASM (High Level Assembler)** | Assembly Compiler | IBM's assembly language compiler | Compiles RIF assembly modules

**Linkage Editor** | Linker | Combines object modules into executable | Creates RIF load modules

**Precompiler** | SQL Preprocessor | Processes embedded SQL statements | Converts SQL to COBOL/PL/I calls

## 🌐 Integration and Connectivity

### Data Access
**IDAA (IBM DB2 Analytics Accelerator)** | Analytics Engine | Hardware accelerator for DB2 queries | Performance monitoring target

**RRS (Resource Recovery Services)** | Transaction Coordinator | Coordinates distributed transactions | Ensures data consistency

**XCF (Cross-System Coupling Facility)** | Inter-system Communication | Communication between z/OS systems | Data sharing coordination

### Monitoring and Management
**RMF (Resource Measurement Facility)** | System Monitor | z/OS performance monitoring tool | Complementary to RIF

**Tivoli** | Enterprise Management | IBM system management suite | Enterprise monitoring integration

**OMEGAMON** | Application Monitor | IBM application performance monitor | Commercial alternative to RIF

---

## 🔍 Quick Reference by Context

### When Reading RIF Code:
- **DCL** = variable declaration  
- **EXEC SQL** = database operation
- **CALL** = procedure invocation
- **ADDR()** = memory address
- **BASED** = pointer-referenced data

### When Building RIF:
- **JCL** = build script
- **PROC** = execution procedure  
- **STEPLIB** = runtime library path
- **DBRM** = compiled SQL
- **CSECT** = program module

### When Running RIF:
- **STC** = running service
- **ECB** = synchronization object
- **POST** = wake up waiting task
- **ATTACH** = create thread
- **ABEND** = program crash

### When Analyzing Data:
- **IFCID** = performance record type
- **Buffer Pool** = database cache
- **Hit Ratio** = cache efficiency  
- **CPU Time** = processor usage
- **Lock** = data access control

---

*This glossary bridges the gap between modern development concepts and mainframe terminology, enabling faster comprehension of RIF architecture and operations.*