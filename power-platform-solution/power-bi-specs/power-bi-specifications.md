# Power BI Dashboard Specifications

## Employee360 AI Lifecycle Management Platform - Analytics & Reporting

### Overview
Enterprise analytics solution providing real-time insights into employee lifecycle operations, compliance, and performance metrics.

---

## Dashboard Architecture

```
Employee360 Analytics Workspace
│
├── 01_Executive_Dashboard.pbix
├── 02_HR_Operations_Dashboard.pbix
├── 03_Compliance_Dashboard.pbix
├── 04_Department_Performance.pbix
└── 05_Audit_Analytics.pbix
```

---

## 1. Executive Dashboard

### Purpose
High-level KPIs and trends for executive leadership

### Data Sources
- SharePoint: LifecycleRequests, WorkflowTasks, Employees, ComplianceAlerts
- Connection: SharePoint Online List (DirectQuery for real-time)

### Pages

#### Page 1: Overview
**Layout**: KPI Cards + Charts

**KPI Cards** (Top row):
- Active Onboardings
  - Value: COUNT of LifecycleRequests WHERE Type='Onboarding' AND Status IN ('Submitted','In Progress')
  - Trend: vs last month
  - Color: Blue
  
- Active Offboardings
  - Value: COUNT of LifecycleRequests WHERE Type IN ('Resignation','Termination') AND Status IN ('Submitted','In Progress')
  - Trend: vs last month
  - Color: Orange
  
- Average Onboarding Duration
  - Value: AVG days between RequestDate and CompletionDate
  - Target: 14 days
  - Color: Green if <14, Red if >14
  
- SLA Compliance Rate
  - Value: (Completed within SLA / Total Completed) * 100
  - Target: >95%
  - Color: Green if >95%, Yellow if 90-95%, Red if <90%
  
- Open Compliance Alerts
  - Value: COUNT of ComplianceAlerts WHERE Status NOT IN ('Resolved','Closed')
  - Severity breakdown: Critical/High/Medium/Low
  - Color: Red if any Critical

**Charts**:

1. **Lifecycle Requests Trend** (Line chart)
   - X-axis: Month
   - Y-axis: Count of requests
   - Legend: Onboarding, Offboarding, Transfer
   - Period: Last 12 months

2. **SLA Performance** (Stacked bar chart)
   - X-axis: Month
   - Y-axis: Count
   - Stacks: On Time, At Risk, Overdue
   - Colors: Green, Yellow, Red

3. **Department Distribution** (Donut chart)
   - Values: Employee count by department
   - Labels: Department name + percentage

4. **Task Completion Rate** (Gauge)
   - Value: (Completed tasks / Total tasks) * 100
   - Target: 95%
   - Ranges: 0-70 (Red), 70-90 (Yellow), 90-100 (Green)

#### Page 2: Trends & Analysis
**Charts**:

1. **Onboarding Bottlenecks** (Funnel chart)
   - Stages: Initial Request → Approval → HR Processing → IT Setup → Completed
   - Shows dropout at each stage
   
2. **Average Time by Stage** (Bar chart)
   - X-axis: Stage name
   - Y-axis: Average days
   - Color: By department

3. **Automation Savings** (Card + Line)
   - Estimated hours saved by automation
   - Calculation: (TasksCompleted * AvgTimePerTask) - ActualEffort
   - Trend over time

4. **Employee Headcount Trend** (Area chart)
   - X-axis: Month
   - Y-axis: Headcount
   - Lines: New hires, Terminations, Net change
   - Period: Last 24 months

### DAX Measures

```dax
// Active Onboardings
Active Onboardings = 
CALCULATE(
    COUNTROWS('LifecycleRequests'),
    'LifecycleRequests'[LifecycleType] = \"Onboarding\",\    'LifecycleRequests'[RequestStatus] IN {\"Submitted\", \"In Progress\", \"Pending Approval\"}
)

// Average Onboarding Duration
Avg Onboarding Duration = 
VAR CompletedOnboardings = 
    FILTER(
        'LifecycleRequests',
        'LifecycleRequests'[LifecycleType] = \"Onboarding\" &&
        'LifecycleRequests'[RequestStatus] = \"Completed\" &&
        NOT(ISBLANK('LifecycleRequests'[CompletionDate]))
    )
RETURN
    AVERAGEX(
        CompletedOnboardings,
        DATEDIFF(
            'LifecycleRequests'[RequestDate],
            'LifecycleRequests'[CompletionDate],
            DAY
        )
    )

// SLA Compliance Rate
SLA Compliance % = 
VAR TotalCompleted = 
    CALCULATE(
        COUNTROWS('LifecycleRequests'),
        'LifecycleRequests'[RequestStatus] = \"Completed\"
    )
VAR CompletedOnTime = 
    CALCULATE(
        COUNTROWS('LifecycleRequests'),
        'LifecycleRequests'[RequestStatus] = \"Completed\",
        'LifecycleRequests'[CompletionDate] <= 'LifecycleRequests'[SLADueDate]
    )
RETURN
    DIVIDE(CompletedOnTime, TotalCompleted, 0) * 100

// Critical Alerts Count
Critical Alerts = 
CALCULATE(
    COUNTROWS('ComplianceAlerts'),
    'ComplianceAlerts'[Severity] = \"Critical\",
    'ComplianceAlerts'[AlertStatus] NOT IN {\"Resolved\", \"Closed\"}
)

// Task Completion Rate
Task Completion Rate = 
VAR TotalTasks = COUNTROWS('WorkflowTasks')
VAR CompletedTasks = 
    CALCULATE(
        COUNTROWS('WorkflowTasks'),
        'WorkflowTasks'[TaskStatus] = \"Completed\"
    )
RETURN
    DIVIDE(CompletedTasks, TotalTasks, 0) * 100

// Automation Savings (Hours)
Automation Savings = 
SUMX(
    FILTER(
        'AuditTrail',
        'AuditTrail'[IsAutomated] = TRUE()
    ),
    // Assuming 15 minutes saved per automated action
    0.25
)

// Month over Month Change
MoM Change = 
VAR CurrentMonth = [Active Onboardings]
VAR PreviousMonth = 
    CALCULATE(
        [Active Onboardings],
        DATEADD('Calendar'[Date], -1, MONTH)
    )
RETURN
    DIVIDE(CurrentMonth - PreviousMonth, PreviousMonth, 0) * 100
```

---

## 2. HR Operations Dashboard

### Purpose
Detailed operational metrics for HR team

### Pages

#### Page 1: Operations Overview

**Visuals**:

1. **Pending Actions** (Table)
   - Columns: Request ID, Employee, Type, Current Stage, Days Open, Assigned To
   - Filter: Status != 'Completed'
   - Conditional formatting: Red if >SLA

2. **Task Distribution by Department** (Stacked column chart)
   - X-axis: Department
   - Y-axis: Task count
   - Stacks: Not Started, In Progress, Completed
   
3. **Top 10 Overdue Tasks** (Table with bars)
   - Columns: Task Name, Assigned To, Days Overdue
   - Bar: Days overdue (conditional color)

4. **My Team Performance** (Matrix)
   - Rows: Team member
   - Columns: Tasks Assigned, Completed, Avg Completion Time
   - Conditional formatting: Traffic light

#### Page 2: Onboarding Analytics

**Visuals**:

1. **Onboarding Pipeline** (Waterfall chart)
   - Shows onboarding stages
   - Displays time spent in each stage
   
2. **Onboarding by Department** (Clustered bar)
   - Y-axis: Department
   - X-axis: Count
   - Colors: By status

3. **New Hire Satisfaction** (Gauge) [If feedback collected]
   - Value: Average satisfaction score
   - Target: 4.5/5

4. **Documentation Compliance** (Progress bar)
   - All required documents collected %

#### Page 3: Offboarding Analytics

**Visuals**:

1. **Offboarding Reasons** (Pie chart)
   - Slices: Resignation, Termination, Retirement, Other
   
2. **Equipment Return Status** (KPI + Table)
   - KPI: % returned on time
   - Table: Pending returns with days overdue

3. **Access Revocation Status** (KPI)
   - % of terminated employees with all access revoked

4. **Exit Interview Completion** (Gauge)
   - % of offboarded employees with completed exit interview

### DAX Measures

```dax
// Overdue Tasks Count
Overdue Tasks = 
CALCULATE(
    COUNTROWS('WorkflowTasks'),
    'WorkflowTasks'[TaskStatus] <> \"Completed\",
    'WorkflowTasks'[DueDate] < TODAY()
)

// Average Task Completion Time
Avg Task Completion = 
AVERAGEX(
    FILTER(
        'WorkflowTasks',
        'WorkflowTasks'[TaskStatus] = \"Completed\" &&
        NOT(ISBLANK('WorkflowTasks'[CompletedDate]))
    ),
    DATEDIFF(
        'WorkflowTasks'[Created],
        'WorkflowTasks'[CompletedDate],
        HOUR
    )
)

// Equipment Return Compliance
Equipment Return Compliance % = 
VAR TotalReturns = 
    CALCULATE(
        COUNTROWS('EquipmentRegister'),
        'EquipmentRegister'[AssetStatus] IN {\"Pending Return\", \"Available\"}
    )
VAR OnTimeReturns = 
    CALCULATE(
        COUNTROWS('EquipmentRegister'),
        'EquipmentRegister'[ActualReturnDate] <= 'EquipmentRegister'[ExpectedReturnDate]
    )
RETURN
    DIVIDE(OnTimeReturns, TotalReturns, 0) * 100

// Access Revocation Status
Access Fully Revoked % = 
VAR TerminatedEmployees = 
    CALCULATE(
        COUNTROWS('Employees'),
        'Employees'[EmployeeStatus] = \"Terminated\",
        'Employees'[EndDate] < TODAY()
    )
VAR FullyRevoked = 
    CALCULATE(
        DISTINCTCOUNT('SystemAccess'[Employee]),
        'SystemAccess'[AccessStatus] = \"Deprovisioned\",
        'Employees'[EmployeeStatus] = \"Terminated\"
    )
RETURN
    DIVIDE(FullyRevoked, TerminatedEmployees, 0) * 100
```

---

## 3. Compliance Dashboard

### Purpose
Monitor compliance and risk metrics

### Pages

#### Page 1: Compliance Overview

**KPI Cards**:
- Total Active Alerts
- Critical Alerts
- Compliance Score (0-100)
- Audit Readiness Score

**Visuals**:

1. **Alerts by Severity** (Donut chart)
   - Slices: Critical, High, Medium, Low
   - Colors: Red, Orange, Yellow, Green

2. **Alert Trend** (Line + Column chart)
   - X-axis: Week
   - Columns: New alerts
   - Line: Resolved alerts

3. **Top Compliance Issues** (Bar chart)
   - Y-axis: Alert type
   - X-axis: Count
   - Sorted: Descending

4. **Alert Resolution Time** (Box plot)
   - By severity
   - Shows median, quartiles

#### Page 2: Risk Analysis

**Visuals**:

1. **Risk Heatmap** (Matrix)
   - Rows: Department
   - Columns: Risk category
   - Values: Count of issues
   - Conditional formatting: Red/Yellow/Green

2. **Unreturned Equipment** (Table)
   - High-value items not returned
   - Days overdue
   - Employee details

3. **Access Not Revoked** (Table)
   - Terminated employees
   - Active system access
   - Days since termination

4. **Expired Documents** (Clustered bar)
   - X-axis: Document type
   - Y-axis: Count
   - Clusters: Expired, Expiring soon

#### Page 3: Audit Trail

**Visuals**:

1. **Activity Volume** (Line chart)
   - X-axis: Hour of day
   - Y-axis: Action count
   - Lines: By action type

2. **Failed Actions** (Table)
   - Recent failed operations
   - Error details
   - Retry status

3. **User Activity** (Tree map)
   - Size: Action count
   - Color: By department
   - Labels: User name

### DAX Measures

```dax
// Compliance Score
Compliance Score = 
VAR TotalChecks = 10 // Number of compliance checks
VAR PassedChecks = 
    CALCULATE(
        // Equipment returned: +1
        IF([Equipment Return Compliance %] > 95, 1, 0) +
        // Access revoked: +1
        IF([Access Fully Revoked %] > 98, 1, 0) +
        // No critical alerts: +2
        IF([Critical Alerts] = 0, 2, 0) +
        // Documents valid: +1
        IF([Expired Documents Count] = 0, 1, 0) +
        // SLA compliance: +2
        IF([SLA Compliance %] > 95, 2, 0) +
        // Onboarding complete: +1
        IF([Incomplete Onboardings] = 0, 1, 0) +
        // Payroll terminated: +1
        IF([Payroll Not Terminated] = 0, 1, 0) +
        // Audit trail complete: +1
        1
    )
RETURN
    (PassedChecks / TotalChecks) * 100

// Alert Resolution Time (Days)
Avg Alert Resolution Time = 
AVERAGEX(
    FILTER(
        'ComplianceAlerts',
        'ComplianceAlerts'[AlertStatus] = \"Resolved\" &&
        NOT(ISBLANK('ComplianceAlerts'[ResolvedDate]))
    ),
    DATEDIFF(
        'ComplianceAlerts'[GeneratedDate],
        'ComplianceAlerts'[ResolvedDate],
        DAY
    )
)

// High Risk Employees (Terminated but still has access)
High Risk Count = 
CALCULATE(
    DISTINCTCOUNT('Employees'[EmployeeNumber]),
    'Employees'[EmployeeStatus] = \"Terminated\",
    'Employees'[EndDate] < TODAY() - 1,
    CALCULATE(
        COUNTROWS('SystemAccess'),
        'SystemAccess'[AccessStatus] = \"Active\"
    ) > 0
)
```

---

## 4. Department Performance Dashboard

### Purpose
Compare department efficiency and performance

### Pages

#### Page 1: Department Comparison

**Visuals**:

1. **Task Completion Rate by Department** (Bar chart)
   - Y-axis: Department
   - X-axis: Completion rate %
   - Benchmark line at 95%

2. **Average Task Duration** (Column chart)
   - X-axis: Department
   - Y-axis: Hours
   - Color: By benchmark comparison

3. **SLA Performance** (Clustered column)
   - X-axis: Department
   - Y-axis: Count
   - Clusters: On Time, Late

4. **Workload Distribution** (Stacked area)
   - X-axis: Week
   - Y-axis: Task count
   - Areas: Each department

#### Page 2: Drill-through (Department Details)

When clicking a department:
- Department name header
- Team member performance table
- Task breakdown by category
- Trend over last 6 months

---

## 5. Audit Analytics Dashboard

### Purpose
Detailed audit trail analysis for compliance officers

### Pages

#### Page 1: Activity Analysis

**Visuals**:

1. **Activity Log** (Table with filters)
   - Searchable
   - Filters: Date range, User, Action type
   - Export enabled

2. **Automated vs Manual Actions** (Pie chart)
   - Slices: Automated, Manual
   - Shows automation rate

3. **Activity by Hour** (Heatmap)
   - X-axis: Hour of day
   - Y-axis: Day of week
   - Color: Activity volume

4. **Failed Operations** (Table)
   - Recent failures
   - Error messages
   - Frequency

---

## Data Model

### Fact Tables
- LifecycleRequests
- WorkflowTasks
- ComplianceAlerts
- AuditTrail
- SystemAccess
- EquipmentRegister

### Dimension Tables
- Employees
- Calendar (Date dimension)
- Departments (custom table)
- ApprovalMatrix

### Relationships
```
Employees (1) ----< (*) LifecycleRequests
LifecycleRequests (1) ----< (*) WorkflowTasks
Employees (1) ----< (*) EquipmentRegister
Employees (1) ----< (*) SystemAccess
Calendar (1) ----< (*) LifecycleRequests[RequestDate]
Calendar (1) ----< (*) WorkflowTasks[DueDate]
```

---

## Refresh Schedule

- **Executive Dashboard**: Every 1 hour (DirectQuery for real-time)\n- **HR Operations**: Every 30 minutes\n- **Compliance Dashboard**: Every 15 minutes (Critical)\n- **Department Performance**: Daily at 2 AM\n- **Audit Analytics**: Every 4 hours\n\n---\n\n## Row-Level Security (RLS)\n\n### Roles\n\n**HR Team**\n```dax\n// Can see all data\n1 = 1\n```\n\n**Manager**\n```dax\n// Can see only their team\n'Employees'[Manager] = USERPRINCIPALNAME()\n```\n\n**Department User**\n```dax\n// Can see only their department\n'Employees'[Department] = [UserDepartment]\n```\n\n**Executive**\n```dax\n// Can see all data but aggregated only\n1 = 1\n```\n\n---\n\n## Performance Optimization\n\n1. **Aggregations**: Pre-aggregate fact tables\n2. **Incremental Refresh**: Load only last 90 days\n3. **Query Reduction**: Disable auto date/time\n4. **Composite Model**: Mix DirectQuery + Import\n5. **Calculated Columns**: Minimize, use measures instead\n\n---\n\n## Deployment\n\n### Workspace Setup\n```\nEmployee360 Analytics Workspace\n├── Development (Dev)\n├── Testing (UAT)\n└── Production (Prod)\n```\n\n### Deployment Pipeline\n1. Develop in Dev workspace\n2. Test in UAT workspace\n3. Deploy to Prod via pipelines\n4. Schedule refreshes\n5. Share with users\n\n---\n\n## Mobile Layout\n\n- Phone layout created for all dashboards\n- Portrait orientation\n- Key metrics only\n- Optimized for touch\n\n---\n\n## Embedding\n\n- Executive Dashboard embedded in Power Apps\n- Compliance Dashboard embedded in Compliance Centre\n- Secure embed enabled\n- RLS enforced\n\n---\n\n**Total Dashboards**: 5\n**Total Pages**: 15+\n**Total Measures**: 50+\n**Total Visuals**: 75+\n**Refresh Frequency**: 15 min - 4 hours\n**Data Retention**: 7 years\n\n**Last Updated**: 2025\n**Version**: 1.0\n**Status**: Specifications Complete\n