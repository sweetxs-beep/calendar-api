---
sidebar_position: 1
---

# Calendar Events API Integration Guide

Events represent scheduled calendar items such as meetings, reminders, or appointments. Each event belongs to a specific calendar and can include details like start/end times, attendees, and colors for categorization.

For example, you can create a “Cristina Costea Interview” event on the team calendar, color it yellow (colorId: 5), invite two attendees (Lola and Pogany), and later add an additional participant (Pisook).

### Endpoints

- **GET** `/users/me/calendarList` - Returns the calendars on the user's calendar list
- **POST** `/calendars/{calendarId}/events` — Create an event  
- **GET** `/calendars/{calendarId}/events` - Returns events on the specified calendar
- **GET** `/calendars/{calendarId}/events/{eventId}` — Retrieve an event  
- **PATCH** `/calendars/{calendarId}/events/{eventId}` — Update (modifies only specified fields)  
- **GET** `/colors` — Returns the color definitions for calendars and events 

### The Calendar Event Resource

#### Attributes 
Attributes are descriptive properties or fields that define the structure and details of a resource (in this case, an event). 

| Field       | Type   | Description |
|-------------|--------|-------------|
| `id`        | string | Unique event identifier. |
| `summary`   | string | Title of the event. |
| `description` | string | Description of the event. |
| `start` / `end` | object | `{ dateTime, timeZone }` (RFC3339 format + IANA timezone). |
| `location`  | string | Optional location of the event. |
| `colorId`   | string | Event color (use `/colors` to fetch valid IDs). |
| `attendees` | array  | List of invited users (`{ email, optional, responseStatus }`). |
| `etag`      | string | Used for concurrency control with `If-Match`. |
| `htmlLink`  | string | Link to the event in the web UI. |
| `reminders` | object | Event reminders. Use `{ useDefault, overrides[] }`. |


**Example Event Resource JSON**
```json 
{
  "id": "evt_123",
  "summary": "Cristina Costea Interview",
  "description": "Scope & timelines",
  "start": { "dateTime": "2025-09-10T10:00:00", "timeZone": "Europe/Bucharest" },
  "end":   { "dateTime": "2025-09-10T11:00:00", "timeZone": "Europe/Bucharest" },
  "location": "The chamber of cats",
  "colorId": "5",
  "attendees": [
    { "email": "lola@example.com", "responseStatus": "needsAction" },
    { "email": "pogany@example.com",   "responseStatus": "needsAction" }
  ],
  "etag": "\"3912345678900\"",
  "htmlLink": "https://calendar.google.com/calendar/event?eid=..."
}
```

## Create a new event 

Create a new event in a specific calendar, with a specific color and attendees list. 

### 1. Find the calendar 

**GET** `/users/me/calendarList`

No path and body parameters are needed. 

**Example request** 
```bash
curl "https://www.googleapis.com/calendar/v3/users/me/calendarList" \
  -H "Authorization: Bearer $TOKEN"
  ```
This request returns the list of calendars. 

**Example JSON response**
```json 
{
  "items": [
    {
      "id": "primary",
      "summary": "My Main Calendar"
    },
    {
      "id": "your-team-cal@group.calendar.google.com",
      "summary": "Team Calendar"
    }
  ]
}
```
:::tip 
Note: Take note of the `id` value for the calendar you want to use.
:::
### 2. Create the event 

**POST** `/calendars/{calendarId}/events`

**Path parameters**
| Name        | Type   | Description |
|-------------|--------|-------------|
| `calendarId` | string | The calendar identifier. Use `"primary"` for the user’s main calendar or a specific calendar `id`. |

**Body parameters**
| Name         | Type   | Description |
|--------------|--------|-------------|
| `summary`    | string | Title of the event. |
| `description`| string | Optional description or notes. |
| `location`   | string | Optional location of the event. |
| `start`      | object | Start time `{ dateTime, timeZone }` in RFC3339 format. |
| `end`        | object | End time `{ dateTime, timeZone }` in RFC3339 format. |
| `attendees`  | array  | List of `{ email }` objects for attendees. |
| `colorId`    | string | Event color ID (from `/colors`). |
| `reminders`  | object | Event reminders. Use `{ useDefault, overrides[] }`. |

:::tip Note:
Call **GET** `/colors` first to fetch valid `colorId` values before creating or updating events. Using an invalid color will return an error.
:::

**Example request**
```bash
curl -X POST "https://www.googleapis.com/calendar/v3/calendars/your-team-cal@group.calendar.google.com/events" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "summary": "Cristina Costea Interview",
    "description": "Scope & timelines",
    "start": { "dateTime": "2025-09-10T10:00:00", "timeZone": "Europe/Bucharest" },
    "end":   { "dateTime": "2025-09-10T11:00:00", "timeZone": "Europe/Bucharest" },
    "location": "The chamber of cats",
    "colorId": "5",
    "attendees": [
      { "email": "lola@example.com" },
      { "email": "pogany@example.com" }
    ]
  }'
  ```
  This example creates an event in the calendar with the `id` `your-team-cal@group.calendar.google.com` (Team Calendar). It is titled “Cristina Costea Interview” and is scheduled to take place on September 10, 2025, from 10:00 AM to 11:00 AM (Europe/Bucharest time) at the location “The chamber of cats.” The event is marked with yellow (`colorId` 5) and has two invited attendees: lola@example.com and pogany@example.com. 

**Example JSON response**
```json 
{
  "id": "evt_123",
  "summary": "Cristina Costea Interview",
  "start": { "dateTime": "2025-09-10T10:00:00+03:00" },
  "end":   { "dateTime": "2025-09-10T11:00:00+03:00" },
  "colorId": "5",
  "attendees": [
    { "email": "lola@example.com", "responseStatus": "needsAction" },
    { "email": "pogany@example.com",   "responseStatus": "needsAction" }
  ],
  "etag": "\"3912345678900\"",
  "htmlLink": "https://calendar.google.com/calendar/event?eid=..."
}
```
---

## Editing an event  

Update a specific event by adding a new attendee. 

### 1. Find the event

**GET** `/calendars/{calendarId}/events`

Search for events within a calendar. You can filter by a keyword (`q`), or by date/time ranges (`timeMin`, `timeMax`). This is useful for locating the `eventId` before updating an event.
:::tip Note: 
If you don’t know the `eventId`, first list or search events to locate it.
:::
**Path parameters**

| Name        | Type   | Description |
|-------------|--------|-------------|
| `calendarId` | string | The calendar identifier. Use `"primary"` for the user’s main calendar or a specific calendar `id`. |

**Query parameters (common)**

| Name          | Type    | Description |
|---------------|---------|-------------|
| `q`           | string  | Full-text search string to match summary, description, or location. |
| `timeMin`     | string  | Lower bound (inclusive) for event start time (RFC3339). |
| `timeMax`     | string  | Upper bound (exclusive) for event start time (RFC3339). |
| `singleEvents`| boolean | Expands recurring events into individual instances. |
| `maxResults`  | integer | Limits the number of results per page. |

**Example request**

```bash
curl "https://www.googleapis.com/calendar/v3/calendars/your-team-cal@group.calendar.google.com/events?q=Cristina%20Costea%20Interview&timeMin=2025-09-01T00:00:00Z&timeMax=2025-09-30T23:59:59Z&singleEvents=true&maxResults=5" \
  -H "Authorization: Bearer $TOKEN"
```
This request searches for a particular event, based on the summary of the event (Cristina Costea Interview), in the interval of September 1, 2025 00:00 and September 30, 2025 23:59 inside the `your-team-cal@group.calendar.google.com` calendar. In case there are multiple events matching the filters, a `maxResults` of 5 events will be listed. 

**Example JSON response**
```json
{
  "items": [
    {
      "id": "evt_123",
      "summary": "Cristina Costea Interview",
      "start": { "dateTime": "2025-09-10T10:00:00+03:00" },
      "end":   { "dateTime": "2025-09-10T11:00:00+03:00" },
      "attendees": [
        { "email": "lola@example.com" },
        { "email": "pogany@example.com" }
      ],
      "etag": "\"3912345678900\""
    }
  ]
}
```
:::tip Note: 
Take note of the `id` (event ID) → `evt_123` and existing `attendees`. 
:::

### 2. Add an attendee to the event

**PATCH** `/calendars/{calendarId}/events/{eventId}`

Add a new attendee to an existing event.  

:::tip Note: 
When updating attendees, you must send the **entire updated list** (existing and new). `PATCH` replaces the full `attendees` array.
:::  

**Path parameters**
| Name        | Type   | Description |
|-------------|--------|-------------|
| `calendarId` | string | The calendar identifier. |
| `eventId`    | string | The unique ID of the event to update. |

**Body parameters (selected)**
| Name        | Type   | Description |
|-------------|--------|-------------|
| `attendees` | array  | Full list of attendees. This array replaces the existing one. |


**Example request**

```bash
curl -X PATCH "https://www.googleapis.com/calendar/v3/calendars/your-team-cal@group.calendar.google.com/events/evt_123" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -H 'If-Match: "3912345678900"' \
  -d '{
    "attendees": [
      { "email": "lola@example.com" },
      { "email": "pogany@example.com" },
      { "email": "pisook@example.com" }
    ]
  }'
```
This request adds one more attendee, `pisook@example.com`, to the event `evt_123`, in the `your-team-cal@group.calendar.google.com` calendar. 

**Example JSON response** 
```json
{
  "id": "evt_123",
  "summary": "Cristina Costea Interview",
  "attendees": [
    { "email": "lola@example.com",   "responseStatus": "needsAction" },
    { "email": "pogany@example.com",     "responseStatus": "needsAction" },
    { "email": "pisook@example.com", "responseStatus": "needsAction" }
  ],
  "etag": "\"3912345678955\""
}
```

![Suvoda](SuvodaLogo.svg)