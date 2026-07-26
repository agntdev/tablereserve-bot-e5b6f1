# Restaurant Reservation Bot — Bot specification

**Archetype:** booking

**Voice:** friendly and professional — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot for restaurant table reservations with real-time availability, booking/rescheduling/cancelation via buttons, reference code confirmation, pre-reservation reminders, and admin dashboard for owners to track bookings and capacity.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- restaurant guests
- restaurant owner/manager

## Success criteria

- Guest receives booking confirmation with reference code
- Owner receives instant admin notifications for new bookings/cancellations
- Automated reminders sent 2 hours pre-reservation
- Real-time availability tracking prevents overbooking

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with calendar and booking options
- **Book table** (button, actor: user, callback: booking:start) — Initiate reservation flow with calendar UI
  - inputs: date, time, party size
  - outputs: booking confirmation with reference code
- **Reschedule** (button, actor: user, callback: booking:reschedule) — Show available alternative slots for existing reservation
- **Cancel** (button, actor: user, callback: booking:cancel) — Cancel current reservation with confirmation prompt
- **Admin view** (button, actor: admin, callback: admin:dashboard) — Show owner dashboard with upcoming bookings and capacity

## Flows

### Reservation flow
_Trigger:_ /start or /book

1. Date selection via calendar
2. Time slot selection from available options
3. Party size input
4. Optional contact details collection
5. Confirmation screen with reference code

_Data touched:_ Reservation

### Reschedule flow
_Trigger:_ booking:reschedule

1. Show available alternative slots based on current booking
2. Select new slot with button confirmation
3. Update original booking status

_Data touched:_ Reservation

### Admin dashboard
_Trigger:_ admin:dashboard

1. Display daily summary of bookings
2. Show real-time capacity percentages
3. List upcoming reservations with no-show flagging options

_Data touched:_ Reservation, Table

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Reservation** _(retention: persistent)_ — Guest reservation with status tracking
  - fields: guest_name, phone, party_size, datetime, duration, tables, status, reference_code, reminder_sent
- **Table** _(retention: persistent)_ — Restaurant table configuration
  - fields: table_id, seats, capacity_type
- **OpeningHours** _(retention: persistent)_ — Restaurant operating hours per weekday
  - fields: day_of_week, start_time, end_time

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure tables/seats via admin menu
- Set custom opening hours
- Flag no-shows from dashboard
- Receive daily booking summaries

## Notifications

- Guest pre-reservation reminders (2h default)
- Owner instant notifications for new bookings/cancellations
- Daily admin summary of bookings

## Permissions & privacy

- Guest phone numbers stored securely
- Admin access restricted to configured chat ID
- Data retention aligned with GDPR requirements

## Edge cases

- Guest enters invalid date format
- Partial reservation info missing
- Concurrent booking attempts for same slot
- Last-minute bookings within reminder window

## Required tests

- End-to-end reservation flow with confirmation code
- Admin dashboard shows accurate capacity percentages
- No-show flagging triggers owner notification
- Reminder suppression for short-notice bookings

## Assumptions

- Default 90-minute dining duration
- 15-minute slot granularity
- 11:00-22:00 default opening hours
- Auto-table-splitting when only total capacity provided
