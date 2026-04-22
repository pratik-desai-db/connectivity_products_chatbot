# Microsoft Copilot Studio Implementation Plan

## **PROJECT OVERVIEW**

**Timeline:** 6 weeks
**Team Required:** 1 Power Platform Developer + 1 Business Analyst + IT Support
**Budget:** Minimal (likely covered by existing licenses)

---

## **PHASE 1: PREPARATION & SETUP (Week 1)**

### **Day 1-2: License & Access Setup**

**Tasks:**
1. **Verify Licensing**
   - Check if you have Microsoft 365 E3/E5 or Copilot Studio licenses
   - If not, request trial license (30 days free)
   - Contact: Your Microsoft account manager

2. **Grant Permissions**
   - Copilot Studio access for developer
   - SharePoint site admin access
   - JIRA API credentials
   - Power Automate premium connector access

3. **Environment Setup**
   - Create dedicated Power Platform environment (Production + Test)
   - Enable Copilot Studio in your tenant
   - Configure data loss prevention policies

**Deliverable:** ✅ Access confirmed, environment ready

---

### **Day 3-5: Requirements Documentation**

**Tasks:**
1. **Map Current JIRA Workflow**
   ```
   Document:
   - Exact JIRA field names (API names)
   - Required vs optional fields
   - Field validations
   - Dropdown values for each field
   - Epic naming conventions
   ```

2. **Define Conversation Flows**
   Create a conversation script. Here's a starter template:

   ```
   BOT: Hi! I'm your Connectivity Assistant. I'll help you request 
        fiber or cable connections for your trading infrastructure.
        What type of connection do you need?
        
        [Fiber Connection] [Cable Connection] [Not Sure]

   CUSTOMER: [Selects Fiber]

   BOT: Great! Which business area is this for?
        [EFAG] [ECAG] [ExR] [CP] [MD+S] [Cash] [PEX] [Funding]

   CUSTOMER: [Selects EFAG]

   BOT: What's the priority level for this request?
        [Low - Standard delivery]
        [Medium - Expedited]
        [High - Urgent]
        [Emergency - Critical trading impact]

   ... and so on
   ```

3. **Identify Edge Cases**
   - What if customer wants multiple connections?
   - What if they need to modify during conversation?
   - What if JIRA is down?

**Deliverable:** ✅ Complete conversation flowchart + JIRA field mapping document

---

## **PHASE 2: BUILD COPILOT (Week 2-3)**

### **Week 2: Create Basic Bot Structure**

**Step 1: Create Your Copilot**
1. Go to https://copilotstudio.microsoft.com
2. Click "Create" → "New Copilot"
3. Name: "Connectivity Request Assistant"
4. Language: English
5. Enable "Generative AI" features

**Step 2: Create Topics (Conversation Flows)**

**Topic 1: Greeting & Connection Type**
```
Topic Name: Start - Connection Request
Trigger Phrases:
- "I need a connection"
- "Request fiber"
- "Cable setup"
- "New connectivity"
- "Help with connection"

Conversation Flow:
1. Message: "I'll help you request connectivity. What type?"
2. Question (Multiple Choice):
   - Save response as: connectionType
   - Options:
     * Fiber Connection
     * Cable Connection
     * I'm not sure
3. Condition:
   - If "I'm not sure" → Redirect to "Help Me Choose" topic
   - Else → Continue to next topic
```

**Topic 2: Business Area Selection**
```
Topic Name: Select Business Area
Trigger: Automatic from Topic 1

Conversation Flow:
1. Message: "Which business area is this for?"
2. Question (Multiple Choice):
   - Save response as: businessArea
   - Options (as buttons):
     * EFAG
     * ECAG
     * ExR
     * CP
     * MD+S
     * Cash
     * PEX
     * Funding
```

**Topic 3: Priority Selection**
```
Topic Name: Set Priority
Trigger: Automatic from Topic 2

Conversation Flow:
1. Message: "What's the priority for this request?"
2. Question (Multiple Choice):
   - Save response as: priority
   - Options:
     * Low (Standard delivery - 5-7 days)
     * Medium (Expedited - 2-3 days)
     * High (Urgent - 24 hours)
     * Emergency (Critical trading impact - Immediate)
```

**Topic 4: Due Date**
```
Topic Name: Set Due Date
Trigger: Automatic from Topic 3

Conversation Flow:
1. Message: "When do you need this delivered?"
2. Question (Date picker):
   - Save response as: dueDate
   - Validation: Must be future date
```

**Topic 5: Additional Details**
```
Topic Name: Collect Additional Info
Trigger: Automatic from Topic 4

Conversation Flow:
1. Message: "Please provide any additional details about your connectivity needs 
            (location, bandwidth requirements, etc.)"
2. Question (Text - Multi-line):
   - Save response as: additionalDetails
   - Optional field
```

**Topic 6: Confirmation & Summary**
```
Topic Name: Confirm and Submit
Trigger: Automatic from Topic 5

Conversation Flow:
1. Adaptive Card (Summary):
   -----------------------------------
   Connection Request Summary
   
   Connection Type: {connectionType}
   Business Area: {businessArea}
   Priority: {priority}
   Due Date: {dueDate}
   Details: {additionalDetails}
   -----------------------------------
   
   Is this correct?
   [Yes, Submit] [No, Start Over]

2. Condition:
   - If Yes → Call Power Automate flow
   - If No → Redirect to Topic 1
```

**Deliverable:** ✅ All conversation topics created and tested in Copilot Studio

---

### **Week 3: Advanced Features**

**Add Smart Features:**

1. **Enable Generative Answers**
   - Allow bot to answer general questions about your connectivity products
   - Upload FAQ document or link to SharePoint knowledge base
   - Settings → Generative AI → Enable "Search public data and websites"

2. **Add Fallback Topic**
   ```
   Topic: System Fallback
   Trigger: When bot doesn't understand
   
   Message: "I didn't quite catch that. I can help you with:
            • New fiber connection requests
            • New cable connection requests
            • Check status of existing requests
            
            What would you like to do?"
   ```

3. **Add Authentication (if needed)**
   - Settings → Security → Authentication
   - Enable "Require users to sign in"
   - Use Microsoft Entra ID (Azure AD)
   - Capture user email automatically

**Deliverable:** ✅ Enhanced bot with AI features and error handling

---

## **PHASE 3: JIRA INTEGRATION (Week 4)**

### **Create Power Automate Flow**

**Step 1: Create Flow from Copilot Studio**
1. In your Copilot, go to Topics
2. Open "Confirm and Submit" topic
3. Add node → "Call an action" → "Create a flow"

**Step 2: Build the Flow**

```
FLOW NAME: Create JIRA Ticket from Copilot

TRIGGER: Power Virtual Agents (Copilot Studio)
Input Parameters:
- connectionType (String)
- businessArea (String)
- priority (String)
- dueDate (Date)
- additionalDetails (String)
- userEmail (String)

ACTIONS:

1. Initialize Variables
   - jiraPriority (convert bot priority to JIRA format)
   - epicName (construct from inputs)

2. Compose - Build JIRA Payload
   {
     "fields": {
       "project": {
         "key": "YOUR_PROJECT_KEY"
       },
       "summary": "[connectionType] Connection Request - [businessArea]",
       "description": {
         "type": "doc",
         "version": 1,
         "content": [
           {
             "type": "paragraph",
             "content": [
               {
                 "type": "text",
                 "text": "Connection Type: [connectionType]\n
                          Business Area: [businessArea]\n
                          Priority: [priority]\n
                          Requested By: [userEmail]\n
                          Due Date: [dueDate]\n\n
                          Additional Details:\n[additionalDetails]"
               }
             ]
           }
         ]
       },
       "issuetype": {
         "name": "Epic"
       },
       "customfield_XXXX": "[businessArea]",  // Replace with actual field ID
       "priority": {
         "name": "[jiraPriority]"
       },
       "duedate": "[dueDate]"
     }
   }

3. HTTP Action - Create JIRA Ticket
   Method: POST
   URI: https://your-domain.atlassian.net/rest/api/3/issue
   Headers:
     - Content-Type: application/json
     - Authorization: Basic [YOUR_BASE64_ENCODED_CREDENTIALS]
   Body: [Output from Compose]

4. Parse JSON (JIRA Response)
   Content: [Body from HTTP]
   Schema: {JIRA response schema}

5. Condition - Check if Successful
   If HTTP Status = 201:
     - Return to Copilot:
       * ticketKey: [Parse JSON - key]
       * ticketUrl: [Parse JSON - self]
       * success: true
   Else:
     - Return to Copilot:
       * success: false
       * errorMessage: [Body]

OUTPUTS:
- ticketKey (String)
- ticketUrl (String)
- success (Boolean)
```

**Step 3: Get JIRA Credentials**

You'll need:
1. **JIRA API Token**
   - Login to JIRA
   - Account Settings → Security → API Tokens
   - Create token
   
2. **Encode Credentials**
   ```
   Format: email@company.com:api_token
   Encode to Base64
   Use in Authorization header: Basic {base64_encoded_string}
   ```

3. **Custom Field IDs**
   - Make GET request: `https://your-domain.atlassian.net/rest/api/3/field`
   - Find IDs for: Business Area, Status, Priority fields

**Deliverable:** ✅ Working Power Automate flow that creates JIRA tickets

---

### **Update Copilot with Flow Response**

Back in Copilot Studio:

**Topic: Confirm and Submit (Update)**
```
After "Yes, Submit" is clicked:

1. Call Power Automate Flow
   - Pass all variables

2. Condition on flow output:
   If success = true:
     Message: "✅ Your request has been submitted!
              
              Ticket Number: {ticketKey}
              
              You can track your request here: {ticketUrl}
              
              Our team will review and contact you within 24 hours.
              
              Is there anything else I can help you with?
              [New Request] [Check Status] [Done]"
   
   Else:
     Message: "❌ Sorry, there was an error creating your ticket.
              Error: {errorMessage}
              
              Please contact support@company.com or try again.
              [Try Again] [Contact Support]"

3. End conversation or redirect based on selection
```

**Deliverable:** ✅ End-to-end flow from chat to JIRA ticket

---

## **PHASE 4: SHAREPOINT INTEGRATION (Week 5)**

### **Embed Copilot in SharePoint**

**Method 1: As a Web Part (Recommended)**

1. **Publish Your Copilot**
   - In Copilot Studio → Channels
   - Click "Custom website"
   - Copy embed code

2. **Add to SharePoint**
   - Go to your SharePoint team site
   - Edit page
   - Add web part → "Embed"
   - Paste embed code
   - Adjust size (recommended: 400px width, 600px height)

3. **Customize Appearance**
   - In Copilot Studio → Settings → Appearance
   - Upload company logo
   - Match brand colors
   - Customize greeting message

**Method 2: As Floating Chat Widget**

```html
<!-- Add to SharePoint page HTML -->
<script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
<div id="webchat" role="main"></div>
<script>
  window.WebChat.renderWebChat(
    {
      directLine: window.WebChat.createDirectLine({
        token: 'YOUR_DIRECT_LINE_TOKEN'
      }),
      userID: 'USER_ID',
      username: 'USER_NAME',
      locale: 'en-US',
      styleOptions: {
        botAvatarImage: 'YOUR_LOGO_URL',
        botAvatarInitials: 'CA',
        accent: '#0078D4',
        backgroundColor: 'White'
      }
    },
    document.getElementById('webchat')
  );
</script>
```

**Deliverable:** ✅ Copilot visible and functional on SharePoint site

---

## **PHASE 5: TESTING & REFINEMENT (Week 6)**

### **Testing Checklist**

**Functional Testing:**
- [ ] All conversation paths work correctly
- [ ] All button options display properly
- [ ] Date picker accepts valid dates only
- [ ] JIRA tickets created with correct fields
- [ ] Ticket numbers returned to user
- [ ] Error handling works (JIRA down scenario)

**User Acceptance Testing:**
- [ ] Recruit 5 internal users
- [ ] Ask them to submit real requests
- [ ] Collect feedback on:
  * Clarity of questions
  * Button label names
  * Overall experience
  * Any confusion points

**Edge Case Testing:**
- [ ] Very long text in additional details
- [ ] Special characters in input
- [ ] Rapid-fire clicking
- [ ] Multiple browser types
- [ ] Mobile device testing

### **Refinement Tasks**

Based on testing:
1. **Adjust conversation flow** if users get confused
2. **Add help tooltips** to complex fields
3. **Improve error messages** to be more helpful
4. **Add proactive messages** ("This usually takes 2-3 minutes...")
5. **Create analytics dashboard** to track:
   - Number of requests
   - Most common connection type
   - Average completion time
   - Drop-off points

**Deliverable:** ✅ Production-ready, tested Copilot

---

## **PHASE 6: LAUNCH & TRAINING (End of Week 6)**

### **Launch Day Checklist**

**Pre-Launch:**
- [ ] Final testing in production environment
- [ ] Backup plan if major issues (email form as fallback)
- [ ] Monitor team ready
- [ ] JIRA team notified of new ticket source

**Communication Plan:**
1. **Email Announcement** (3 days before)
   ```
   Subject: New AI Assistant for Connectivity Requests

   We're excited to introduce our new Connectivity Request Assistant!
   
   Starting [DATE], you can request fiber and cable connections 
   directly through our SharePoint site using our AI chatbot.
   
   Benefits:
   • 24/7 availability
   • Faster request submission (2-3 minutes)
   • Instant ticket confirmation
   • No forms to fill out
   
   Link: [SharePoint Site URL]
   
   The old email process will remain available during transition.
   ```

2. **Quick Start Guide** (1-page PDF)
   - Screenshot walkthrough
   - 5-step process
   - FAQ section
   - Support contact

3. **Live Demo Session** (Optional)
   - 15-minute Teams meeting
   - Walk through example request
   - Q&A

**Launch Day:**
- Monitor conversations in real-time
- Be ready to fix issues quickly
- Collect user feedback
- Respond to questions

**Deliverable:** ✅ Successfully launched with user adoption plan

---

## **POST-LAUNCH: ONGOING OPTIMIZATION**

### **Week 1-2 After Launch**

**Daily Monitoring:**
- Check Copilot Studio Analytics
- Review created JIRA tickets for accuracy
- Monitor support emails for issues

**Metrics to Track:**
```
Success Metrics:
• Completion rate (started vs. completed conversations)
• Average conversation time
• JIRA ticket creation success rate
• User satisfaction (add rating at end of conversation)

Target Goals:
• >80% completion rate
• <3 minutes average time
• >95% JIRA creation success
• >4.0/5 user rating
```

### **Monthly Improvements**

1. **Analyze Conversation Logs**
   - Where do users drop off?
   - What questions cause confusion?
   - Common typos or misunderstandings?

2. **Update Topics**
   - Add frequently asked questions as new topics
   - Simplify complex paths
   - Add more help content

3. **Enhance AI Training**
   - Add trigger phrases users actually use
   - Improve generative answers with real examples
   - Update knowledge base

4. **Feature Additions** (Future roadmap)
   - Check request status by ticket number
   - Modify existing requests
   - Escalate urgent requests to Teams channel
   - Integration with scheduling for installation
   - Automated status updates to customers

---

## **RESOURCES & SUPPORT**

### **Documentation Links**
- [Copilot Studio Docs](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
- [Power Automate JIRA Connector](https://learn.microsoft.com/en-us/connectors/jira/)
- [JIRA REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)

### **Support Channels**
- Microsoft Support (if license issues)
- Internal IT team (SharePoint/Azure AD)
- JIRA admin (for custom fields)

### **Training for Your Team**
- [Microsoft Learn: Build Copilots](https://learn.microsoft.com/en-us/training/paths/work-power-virtual-agents/)
- [Power Automate Training](https://learn.microsoft.com/en-us/training/powerplatform/power-automate)

---

## **BUDGET BREAKDOWN**

| Item | Cost | Notes |
|------|------|-------|
| Copilot Studio License | $0-$200/month | Often included in E5 |
| Power Automate Premium | $15/user/month | For JIRA connector |
| Development Time | Internal resource | 6 weeks part-time |
| Testing | Internal resources | 1 week |
| **Total** | **~$200-500/month** | After initial setup |

---

## **RISK MITIGATION**

| Risk | Mitigation |
|------|------------|
| JIRA API changes | Use versioned API endpoints; monitor changelog |
| User adoption low | Phased rollout; incentivize early adopters |
| Bot misunderstands | Fallback to human (escalation path) |
| License costs increase | ROI tracking; demonstrate time savings |
| Data security concerns | Use within Microsoft tenant; no external data |

---

## **SUCCESS CRITERIA**

✅ **Launch Success:**
- Copilot live on SharePoint by Week 6
- At least 10 successful test requests
- No critical bugs

✅ **30-Day Success:**
- 50% of new requests through Copilot
- >80% completion rate
- Positive user feedback

✅ **90-Day Success:**
- 80%+ of requests through Copilot
- Measurable time savings (vs. email)
- Reduced back-and-forth clarifications
- Expansion to other request types

---

## **NEXT STEPS - WHAT YOU NEED TO DO NOW:**

### **This Week:**
1. ✅ Verify you have Copilot Studio access (check with IT)
2. ✅ Get JIRA API credentials (contact JIRA admin)
3. ✅ Document exact JIRA field names and values
4. ✅ Assign project resources (who will build this?)

### **Week 1 Starting:**
5. ✅ Schedule kickoff meeting with stakeholders
6. ✅ Create test environment
7. ✅ Start building first topic in Copilot Studio

---

**Would you like me to:**
1. Create the actual conversation script word-for-word?
2. Provide the exact Power Automate flow JSON?
3. Write the JIRA API integration code?
4. Design the user training materials?

**Let me know which part you want to dive deeper into first!** 🚀