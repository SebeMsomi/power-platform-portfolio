# SharePoint Lists Schema Documentation

## Employee360 AI Lifecycle Management Platform - Data Architecture

### Overview
This directory contains complete SharePoint list schemas for the Employee360 platform. Each JSON file represents a SharePoint list with full column definitions, datatypes, validations, relationships, views, and permissions.

### Lists Summary

| List Name | Purpose | Key Features | Record Count Estimate |
|-----------|---------|--------------|---------------------|
| **Employees** | Central employee master data | 23 columns, lookup relationships, status tracking | 10,000+ |
| **LifecycleRequests** | All lifecycle change requests | 24 columns, SLA tracking, workflow integration | 5,000/year |
| **WorkflowTasks** | Department tasks for requests | 23 columns, dependency tracking, escalation | 50,000/year |
| **ApprovalMatrix** | Approval routing configuration | 16 columns, multi-level approvals, SLA config | 50-100 |
| **EquipmentRegister** | Equipment tracking & returns | 24 columns, assignment tracking, maintenance | 15,000+ |
| **SystemAccess** | Access provisioning/deprovisioning | 24 columns, elevated access tracking, compliance | 100,000+ |
| **EmployeeDocuments** | Document library for employee docs | 19 columns, version control, IRM enabled | 100,000+ |
| **AuditTrail** | Comprehensive audit logging | 23 columns, 7-year retention, security events | 1,000,000+/year |
| **ComplianceAlerts** | System-generated compliance alerts | 26 columns, risk scoring, escalation tracking | 10,000/year |
| **HRKnowledgeBase** | FAQ and knowledge articles | 24 columns, Copilot integration, analytics | 500-1,000 |

### List Relationships

```
Employees (1) ─────> (M) LifecycleRequests
    │
    ├──> (M) EquipmentRegister
    ├──> (M) SystemAccess
    ├──> (M) EmployeeDocuments
    └──> (M) ComplianceAlerts

LifecycleRequests (1) ─────> (M) WorkflowTasks
    │
    └──> (M) AuditTrail

ApprovalMatrix ─────> Configuration table (referenced by Power Automate)

HRKnowledgeBase ─────> Copilot Studio integration
```

### Column Types Used

- **Text**: Standard text fields with validation
- **Note**: Multi-line text (plain or rich)
- **Choice**: Dropdown selections
- **Number**: Numeric values with min/max
- **Currency**: Monetary values
- **DateTime**: Date and time fields
- **Boolean**: Yes/No fields
- **User**: Person or Group picker
- **Lookup**: Relationships to other lists
- **Calculated**: Formula-based fields
- **URL**: Hyperlinks and images

### Key Design Principles

1. **Normalization**: Proper relational structure with lookup columns
2. **Audit Trail**: Comprehensive tracking of all changes
3. **Compliance**: Built-in retention policies and IRM
4. **Performance**: Strategic indexing on high-query columns
5. **Validation**: Column-level and formula-based validation
6. **Security**: Row-level security and role-based permissions
7. **Usability**: Multiple views for different user roles
8. **Scalability**: Designed for 10,000+ employees

### Security Model

Each list includes:
- **Break Permission Inheritance**: Custom permissions
- **Role-Based Access**: 8 security roles
- **Row-Level Security**: Users see only relevant records
- **Audit Logging**: All access tracked in AuditTrail list

### Views Configuration

Each list includes optimized views:
- **Filtered Views**: Pre-filtered for common scenarios
- **Grouped Views**: Organized by category/department
- **Personal Views**: "My Items" views
- **Role-Based Views**: Department-specific views

### Integration Points

1. **Power Automate**: All lists are flow-enabled
2. **Power Apps**: Model-driven and canvas apps
3. **Power BI**: Direct Query and Import modes
4. **Copilot Studio**: Knowledge base integration
5. **Microsoft Teams**: Embedded lists and notifications

### Deployment Instructions

1. **Create SharePoint Site**
   ```
   Site Type: Communication Site or Team Site
   Site Name: Employee360 Lifecycle
   URL: /sites/employee360
   ```

2. **Create Lists**
   - Import JSON schemas in order (01-10)
   - Use PowerShell or PnP Provisioning for bulk creation
   - Verify lookup column dependencies

3. **Configure Permissions**
   - Create 8 security groups in Entra ID
   - Apply permissions as per schema
   - Test row-level security

4. **Create Views**
   - All views defined in JSON
   - Customize colors/formatting as needed
   - Enable modern view experience

5. **Enable Features**
   - Versioning on critical lists
   - Content types for document library
   - IRM on EmployeeDocuments
   - Retention policies on AuditTrail

### Sample PowerShell Deployment Script

```powershell
# Connect to SharePoint Online
Connect-PnPOnline -Url "https://yourtenant.sharepoint.com/sites/employee360" -Interactive

# Create lists from JSON schemas
$schemas = Get-ChildItem -Path "./sharepoint-schemas" -Filter "*.json"

foreach ($schema in $schemas) {
    $json = Get-Content $schema.FullName | ConvertFrom-Json
    
    # Create list
    New-PnPList -Title $json.listName -Template $json.listTemplate -Url $json.listName
    
    # Add columns (simplified - full script would iterate through all columns)
    foreach ($column in $json.columns) {
        Add-PnPField -List $json.listName -DisplayName $column.displayName -InternalName $column.internalName -Type $column.type -Required:$column.required
    }
}
```

### Data Migration Considerations

1. **Employee Data**: Import from existing HRIS system
2. **Historical Data**: Retain for compliance (7 years)
3. **Document Migration**: Use SharePoint Migration Tool
4. **Validation**: Run data quality checks post-migration

### Performance Optimization

1. **Indexed Columns**: Already specified in schemas
2. **List Throttling**: Keep views under 5,000 items
3. **Archive Strategy**: Move old records to separate lists
4. **Caching**: Enable Power BI caching for reports

### Maintenance Tasks

- **Weekly**: Review compliance alerts
- **Monthly**: Audit data quality
- **Quarterly**: Review and update knowledge base
- **Annually**: Archive completed lifecycle requests

### Support & Documentation

For implementation support:
1. Refer to individual JSON schemas for detailed column specs
2. Review Power Automate flows in `/power-automate-flows` directory
3. Check Power Apps specifications in `/power-apps-specs` directory

---

**Total Lists**: 10
**Total Columns**: ~230+
**Total Views**: 40+
**Relationships**: 15+ lookup columns
**Estimated Storage**: 100GB+ (with documents)

**Last Updated**: 2025
**Version**: 1.0
**Status**: Production-Ready
