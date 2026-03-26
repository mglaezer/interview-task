# Task 4: The Dispatcher Trap (Notification Router)

## Goal

A CLI service that routes notifications to different providers (SMS, Email, Push).

## Steps

**Step 1.** Implement a simple router where `Type: SMS` goes to a Twilio mock, and `Type: Email` goes to an SMTP mock.

**Step 2.** Add "Priorities." High priority messages must go to ALL channels; Low priority only goes to Push.

## The Pivot (The Trap)

Add a "Failover" rule: "If the Email provider returns a 500 error, automatically retry via SMS, but only if the user has a phone number on file."

## The Catch

Without a design, the AI will create a **Circular Dependency**. The Email service will suddenly need to know about the SMS service and the User Database. The code becomes a "God Object."

## What to Look For

Does the candidate implement an **Observer** or a **Middleware/Dispatcher**? A good design involves a central "Dispatcher" that handles the logic, keeping the individual providers "dumb" and independent.
