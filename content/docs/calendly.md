# How to Implement Calendly Integration into Your Web Application

Integrating Calendly into your web application can streamline meeting scheduling and automatically sync calendar events with your CRM or internal systems. This guide walks through the complete implementation process, from OAuth authentication to webhook processing.

## Overview

A complete Calendly integration involves three main phases:
1. **OAuth Authentication** - Authorize your application to access Calendly data
2. **Webhook Subscription** - Set up real-time notifications for calendar events
3. **Webhook Processing** - Receive and process event data in your application

## Phase 1: Setting Up Your Calendly Application

### Create a Developer Account

First, you'll need to create a Calendly developer account and register your application:

1. Sign up for a developer account at [Calendly Developer Console](https://developer.calendly.com/console/apps)
2. Create a new OAuth application and configure:
   - **Application Name**: Your app's name
   - **Application Type**: Web or Native
   - **Environment**: Start with Sandbox for development, then create a Production app when ready
   - **Redirect URI**: 
     - Sandbox: HTTP with localhost is allowed (e.g., `http://localhost:1234`)
     - Production: Must use HTTPS (e.g., `https://yourdomain.com/auth/calendly`)

3. Save your credentials - you'll receive:
   - Client ID
   - Client Secret
   - Webhook Signing Key (critical for security - you can only view this once!)

**Documentation**: [Create a Developer Account](https://developer.calendly.com/create-a-developer-account)

## Phase 2: Implementing OAuth Authentication

### Step 1: Get Authorization Code

Redirect users to Calendly's authorization page to grant access to your application:

```
https://auth.calendly.com/oauth/authorize?client_id=YOUR_CLIENT_ID&response_type=code&redirect_uri=https://yourdomain.com/auth/calendly
```

**Parameters**:
- `client_id` - Your application's Client ID
- `response_type` - Always "code"
- `redirect_uri` - Must match the URI registered in your app settings

When users grant access, Calendly redirects to your specified URI with the authorization code:

```
https://yourdomain.com/auth/calendly?code=f04281d639d8248435378b0365de7bd1f53bf452eda187d5f1e07ae7f04546d6
```

**Documentation**: [Get Authorization Code](https://developer.calendly.com/api-docs/60a14408d6743-get-authorization-code)

### Step 2: Exchange Code for Access Token

Make a POST request to exchange the authorization code for an access token:

**Endpoint**: `POST https://auth.calendly.com/oauth/token`

**Request Body**:
```json
{
  "grant_type": "authorization_code",
  "code": "AUTHORIZATION_CODE_FROM_STEP_1",
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET",
  "redirect_uri": "https://yourdomain.com/auth/calendly"
}
```

**Response**:
```json
{
  "access_token": "eyJraWQiOiIxY2UxZTEzNjE...",
  "refresh_token": "eyJraWQiOiIxY2UxZTEzNjE...",
  "token_type": "Bearer",
  "expires_in": 7200,
  "scope": "default"
}
```

Store the access token securely - you'll need it for all subsequent API calls.

## Phase 3: Creating a Webhook Subscription

### Subscribe to Calendar Events

Create a webhook subscription to receive real-time notifications when meetings are scheduled, canceled, or modified.

**Endpoint**: `POST https://api.calendly.com/webhook_subscriptions`

**Headers**:
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request Body**:
```json
{
  "url": "https://yourdomain.com/api/webhook/calendly",
  "events": [
    "invitee.created",
    "invitee.canceled",
    "invitee_no_show.created",
    "invitee_no_show.deleted"
  ],
  "organization": "https://api.calendly.com/organizations/AAAAAAAAAAAAAAAA",
  "user": "https://api.calendly.com/users/BBBBBBBBBBBBBBBB",
  "scope": "user",
  "signing_key": "YOUR_WEBHOOK_SIGNING_KEY"
}
```

**Key Parameters**:
- `url` - Your server endpoint that will receive webhook notifications
- `events` - Array of events to subscribe to:
  - `invitee.created` - New meeting bookings
  - `invitee.canceled` - Meeting cancellations
  - `invitee_no_show.created` - No-show events
  - `routing_form_submission.created` - Routing form submissions (organization scope only)
- `scope` - Either "user" (individual events) or "organization" (all team events)
- `signing_key` - Your webhook signing key for payload verification

**Documentation**: 
- [Create Webhook Subscription](https://developer.calendly.com/api-docs/c1ddc06ce1f1b-create-webhook-subscription)
- [Webhook Subscriptions Guide](https://developer.calendly.com/receive-data-from-scheduled-events-in-real-time-with-webhook-subscriptions)

## Phase 4: Processing Webhook Notifications

### Webhook Payload Structure

When an event occurs, Calendly sends a POST request to your webhook URL with this structure:

```json
{
  "created_at": "2020-11-23T17:51:19.000000Z",
  "created_by": "https://api.calendly.com/users/AAAAAAAAAAAAAAAA",
  "event": "invitee.created",
  "payload": {
    "uri": "https://api.calendly.com/scheduled_events/AAAAAAAAAAAAAAAA/invitees/AAAAAAAAAAAAAAAA",
    "email": "test@example.com",
    "name": "John Doe",
    "status": "active",
    "timezone": "America/New_York",
    "scheduled_event": {
      "uri": "https://api.calendly.com/scheduled_events/GBGBDCAADAEDCRZ2",
      "name": "15 Minute Meeting",
      "status": "active",
      "start_time": "2019-08-24T14:15:22.123456Z",
      "end_time": "2019-08-24T14:15:22.123456Z",
      "location": {
        "type": "physical",
        "location": "string"
      },
      "event_memberships": [
        {
          "user": "https://api.calendly.com/users/GBGBDCAADAEDCRZ2",
          "user_email": "user@example.com",
          "user_name": "John Smith"
        }
      ],
      "event_guests": []
    }
  }
}
```

**Documentation**: [Webhook Payload](https://developer.calendly.com/api-docs/69c58da556b61-webhook-payload)

### Verifying Webhook Signatures

**Critical**: Always verify webhook signatures to ensure requests are authentic and haven't been tampered with.

Calendly includes a `Calendly-Webhook-Signature` header with each webhook:

```
Calendly-Webhook-Signature: t=1492774577,v1=5257a869e7ecebeda32affa62cdca3fa51cad7e77a0e56ff536d0ce8e108d8bd
```

**Verification Steps**:

1. Extract the timestamp (`t`) and signature (`v1`) from the header
2. Create the signed payload: `timestamp + "." + JSON.stringify(requestBody)`
3. Compute HMAC-SHA256 hash using your signing key
4. Compare the computed signature with the received signature
5. Verify the timestamp is within your tolerance window (e.g., 3 minutes)

**Example Implementation (Node.js)**:

```javascript
const crypto = require('crypto');

const webhookSigningKey = process.env.WEBHOOK_SIGNING_KEY;
const calendlySignature = req.get('Calendly-Webhook-Signature');

// Parse signature header
const { t, signature } = calendlySignature.split(',').reduce((acc, current) => {
  const [key, value] = current.split('=');
  if (key === 't') acc.t = value;
  if (key === 'v1') acc.signature = value;
  return acc;
}, { t: '', signature: '' });

// Create signed payload
const data = t + '.' + JSON.stringify(req.body);

// Compute expected signature
const expectedSignature = crypto
  .createHmac('sha256', webhookSigningKey)
  .update(data, 'utf8')
  .digest('hex');

// Verify signature matches
if (expectedSignature !== signature) {
  throw new Error('Invalid Signature');
}

// Prevent replay attacks - reject old timestamps
const threeMinutes = 180000;
const timestampMilliseconds = Number(t) * 1000;

if (timestampMilliseconds < Date.now() - threeMinutes) {
  throw new Error('Signature timestamp outside tolerance zone');
}
```

**Documentation**: [Webhook Signatures](https://developer.calendly.com/api-docs/4c305798a61d3-webhook-signatures)

## Phase 5: Processing Event Data

### Extracting Useful Information

From the webhook payload, you can extract:

**Event Details**:
- Meeting title (`payload.scheduled_event.name`)
- Start/end times (`payload.scheduled_event.start_time`, `end_time`)
- Location (`payload.scheduled_event.location`)
- Meeting notes (`payload.scheduled_event.meeting_notes_plain`)

**Invitee Information**:
- Name and email (`payload.name`, `payload.email`)
- First/last name (if configured separately)
- Timezone (`payload.timezone`)
- Custom question responses (`payload.questions_and_answers`)

**Host/Attendee Details**:
- Event memberships (`payload.scheduled_event.event_memberships`)
- Additional guests (`payload.scheduled_event.event_guests`)

### Common Implementation Patterns

**1. Contact Management**: Create or update contacts in your CRM from invitee data

**2. Calendar Synchronization**: Store events in your database with:
- URI-based unique identification (handles rescheduling)
- IsActive flags for soft deletion of canceled events
- Proper host/invitee relationship tracking

**3. Activity Logging**: Generate activity records for all event participants

**4. User Interface Integration**: Display events in:
- Main calendar views
- Contact activity feeds
- Dashboard widgets

## Architecture Recommendations

### Backend Components

**1. Webhook Controller**: 
- Validate signatures
- Deserialize payloads
- Route to appropriate handlers

**2. Event Manager**:
- Process event lifecycle (create, reschedule, cancel)
- Manage recipients and relationships
- Handle URI-based event tracking

**3. Data Layer**:
- Store calendar events
- Track event recipients (hosts, invitees, guests)
- Maintain integration settings

### Database Schema

Consider these tables:

**CalendarEvent**:
- URI (unique identifier)
- Title, start/end times, location
- Integration source
- IsActive flag for soft deletion

**CalendarEventRecipient**:
- Event relationship
- Contact information
- Role (host/invitee/guest)

**IntegrationSite**:
- User/organization association
- OAuth credentials
- Webhook signing keys (encrypted)

## Security Best Practices

1. **Always verify webhook signatures** - Never trust unverified webhooks
2. **Implement replay attack prevention** - Use timestamp validation
3. **Encrypt sensitive data** - Store API keys and signing keys encrypted
4. **Use HTTPS** - Required for production webhook endpoints
5. **Rate limiting** - Implement rate limits on your webhook endpoint
6. **Error handling** - Log errors but avoid exposing sensitive information

## Testing Your Integration

### Development Tips

1. Start with Calendly's **Sandbox environment** for testing
2. Use tools like **ngrok** to expose localhost for webhook testing
3. Log all webhook payloads during development
4. Test all event types: create, reschedule, cancel, no-show

### Common Gotchas

- **Rescheduling creates new events**: Use URI-based tracking to link rescheduled events
- **Signature verification timing**: Always validate before processing
- **Timezone handling**: Store times in UTC, display in user's timezone
- **Multiple guests**: Process both event_memberships and event_guests arrays

## Conclusion

Implementing Calendly integration provides seamless calendar synchronization and eliminates manual data entry. By following OAuth best practices, properly verifying webhooks, and handling all event types, you can build a robust integration that enhances your application's scheduling capabilities.

### Key Takeaways

- Register your app in Calendly Developer Console
- Implement proper OAuth flow with secure token storage
- Create webhook subscriptions for real-time event notifications
- Always verify webhook signatures to ensure authenticity
- Handle all event lifecycle stages (create, reschedule, cancel)
- Store events with URI-based unique identification

### Additional Resources

- [Calendly API Documentation](https://developer.calendly.com)
- [Developer Console](https://developer.calendly.com/console/apps)
- [API Reference](https://developer.calendly.com/api-docs)
- [Community Forum](https://community.calendly.com)

Happy integrating! 🚀