---
sidebar_position: 1
---

# Calendar API Overview   


Suvoda Calendar is an online service by Suvoda for creating, managing, and sharing events and appointments. It has features such as scheduling events, setting reminders, inviting guests, color-coding, and sharing calendars. 

An Application Programming Interface (API) is a set of rules and tools that allows two software programs to communicate with each other. Instead of clicking through the Suvoda Calendar interface, applications can use the Suvoda Calendar API to automatically create, read, update, and delete calendar data.

The Calendar Events API is designed specifically for adding and editing events. Rather than manually creating or changing events in the interface, applications can send requests with event details (such as the title, time, or attendees) and receive a response confirming the action. 

An API integration guide provides instructions, technical specs and a clear narrative for developers on how applications exchange data and communicate. 

## Creating an event with the API
Creating an event inside a specific calendar requires first identifying the calendar, then adding the event.  

1. **Find the calendar**. First, your application sends a `GET` request to the API. This request returns a list of calendars, each with unique IDs. 

2. **Create the event**. Then, once you have the `calendarId` you need, your application sends a `POST` request to the API. This request acts as a blueprint for the new event. The API reads this blueprint and creates the event in the specified calendar with all the details you provided.

- **The request** is what you send. It includes information such as calendar IDs, event title, start and end times, the color you’d like to assign, and a list of attendees. 

- **The response** is what you receive. The API responds with a confirmation that the event was created with the unique `eventId` that was assigned to it. This ID is needed to update or delete the event.
:::important 
To use the Suvoda Calendar API, your application must authenticate with a valid access token that includes the necessary scopes (permissions) for the intended action. Missing scopes will result in an authorization error.
:::
:::note Example
To view, edit, share, or permanently delete calendars, use a scope such as `https://www.googleapis.com/auth/calendar`. To view events only, use `https://www.googleapis.com/auth/calendar.readonly`. 
:::

## Editing an event with the API 
Editing an event requires first identifying the event, then sending the update request. 

1. **Find the event**. First, tell the API which event you want to change. You do this by making a `GET` request to retrieve the event's unique `eventId`. You can search for events within a specific calendar by their title or date range.

2. **Edit the event**. Then, send the update through a `PATCH` request (used to modify an existing item). The request body needs to contain only the fields you want to change.  If you only want to add a new attendee, send the `attendees` field in the request. The API then updates only that specific part of the event without affecting the other details like the title or time.


![Suvoda](SuvodaLogo.svg)