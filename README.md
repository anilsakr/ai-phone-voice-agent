# ai-phone-voice-agent
# n8n AI Voice Agent - Import & Setup Guide

## Quick Start: Importing the Workflow

### Step 1: Import the Workflow File

1. **Open n8n**
   - Cloud: Go to https://cloud.n8n.io
   - Self-hosted: Open http://localhost:5678

2. **Import the Workflow**
   - Click on **"Workflows"** in the left sidebar
   - Click the **"+"** button or **"Add Workflow"**
   - Click the **three dots menu** (⋮) in the top right
   - Select **"Import from File"**
   - Choose the `ai-voice-agent-workflow.json` file
   - Click **"Open"** or **"Import"**

3. **Save the Workflow**
   - The workflow will open in the editor
   - Click **"Save"** in the top right corner
   - Give it a name: "AI Voice Agent - Production"

---

## Step 2: Configure Required Credentials

### 2.1 OpenAI API Credentials

1. **Get OpenAI API Key**
   - Go to https://platform.openai.com/api-keys
   - Click "Create new secret key"
   - Copy the API key (starts with `sk-`)

2. **Add to n8n**
   - In n8n, click **"Credentials"** in the left sidebar
   - Click **"Add Credential"**
   - Search for **"OpenAI"**
   - Paste your API key
   - Name it: "OpenAI API"
   - Click **"Save"**

3. **Link to Workflow**
   - Open your workflow
   - Click on the **"OpenAI - Process Intent"** node
   - In the "Credential to connect with" dropdown, select "OpenAI API"
   - Click outside the node to save

### 2.2 Google Sheets Credentials (for storing appointments)

1. **Enable Google Sheets API**
   - Go to https://console.cloud.google.com
   - Create a new project or select existing
   - Enable "Google Sheets API"
   - Create OAuth 2.0 credentials

2. **Add to n8n**
   - In n8n, click **"Credentials"**
   - Click **"Add Credential"**
   - Search for **"Google Sheets OAuth2 API"**
   - Follow the OAuth flow
   - Name it: "Google Sheets API"
   - Click **"Save"**

3. **Create Your Spreadsheet**
   - Go to https://sheets.google.com
   - Create a new spreadsheet
   - Name it: "AI Voice Agent - Appointments"
   - Add column headers in first row:
     ```
     Appointment ID | Name | Email | Phone | Date | Time | Status | Call SID | Created At | Source
     ```
   - Copy the Spreadsheet ID from the URL:
     - URL: `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID_HERE/edit`
     - Copy the `SPREADSHEET_ID_HERE` part

4. **Link to Workflow**
   - In the workflow, click on **"Save to Google Sheets"** node
   - Select "Google Sheets API" credential
   - In "Document ID", paste your Spreadsheet ID
   - In "Sheet Name", enter "Sheet1" (or your sheet name)
   - Click outside to save

### 2.3 Gmail Credentials (for confirmation emails)

1. **Add to n8n**
   - In n8n, click **"Credentials"**
   - Click **"Add Credential"**
   - Search for **"Gmail OAuth2"**
   - Follow the OAuth flow
   - Name it: "Gmail OAuth2"
   - Click **"Save"**

2. **Link to Workflow**
   - Click on **"Send Confirmation Email"** node
   - Select "Gmail OAuth2" credential
   - Update the "From Email" to your email address
   - Click outside to save

### 2.4 Twilio Credentials

1. **Get Twilio Credentials**
   - Go to https://www.twilio.com/console
   - Find your **Account SID**
   - Find your **Auth Token**
   - Note your **Phone Number**

2. **Add to n8n**
   - In n8n, click **"Credentials"**
   - Click **"Add Credential"**
   - Search for **"Twilio API"**
   - Enter Account SID and Auth Token
   - Name it: "Twilio API"
   - Click **"Save"**

---

## Step 3: Configure Webhook URLs

### 3.1 Get Your n8n Webhook URLs

1. **Activate Workflow First (Important!)**
   - In your workflow editor, toggle the switch to **"Active"** (top right)
   - This generates the webhook URLs

2. **Get Webhook URLs**
   - Click on the **"Webhook - Inbound Call"** node
   - You'll see two URLs:
     - **Production URL**: `https://your-instance.app.n8n.cloud/webhook/inbound-call`
     - **Test URL**: `https://your-instance.app.n8n.cloud/webhook-test/inbound-call`
   - Copy the **Production URL**

3. **For Local Development (using ngrok)**
   ```bash
   # Install ngrok
   npm install -g ngrok
   
   # Start ngrok tunnel
   ngrok http 5678
   
   # Use the ngrok URL instead:
   # https://abc123.ngrok.io/webhook/inbound-call
   ```

### 3.2 Configure Twilio Webhook

1. **Go to Twilio Console**
   - Navigate to Phone Numbers → Manage → Active Numbers
   - Click on your phone number

2. **Configure Voice Settings**
   - Scroll to "Voice Configuration"
   - Under "A CALL COMES IN":
     - Select **"Webhook"**
     - Paste your n8n webhook URL: `https://your-instance.app.n8n.cloud/webhook/inbound-call`
     - Select **"HTTP POST"**
   - Click **"Save"**

3. **Test the Connection**
   - Call your Twilio phone number
   - You should hear the greeting from the workflow
   - Check the n8n executions panel to see if it triggered

---

## Step 4: Test the Workflow

### 4.1 Manual Testing in n8n

1. **Use Test Workflow**
   - In the workflow editor, click **"Test Workflow"** button
   - Click on the **"Webhook - Inbound Call"** node
   - Click **"Listen for Test Event"**
   - This generates a test URL

2. **Simulate a Twilio Request**
   - Use a tool like Postman or curl:
   ```bash
   curl -X POST https://your-instance.app.n8n.cloud/webhook-test/inbound-call \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "CallSid=TEST123" \
     -d "From=+1234567890" \
     -d "To=+0987654321" \
     -d "CallStatus=in-progress"
   ```

3. **Check Execution**
   - Watch the workflow execute step by step
   - Each node should turn green when successful
   - Fix any errors that appear

### 4.2 Real Phone Testing

1. **Call Your Twilio Number**
   - Use your mobile phone
   - Call the Twilio number you configured

2. **Test Conversation Flow**
   - Listen to the greeting
   - Say: "I want to book an appointment"
   - Provide information when asked:
     - Name: "John Doe"
     - Email: "john@example.com"
     - Date: "Tomorrow"
     - Time: "2 PM"
   - Confirm the booking

3. **Verify Results**
   - Check your Google Sheet for the new entry
   - Check your email for the confirmation
   - Check n8n executions for any errors

---

## Step 5: Customization Guide

### 5.1 Modify the Greeting Message

1. Click on **"Respond with Greeting"** node
2. Edit the `<Say>` text in the TwiML:
   ```xml
   <Say voice="Polly.Joanna">
       Hello! Welcome to [YOUR BUSINESS NAME].
       I'm your AI assistant. How can I help you today?
   </Say>
   ```
3. Save the node

### 5.2 Change Voice Settings

Available voices in Twilio:
- **Female voices**: Polly.Joanna, Polly.Kendra, Polly.Kimberly, Polly.Salli
- **Male voices**: Polly.Joey, Polly.Justin, Polly.Matthew

Change in any `<Say>` tag:
```xml
<Say voice="Polly.Matthew">Your message here</Say>
```

### 5.3 Modify AI Assistant Personality

1. Click on **"OpenAI - Process Intent"** node
2. Edit the System message:
   ```
   You are a [FRIENDLY/PROFESSIONAL/CASUAL] AI assistant for [BUSINESS NAME].
   
   Your personality:
   - [Warm and empathetic / Direct and efficient]
   - [Use casual language / Use formal language]
   - [Add humor / Stay serious]
   
   Your role is to:
   1. [Your specific tasks]
   2. [Your specific guidelines]
   ```

### 5.4 Add Business Hours Check

1. Add a **Function** node after "Parse Call Data"
2. Use this code:
   ```javascript
   const now = new Date();
   const hour = now.getHours();
   const day = now.getDay(); // 0 = Sunday
   
   const isBusinessHours = 
     day >= 1 && day <= 5 &&  // Monday to Friday
     hour >= 9 && hour < 17;  // 9 AM to 5 PM
   
   return {
     json: {
       ...$input.item.json,
       isBusinessHours: isBusinessHours
     }
   };
   ```

3. Add an **IF** node to route to different responses
4. For after-hours, create a TwiML response:
   ```xml
   <Response>
       <Say voice="Polly.Joanna">
           Thank you for calling. Our office hours are 
           Monday through Friday, 9 AM to 5 PM.
           Please call back during business hours, 
           or visit our website to book online.
       </Say>
       <Hangup/>
   </Response>
   ```

### 5.5 Add Available Time Slots

1. Add a **Google Sheets** node to read existing appointments
2. Add a **Function** node to calculate available slots:
   ```javascript
   // Get all appointments for requested date
   const requestedDate = $json.appointmentDate;
   const appointments = $node["Read Appointments"].json;
   
   // Define business hours
   const timeSlots = [
     "09:00 AM", "10:00 AM", "11:00 AM",
     "02:00 PM", "03:00 PM", "04:00 PM"
   ];
   
   // Filter out booked slots
   const bookedTimes = appointments
     .filter(apt => apt.Date === requestedDate)
     .map(apt => apt.Time);
   
   const available = timeSlots.filter(
     slot => !bookedTimes.includes(slot)
   );
   
   return {
     json: {
       ...$json,
       availableSlots: available,
       hasAvailability: available.length > 0
     }
   };
   ```

---

## Step 6: Monitoring & Debugging

### 6.1 View Workflow Executions

1. **Access Executions Panel**
   - In n8n, click **"Executions"** in the left sidebar
   - See all workflow runs with status (Success/Error)

2. **Debug Failed Executions**
   - Click on a failed execution
   - See which node failed
   - View error messages
   - Check input/output data for each node

### 6.2 Add Error Handling

1. **Add Error Trigger Node**
   - In workflow editor, click **"Add Node"**
   - Search for **"Error Trigger"**
   - This catches any workflow errors

2. **Add Notification on Error**
   - Connect to HTTP Request node
   - Send error to Slack/Discord/Email:
   ```javascript
   // Function node to format error
   const error = $json;
   return {
     json: {
       text: `🚨 Voice Agent Error\nWorkflow: ${error.workflow.name}\nError: ${error.error.message}`
     }
   };
   ```

### 6.3 View Logs in Real-Time

- **Webhook logs**: Check Twilio console for incoming requests
- **n8n logs**: Check executions panel for workflow runs
- **AI logs**: Check OpenAI usage dashboard

---

## Step 7: Going Live

### 7.1 Pre-Launch Checklist

- [ ] All credentials configured and tested
- [ ] Webhook URLs updated in Twilio
- [ ] Google Sheet is set up with correct columns
- [ ] Confirmation email template customized
- [ ] AI prompts tested and refined
- [ ] Business hours logic implemented (if needed)
- [ ] Error handling nodes added
- [ ] Workflow activated (toggle is ON)
- [ ] Test calls completed successfully
- [ ] Confirmation emails received
- [ ] Data saving to sheet correctly

### 7.2 Upgrade Twilio to Production

1. **Verify Your Business**
   - Complete Twilio business verification
   - This removes trial limitations

2. **Add Payment Method**
   - Add credit card to Twilio account
   - Set up auto-recharge

3. **Upgrade Account**
   - Remove trial restrictions
   - Enable production features

### 7.3 Monitor First Week

1. **Daily Checks**
   - Review all executions
   - Check for errors
   - Monitor response times
   - Review appointment data quality

2. **Gather Feedback**
   - Call the number yourself
   - Have team members test
   - Review customer feedback
   - Adjust AI prompts based on issues

3. **Optimize**
   - Refine AI responses
   - Adjust timeout values
   - Update available time slots
   - Improve error messages

---

## Troubleshooting Common Issues

### Issue 1: Workflow Not Triggering
**Symptoms**: Call comes in but nothing happens

**Solutions**:
1. Check workflow is **Active** (toggle is ON)
2. Verify webhook URL in Twilio matches n8n
3. Check n8n is accessible (not localhost for cloud Twilio)
4. Review Twilio debugger console for errors

### Issue 2: No Speech Recognition
**Symptoms**: AI doesn't hear what customer says

**Solutions**:
1. Check Twilio Speech settings in TwiML
2. Increase timeout value: `timeout="5"` to `timeout="10"`
3. Add `speechTimeout="auto"` parameter
4. Test with clear, loud speech

### Issue 3: AI Response Not Appropriate
**Symptoms**: AI gives wrong or irrelevant responses

**Solutions**:
1. Review and improve the system prompt
2. Add more examples to the prompt
3. Increase conversation context
4. Lower AI temperature for more consistent responses

### Issue 4: Appointments Not Saving
**Symptoms**: Execution succeeds but no data in sheet

**Solutions**:
1. Check Google Sheets credentials are valid
2. Verify spreadsheet ID is correct
3. Check sheet name matches exactly
4. Ensure columns match the mapping

### Issue 5: High Latency
**Symptoms**: Long pauses between speech and response

**Solutions**:
1. Use faster OpenAI model (gpt-3.5-turbo)
2. Reduce max_tokens in AI settings
3. Cache common responses
4. Consider using Claude Haiku for faster responses

### Issue 6: Confirmation Email Not Sending
**Symptoms**: Appointment books but no email

**Solutions**:
1. Check Gmail credentials are valid
2. Verify "from" email is correct
3. Check spam folder
4. Enable "Less secure app access" if needed
5. Use SendGrid instead of Gmail for higher volume

---

## Advanced Configurations

### Add Appointment Reminders

1. Create a new workflow: "Appointment Reminders"
2. Add a **Schedule Trigger** node (runs daily)
3. Query Google Sheets for tomorrow's appointments
4. Send SMS reminder via Twilio:
   ```javascript
   // For each appointment tomorrow
   // Send SMS with Twilio
   "Hi {{ name }}! Reminder: You have an appointment tomorrow at {{ time }}. See you then!"
   ```

### Add Calendar Integration

1. Add **Google Calendar** node after saving to Sheets
2. Create calendar event:
   - Summary: "Appointment - {{ $json.customerName }}"
   - Start: "{{ $json.appointmentDate }} {{ $json.appointmentTime }}"
   - Duration: 1 hour
   - Description: "Phone: {{ $json.customerPhone }}"

### Add CRM Integration

1. Add **HubSpot/Salesforce** node
2. Create contact if not exists
3. Create deal/opportunity for appointment
4. Add notes about the call

---

## Cost Optimization

### Reduce OpenAI Costs
- Use **gpt-3.5-turbo** instead of gpt-4 (10x cheaper)
- Set lower **max_tokens** (150 instead of 500)
- Cache common responses in n8n
- Use simpler prompts

### Reduce Twilio Costs
- Use Twilio's native TTS for simple messages
- Optimize call duration
- Set maximum call length
- Use regional phone numbers

### Optimize n8n Usage
- Use n8n cloud free tier for testing
- Self-host for production (cheaper for high volume)
- Implement caching where possible

---

## Support & Resources

### Getting Help
1. **n8n Community Forum**: https://community.n8n.io
2. **n8n Discord**: Join for real-time help
3. **Twilio Support**: https://support.twilio.com
4. **OpenAI Community**: https://community.openai.com

### Documentation
- **n8n Docs**: https://docs.n8n.io
- **Twilio Voice Docs**: https://www.twilio.com/docs/voice
- **OpenAI API Docs**: https://platform.openai.com/docs

### Video Tutorials
- Search YouTube for "n8n voice agent"
- Twilio voice tutorials
- n8n workflow examples

---

## Next Steps

1. **Start Small**: Begin with the basic workflow
2. **Test Thoroughly**: Make 10-20 test calls
3. **Iterate**: Refine based on results
4. **Add Features**: Implement advanced features one at a time
5. **Monitor**: Watch executions and optimize
6. **Scale**: Increase capacity as needed

---

## Success Checklist

After importing and configuring, you should be able to:
- ✅ Receive inbound calls
- ✅ Have AI respond with greeting
- ✅ Collect customer information
- ✅ Book appointments
- ✅ Save to Google Sheets
- ✅ Send confirmation emails
- ✅ View all data in one place

**Congratulations!** You now have a fully functional AI voice agent! 🎉

---

## Quick Reference: Node Functions

| Node Name | Purpose | Key Settings |
|-----------|---------|--------------|
| Webhook - Inbound Call | Receives call from Twilio | Path: `/inbound-call` |
| Parse Call Data | Extracts call information | Processes Twilio data |
| Respond with Greeting | Sends initial greeting | TwiML with Gather |
| Webhook - Process Speech | Receives customer speech | Path: `/process-speech` |
| Extract Speech | Gets transcribed text | Parses SpeechResult |
| OpenAI - Process Intent | AI processes request | Model: gpt-4, Temp: 0.7 |
| Parse AI Response | Extracts booking data | Detects name, email, etc |
| If Ready to Book | Routes based on completion | Checks all info present |
| Validate Appointment Data | Ensures data quality | Email/phone validation |
| Save to Google Sheets | Stores appointment | Appends row to sheet |
| Generate Confirmation | Creates message | Formats confirmation |
| Respond with Confirmation | Sends final message | TwiML response |
| Continue Conversation | Loops back for more info | Redirects to process-speech |
| Send Confirmation Email | Emails customer | Gmail with HTML template |

---

Remember: Start simple, test often, and iterate based on real usage!