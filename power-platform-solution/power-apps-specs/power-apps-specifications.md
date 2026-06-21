# Power Apps Specifications

## Employee360 AI Lifecycle Management Platform - Application Design

### Application Architecture

**Primary App**: Model-Driven App (Employee360 Central)
**Supplementary Apps**: 3 Canvas Apps for specialized workflows

---

## 1. Model-Driven App: Employee360 Central

### Overview
- **App Type**: Model-Driven
- **Data Source**: SharePoint Lists
- **Target Users**: HR Team, Managers, IT, Facilities, Payroll
- **Access**: Web + Mobile (iOS/Android)

### Site Map Structure

```
Employee360 Central
│
├── 🏠 Dashboard
│   ├── Executive Dashboard (iframe - Power BI)
│   ├── My Tasks
│   └── My Requests
│
├── 👥 Employees
│   ├── All Active Employees
│   ├── Pre-Onboarding
│   ├── On Notice Period
│   └── Employee Details Form
│
├── 🔄 Lifecycle Requests
│   ├── Active Requests
│   ├── Pending Approval
│   ├── Overdue Requests
│   ├── My Requests
│   ├── Completed Requests
│   └── Request Details Form
│
├── ✅ Workflow Tasks
│   ├── My Tasks
│   ├── Overdue Tasks
│   ├── By Department
│   │   ├── HR Tasks
│   │   ├── IT Tasks
│   │   ├── Payroll Tasks
│   │   └── Facilities Tasks
│   └── Task Details Form
│
├── ✅ Approvals
│   ├── Pending My Approval
│   ├── Approval History
│   └── Approval Matrix Configuration
│
├── 📦 Equipment
│   ├── Available Equipment
│   ├── Assigned Equipment
│   ├── Pending Return
│   ├── High Value Assets
│   └── Equipment Details Form
│
├── 🔑 System Access
│   ├── Active Access
│   ├── Pending Provisioning
│   ├── Pending Deprovisioning
│   ├── Elevated Access
│   └── Access Details Form
│
├── 📄 Documents
│   ├── All Documents
│   ├── Pending Signature
│   ├── Expiring Soon
│   ├── Compliance Documents
│   └── Document Upload
│
├── ⚠️ Compliance
│   ├── Active Alerts
│   ├── Critical Alerts
│   ├── My Alerts
│   ├── By Department
│   └── Compliance Dashboard (iframe - Power BI)
│
├── 📖 Audit Trail
│   ├── Recent Activity
│   ├── Failed Actions
│   ├── High Severity Events
│   └── Compliance Review
│
├── ❓ Knowledge Base
│   ├── Browse Articles
│   ├── By Category
│   ├── Most Viewed
│   └── Article Details
│
└── ⚙️ Administration
    ├── User Management
    ├── Security Roles
    ├── Approval Matrix Config
    └── System Settings
```

### Key Forms

#### 1. Employee Details Form
**Layout**: 3-column with tabs

**Main Tab**:
- Header: Employee photo, name, status badge
- Column 1: Basic Info (Name, Email, Phone)
- Column 2: Employment (Department, Title, Manager)
- Column 3: Location (Office, Work Type)

**Employment Tab**:
- Start/End dates
- Employee type, cost center
- Security clearance

**Access & Equipment Tab**:
- Sub-grid: Active system access
- Sub-grid: Assigned equipment

**Documents Tab**:
- Sub-grid: Employee documents

**Lifecycle History Tab**:
- Sub-grid: Lifecycle requests
- Timeline: All activities

#### 2. Lifecycle Request Form
**Layout**: 2-column with tabs

**Main Tab**:
- Request ID (read-only), Status badge
- Lifecycle type, Employee
- Request/Effective dates
- Priority, Current stage
- Progress indicator (% complete)

**Details Tab**:
- From/To details (department, manager, location)
- Reason for change
- Approval comments

**Tasks Tab**:
- Sub-grid: Related workflow tasks
- Grouped by department
- Status indicators

**Approvals Tab**:
- Approval history
- Current approver
- Timeline view

**Audit Tab**:
- Sub-grid: Audit trail
- Filtered to this request

#### 3. Workflow Task Form
**Layout**: 2-column

**Main Section**:
- Task ID, Name, Description
- Department, Assigned to
- Status, Priority
- Due date (with overdue indicator)
- Percent complete slider

**Additional Info**:
- Related request (link)
- Blocking task (if any)
- Estimated vs actual effort
- Completion notes

**Business Process Flow**:
Task Lifecycle: Not Started → In Progress → Completed

#### 4. Equipment Details Form
**Layout**: 2-column with tabs

**Main Tab**:
- Asset tag, Device type
- Manufacturer, Model, Serial number
- Status, Value
- Purchase/Warranty dates

**Assignment Tab**:
- Assigned to, Assignment date
- Expected/Actual return date
- Return condition
- Days assigned (calculated)

**Maintenance Tab**:
- Maintenance schedule
- Last/Next maintenance dates

#### 5. Compliance Alert Form
**Layout**: 2-column

**Main Section**:
- Alert ID, Type, Severity badge
- Status, Category
- Description, Recommended action
- Related employee, request, task

**Resolution Section**:
- Assigned to, Due date
- Acknowledged by/date
- Resolved by/date
- Resolution notes
- Risk score gauge

### Business Process Flows

#### 1. Onboarding Process Flow
**Stages**:
1. Initial Request → HR creates employee record
2. Approval → Manager approves
3. Pre-Onboarding → Tasks assigned (5 days before start)
4. Day 1 Setup → Access provisioned
5. Week 1 Orientation → Training completed
6. Completed → All tasks done

#### 2. Offboarding Process Flow
**Stages**:
1. Resignation/Termination → Request submitted
2. Approval → Approved by appropriate authority
3. Knowledge Transfer → Handover completed
4. Access Revocation → IT removes all access
5. Equipment Return → All items returned
6. Final Settlement → Payroll processed
7. Completed → Employee status updated

### Dashboards (Model-Driven)

#### 1. My Dashboard (Employee Self-Service)
- My active tasks (list)
- My pending approvals (list)
- My lifecycle requests (chart)
- Quick actions (buttons)

#### 2. HR Operations Dashboard
- Active onboardings (chart)
- Active offboardings (chart)
- Overdue tasks by department (chart)
- Recent alerts (list)

#### 3. Executive Dashboard
- Embedded Power BI report
- Key metrics cards
- Trend charts

### Views Configuration

Each entity includes:
- **Personal views**: My items, My team
- **Public views**: All active, By department, By status
- **System views**: Advanced find, Quick find, Lookup

**View Features**:
- Column sorting
- Filtering
- Grouping
- Quick edit (editable grid)
- Export to Excel

### Security Roles

| Role | Access Level | Entities |
|------|--------------|----------|
| **HR Administrator** | Full CRUD | All entities |
| **HR Coordinator** | CRUD | Employees, Requests, Tasks, Documents |
| **Manager** | Read all, Create requests | Employees (team), Requests, Tasks (team) |
| **IT Administrator** | CRUD | System Access, Equipment, IT Tasks |
| **Payroll Team** | Read, Edit | Employees (payroll data), Payroll Tasks |
| **Facilities Team** | CRUD | Equipment, Facilities Tasks |
| **Compliance Officer** | Read all | All entities + Audit Trail |
| **Employee** | Read own | Own employee record, Own requests |

### Responsive Design

**Desktop (1920x1080)**:
- 3-column layouts
- Full navigation
- Inline editing
- Sub-grids expanded

**Tablet (768x1024)**:
- 2-column layouts
- Collapsible navigation
- Modal editing

**Mobile (375x667)**:
- Single column
- Bottom navigation
- Full-screen forms
- Swipe actions

---

## 2. Canvas App: Employee Self-Service Portal

### Purpose
Simplified interface for employees to:
- View their information
- Request transfers
- Report equipment issues
- Access knowledge base
- View their lifecycle status

### Screens

#### Screen 1: Home Dashboard
**Layout**: Card-based grid

**Components**:
- Welcome banner with employee photo
- Quick stats (Days employed, Pending tasks, etc.)
- Action cards:
  - View My Profile
  - Request Transfer
  - Report Issue
  - Browse Knowledge Base
  - My Equipment
- Recent notifications list

#### Screen 2: My Profile
**Layout**: Single column, sections

**Sections**:
- Personal information (read-only)
- Contact details (editable)
- Emergency contact (editable)
- My equipment list
- My system access list

#### Screen 3: Request Transfer
**Layout**: Wizard (multi-step form)

**Steps**:
1. Transfer Type (Department/Location/Role)
2. Details (To department, Reason)
3. Effective Date
4. Review & Submit

#### Screen 4: Knowledge Base
**Layout**: Search + Category browse

**Features**:
- Search bar with autocomplete
- Category tiles
- Article list with ratings
- Article detail view
- "Was this helpful?" feedback

#### Screen 5: My Equipment
**Layout**: Card list

**Features**:
- Equipment cards (photo, name, status)
- Report issue button
- Return request button
- View history

### Navigation
**Bottom Nav Bar** (Mobile):
- Home
- Profile
- Requests
- Help
- More

**Side Nav Panel** (Tablet/Desktop)

### Design System

**Colors**:
- Primary: #0078D4 (Microsoft Blue)
- Success: #107C10
- Warning: #FFB900
- Error: #D13438
- Neutral: #F3F2F1

**Fonts**:
- Headers: Segoe UI Bold
- Body: Segoe UI Regular
- Size scale: 12/14/16/20/24/32

**Components**:
- Custom header component
- Status badge component
- Action card component
- List item component
- Empty state component

---

## 3. Canvas App: Manager Dashboard

### Purpose
Dedicated app for managers to:
- View team members
- Approve requests
- Monitor team tasks
- Track onboarding progress

### Screens

#### Screen 1: Team Overview
- Team member cards
- Active onboardings
- Pending approvals (badge count)
- Team metrics (headcount, utilization)

#### Screen 2: Team Member Details
- Employee information
- Lifecycle history
- Current tasks
- Performance notes

#### Screen 3: Pending Approvals
- Approval request cards
- Approve/Reject actions
- Comments
- History

#### Screen 4: Team Tasks
- Gantt chart view
- Task list grouped by employee
- Overdue indicators

---

## 4. Canvas App: Onboarding Checklist

### Purpose
Interactive checklist for new hires' first week

### Screens

#### Screen 1: Welcome
- Welcome message
- Start date countdown (if pre-onboarding)
- Manager information
- First day details

#### Screen 2: Day 1 Checklist
- Building access setup ✓
- Laptop received ✓
- Email account active ✓
- Teams access verified ✓
- Meet the team ✓

#### Screen 3: Week 1 Checklist
- Orientation completed ✓
- HR policies acknowledged ✓
- Training scheduled ✓
- Systems training ✓
- Probation explained ✓

#### Screen 4: Resources
- Important contacts
- Office map
- FAQ links
- IT support

### Features
- Check items as complete
- Progress indicator
- Push notifications for pending items
- Manager can view progress

---

## Component Library

### Reusable Components

1. **StatusBadge**
   - Input: Status text
   - Output: Colored badge
   - Colors based on status

2. **ProgressBar**
   - Input: Current, Total
   - Output: Visual progress bar
   - Percentage display

3. **ActionButton**
   - Standardized button styles
   - Icon + Text
   - Loading state

4. **EmptyState**
   - Icon
   - Message
   - Action button

5. **ConfirmDialog**
   - Modal overlay
   - Confirm/Cancel
   - Custom message

6. **LoadingSpinner**
   - Centered spinner
   - Overlay dimming

7. **NotificationBanner**
   - Success/Warning/Error states
   - Dismissible
   - Auto-hide timer

---

## Accessibility

- **WCAG 2.1 AA Compliant**
- Screen reader support
- Keyboard navigation
- Color contrast ratios > 4.5:1
- Focus indicators
- Alt text for all images
- ARIA labels

---

## Performance Optimization

1. **Data Delegation**: All queries delegable
2. **Lazy Loading**: Load data on-demand
3. **Caching**: Cache lookup data
4. **Collections**: Use for frequently accessed data
5. **Image Optimization**: Compressed images
6. **Concurrent Calls**: Parallel data loading

---

## Testing Checklist

- [ ] All forms load correctly
- [ ] CRUD operations work
- [ ] Business process flows advance
- [ ] Security roles enforced
- [ ] Mobile responsive
- [ ] Offline capabilities (if enabled)
- [ ] Performance acceptable (<3s load)
- [ ] No console errors
- [ ] Accessibility tests pass

---

**Total Apps**: 4 (1 Model-Driven + 3 Canvas)
**Total Screens**: 25+
**Total Forms**: 10
**Total Views**: 50+
**Target Users**: All employees (10,000+)

**Last Updated**: 2025
**Version**: 1.0
**Status**: Design Complete - Ready for Development
