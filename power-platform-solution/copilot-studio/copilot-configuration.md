# Copilot Studio Configuration

## Employee360 AI Assistant - Copilot Agent

### Overview
**Agent Name**: Employee360 Assistant
**Type**: Conversational AI Agent
**Integration**: Microsoft Teams + Embedded in Power Apps
**Language**: English (expandable to multiple languages)
**Knowledge Source**: SharePoint HR Knowledge Base

---

## Agent Configuration

### Basic Settings
- **Agent Name**: Employee360 Assistant
- **Icon**: Company logo or HR icon
- **Greeting Message**: "Hi! I'm your Employee360 assistant. I can help you with onboarding status, equipment queries, policy questions, and more. What can I help you with today?"
- **Fallback Message**: "I didn't quite understand that. Could you rephrase your question? You can ask about:
  - Onboarding status
  - Equipment
  - System access
  - HR policies
  - Task status"

---

## Topics

### System Topics (Pre-built)
1. **Greeting** - Welcome message
2. **Goodbye** - Farewell message
3. **Escalate** - Transfer to human agent
4. **Start Over** - Reset conversation
5. **Multiple Topics Matched** - Disambiguation

### Custom Topics

#### Topic 1: Check Onboarding Status
**Trigger Phrases**:
- "Where is my onboarding?"
- "What's the status of onboarding?"
- "Check onboarding progress"
- "When will I be onboarded?"
- "Is my onboarding complete?"

**Conversation Flow**:
1. Ask for employee email (if not authenticated)
2. Call Power Automate flow: "Get Onboarding Status"
3. Display response:
   - Request ID
   - Current stage
   - Percent complete
   - Pending tasks (if any)
   - Expected completion date
4. Offer actions:
   - "View full details"
   - "Contact HR"
   - "Ask another question"

**Power Automate Flow**: 
```json
{
  "input": "employeeEmail",
  "query": "Get lifecycle requests where Type='Onboarding' and Employee.Email='{email}' and Status!='Completed'",
  "output": {
    "requestId": "string",
    "currentStage": "string",
    "percentComplete": "number",
    "pendingTasks": "array"
  }
}
```

#### Topic 2: Check Task Status
**Trigger Phrases**:
- "What tasks are overdue?"
- "Show my pending tasks"
- "Which tasks do I need to complete?"
- "Task list"

**Conversation Flow**:
1. Authenticate user
2. Call flow: "Get User Tasks"
3. Display:
   - Overdue tasks count
   - Tasks due today
   - Tasks due this week
4. Show list (top 5)
5. Offer: "View all tasks in app"

#### Topic 3: Check Equipment Status
**Trigger Phrases**:
- "What equipment is assigned to me?"
- "When should I return my laptop?"
- "Equipment assigned to [employee name]"
- "Check equipment status"

**Conversation Flow**:
1. Ask: "Whose equipment would you like to check?" (default: logged-in user)
2. Call flow: "Get Equipment by Employee"
3. Display:
   - Device type
   - Asset tag
   - Assignment date
   - Return date (if applicable)
4. Offer actions:
   - "Report an issue"
   - "Request return"

#### Topic 4: Check Access Status
**Trigger Phrases**:
- "What systems do I have access to?"
- "When will my access be provisioned?"
- "Check system access"
- "Access status"

**Conversation Flow**:
1. Authenticate user
2. Call flow: "Get System Access"
3. Display:
   - Active access (count)
   - Pending provisioning (if any)
   - Recent access changes
4. Option: "Request new access"

#### Topic 5: Find Approver
**Trigger Phrases**:
- "Who needs to approve my request?"
- "Who has not approved?"
- "Check approval status"
- "Who is my approver?"

**Conversation Flow**:
1. Ask for request ID
2. Call flow: "Get Approval Status"
3. Display:
   - Current approval level
   - Current approver(s)
   - Approval status
   - Pending approvals
4. Option: "Send reminder"

#### Topic 6: Offboarding Employees
**Trigger Phrases**:
- "Which employees are offboarding?"
- "Who is leaving?"
- "Show terminations this month"
- "Offboarding list"

**Conversation Flow**:
1. Verify user has HR/Manager role
2. Ask time period (this week/month)
3. Call flow: "Get Offboarding List"
4. Display:
   - Employee names
   - Last working dates
   - Status
5. Option: "View details"

#### Topic 7: HR Policy Questions
**Trigger Phrases**:
- "What is the leave policy?"
- "Tell me about [policy name]"
- "HR policy for [topic]"
- "How do I [HR process]?"

**Conversation Flow**:
1. Extract policy topic from query
2. Search Knowledge Base
3. Display top 3 matching articles
4. Show article summary
5. Options:
   - "Read full article"
   - "Search again"
   - "Contact HR"

**Generative AI**: Enable generative answers from Knowledge Base

#### Topic 8: Report Issue
**Trigger Phrases**:
- "Report a problem"
- "I have an issue"
- "Something is not working"
- "Need help"

**Conversation Flow**:
1. Ask: "What type of issue?"
   - Equipment issue
   - System access issue
   - Onboarding issue
   - Other
2. Collect details:
   - Description
   - Priority
3. Create workflow task or ticket
4. Provide ticket number
5. "Someone will contact you within 4 hours"

#### Topic 9: Contact Information
**Trigger Phrases**:
- "Who is the HR manager?"
- "Contact IT support"
- "How do I reach [department]?"
- "Support contact"

**Conversation Flow**:
1. Identify department/role
2. Display:
   - Contact name
   - Email
   - Phone
   - Teams link
3. Option: "Message them now"

#### Topic 10: My Manager
**Trigger Phrases**:
- "Who is my manager?"
- "Manager information"
- "Report to whom?"

**Conversation Flow**:
1. Get user's manager from Employee list
2. Display:
   - Manager name
   - Email
   - Department
   - Teams presence
3. Options:
   - "Chat with manager"
   - "View org chart"

---

## Entities

Custom entities to extract from user input:

1. **EmployeeName**
   - Type: List
   - Values: Synced from Employees list

2. **DepartmentName**
   - Type: List
   - Values: HR, IT, Finance, Operations, etc.

3. **RequestID**
   - Type: Regex
   - Pattern: `LCR-\d{8}-\d{4}`

4. **EquipmentType**
   - Type: List
   - Values: Laptop, Desktop, Monitor, Phone, etc.

5. **PolicyCategory**
   - Type: List
   - Values: Leave, Benefits, Code of Conduct, etc.

---

## Actions (Power Automate Integrations)

### Action 1: Get Onboarding Status
**Input**: employeeEmail (string)
**Output**: 
```json
{
  "requestId": "LCR-20250115-0001",
  "currentStage": "IT Provisioning",
  "percentComplete": 65,
  "pendingTasks": [
    "Setup VPN Access",
    "Assign Desk"
  ],
  "expectedCompletion": "2025-01-20"
}
```

### Action 2: Get User Tasks
**Input**: userEmail (string)
**Output**:
```json
{
  "overdue": 2,
  "dueToday": 1,
  "dueThisWeek": 4,
  "tasks": [
    {
      "taskName": "Setup Email Account",
      "dueDate": "2025-01-15",
      "priority": "High"
    }
  ]
}
```

### Action 3: Get Equipment by Employee
**Input**: employeeEmail (string)
**Output**:
```json
{
  "equipment": [
    {
      "deviceType": "Laptop",
      "assetTag": "LAP-12345",
      "assignmentDate": "2024-06-01",
      "status": "Assigned"
    }
  ]
}
```

### Action 4: Get Approval Status
**Input**: requestId (string)
**Output**:
```json
{
  "currentLevel": 2,
  "currentApprover": "Jane Smith",
  "approverEmail": "jane.smith@company.com",
  "status": "Pending",
  "waitingDays": 3
}
```

### Action 5: Search Knowledge Base
**Input**: searchQuery (string)
**Output**:
```json
{
  "articles": [
    {
      "title": "Annual Leave Policy",
      "summary": "Employees are entitled to 20 days...",
      "url": "https://..."
    }
  ]
}
```

### Action 6: Create Support Ticket
**Input**: 
```json
{
  "issueType": "Equipment",
  "description": "Laptop screen is broken",
  "priority": "High",
  "userEmail": "user@company.com"
}
```
**Output**: ticketId (string)

---

## Generative AI Configuration

### Knowledge Sources
1. **SharePoint - HR Knowledge Base**
   - Site: Employee360 site
   - List: HRKnowledgeBase
   - Filter: IsCopilotEnabled = true AND ArticleStatus = Published
   - Refresh: Daily

2. **Public Web Content** (optional)
   - Company handbook URL
   - HR policies site

### Generative Answers Settings
- **Enable**: Yes
- **Content Moderation**: High
- **Boost Conversational Style**: Enabled
- **Cite Sources**: Always
- **Response Length**: Medium (2-3 paragraphs)

### AI Capabilities
- **Sentiment Analysis**: Track user satisfaction
- **Intent Recognition**: Improved topic matching
- **Entity Extraction**: Auto-extract names, dates, IDs
- **Language Understanding**: Handle variations

---

## Authentication

### Authentication Method
**Microsoft Entra ID (Azure AD) SSO**

- **Scope**: User.Read, Sites.Read.All
- **Login Prompt**: "Sign in with your work account"
- **Session**: Persist for 30 days
- **Token Refresh**: Automatic

### Variable Mapping
- `User.Email` → Authenticated email
- `User.DisplayName` → User's name
- `User.Id` → Entra ID object ID

---

## Channels

### 1. Microsoft Teams
**Deployment**: Add to Teams app catalog
**Scope**: 
- Personal app (1:1 chat)
- Team channel (general queries)

**Features**:
- Proactive messaging (task reminders)
- Adaptive cards for rich responses
- Quick replies

### 2. Power Apps Embedded
**Location**: Employee Self-Service Portal
**Position**: Floating button (bottom-right)
**Trigger**: Click on help icon

### 3. Company Website (Future)
**Widget**: Embedded chat widget
**Pages**: HR portal, Careers page

---

## Analytics & Monitoring

### Key Metrics
1. **Usage**:
   - Total conversations
   - Active users
   - Conversations per user
   - Peak usage times

2. **Performance**:
   - Resolution rate
   - Average conversation length
   - Escalation rate
   - Response time

3. **Topics**:
   - Most triggered topics
   - Abandoned topics
   - Confusion rate per topic

4. **User Satisfaction**:
   - CSAT score
   - Thumbs up/down ratio
   - Sentiment analysis

### Dashboards
- **Real-time Monitor**: Active conversations
- **Weekly Report**: Usage trends
- **Monthly Review**: Topic performance

---

## Conversation Design Best Practices

1. **Be Conversational**: Use natural language
2. **Be Concise**: Keep responses short
3. **Provide Options**: Always offer next steps
4. **Show Empathy**: Acknowledge user's concerns
5. **Clarify**: Ask follow-up questions when unclear
6. **Cite Sources**: Link to knowledge articles
7. **Escalate Gracefully**: Easy path to human agent

---

## Escalation to Human Agent

**Trigger Conditions**:
- User explicitly requests human
- Confidence score < 0.5 for 3 consecutive messages
- Negative sentiment detected
- Complex query requiring manual review

**Escalation Process**:
1. "I'll connect you with a team member"
2. Create support ticket
3. Notify HR support queue (Teams)
4. Transfer conversation context
5. "Someone will reach out within 30 minutes"

---

## Sample Conversations

### Example 1: Check Onboarding
```
User: Where is my onboarding?

Bot: Let me check that for you!

[Calls flow]

Bot: Here's your onboarding status:
📋 Request ID: LCR-20250115-0042
✅ Progress: 75% complete
📍 Current Stage: IT Provisioning

Pending Tasks:
• Setup VPN Access (Due: Jan 18)
• Assign Desk (Due: Jan 19)

Expected completion: January 20

Would you like to:
1️⃣ View full details
2️⃣ Contact your HR coordinator
3️⃣ Ask another question

User: View full details

Bot: [Opens link to request in Power App]
I've opened the full details. Is there anything else I can help with?
```

### Example 2: Equipment Query
```
User: What equipment do I have?

Bot: Let me look that up!

[Retrieves data]

Bot: You currently have 3 items assigned:

💻 Laptop
• Asset Tag: LAP-54321
• Model: Dell Latitude 5520
• Assigned: June 1, 2024

🖥️ Monitor
• Asset Tag: MON-78901
• Model: Dell 27" 4K
• Assigned: June 1, 2024

🔑 Building Access Card
• Asset Tag: ACC-11223
• Assigned: June 1, 2024

Need to:
🔧 Report an issue?
📦 Request equipment return?
❓ Ask something else?

User: Report an issue

Bot: Which equipment has an issue?
[Shows buttons: Laptop | Monitor | Access Card]

[User continues...]
```

---

## Testing Checklist

- [ ] All topics trigger correctly
- [ ] Power Automate flows return data
- [ ] Authentication works
- [ ] Generative answers cite sources
- [ ] Escalation path functions
- [ ] Teams integration works
- [ ] Power Apps embedding successful
- [ ] Analytics tracking active
- [ ] User feedback captured
- [ ] Error handling graceful

---

## Maintenance

**Weekly**:
- Review unrecognized intents
- Add new trigger phrases

**Monthly**:
- Analyze topic performance
- Update low-performing topics
- Refresh knowledge base

**Quarterly**:
- Review escalation rate
- Update conversation flows
- Add new topics based on trends

---

**Total Topics**: 15 (5 system + 10 custom)
**Total Actions**: 6 Power Automate flows
**Knowledge Sources**: 1 (SharePoint)
**Channels**: 3 (Teams, Power Apps, Web)
**Expected Usage**: 500+ conversations/day

**Last Updated**: 2025
**Version**: 1.0
**Status**: Configuration Ready
