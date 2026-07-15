# Beta Health Care — Focused API Endpoints

## Purpose and scope

This is the implementation guide for the current Beta Health Care API focus:

- user accounts and student/doctor profile management;
- JWT authentication;
- doctor availability and bookable time slots;
- appointment scheduling and lifecycle management;
- appointment-backed reviews;
- user notifications and notification preferences; and
- doctor-managed recurring appointments that students can only view.

The following areas are deliberately **out of scope** for this phase: medical records, consultation notes, prescriptions, department holidays, waiting queues, departments, staff management, dashboard analytics, and audit logs.

Base path: `/api`

All identifiers are MongoDB ObjectIds. All timestamps are ISO 8601 UTC timestamps. Date-only values use `YYYY-MM-DD`; recurring schedule times use the clinic time zone `Africa/Lagos` (WAT, UTC+01:00).

## Roles and access conventions

| Role | Meaning in this document |
| --- | --- |
| `ADMIN` | Creates, activates/deactivates, and views user accounts and their profiles. |
| `STUDENT` | Manages only their own profile and appointments; may view only their own recurring schedules. |
| `DOCTOR` | Manages only their own profile, availability, appointments, and recurring schedules. |

Unless an endpoint is explicitly public, send `Authorization: Bearer <accessToken>`.

`/me` always resolves from the JWT; callers must never submit their own `userId`, `studentId`, or `doctorId` to impersonate someone else.

## Common response conventions

Successful list endpoints return:

```json
{
  "data": [],
  "pagination": { "page": 1, "limit": 20, "total": 0 }
}
```

Errors return:

```json
{
  "error": "CONFLICT",
  "message": "The selected time slot is no longer available.",
  "details": {}
}
```

Use `400` for invalid input, `401` for missing/invalid authentication, `403` for an authenticated caller without access, `404` for an inaccessible or missing resource, and `409` for uniqueness, booking, or state-transition conflicts.

---

## 1. Authentication

Public registration is not provided. As stated in the project README, an administrator creates accounts and distributes temporary credentials.

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/auth/login` | Public | Authenticate an existing user. |
| `POST` | `/auth/logout` | Authenticated | End the current server-tracked session, if token/session revocation is enabled. |

### `POST /auth/login`

Request body:

```json
{ "email": "student@example.edu", "password": "password" }
```

Response `200`:

```json
{
  "token": "<jwt>",
  "user": { "id": "…", "email": "student@example.edu", "role": "STUDENT", "isActive": true }
}
```

---

## 2. User, student, and doctor management

### Administrative account management

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/admin/users` | `ADMIN` | Create a user account only. |
| `GET` | `/admin/users` | `ADMIN` | List users; filters: `role`, `isActive`, `search`, `page`, `limit`. |
| `GET` | `/admin/users/:userId` | `ADMIN` | Get a user and their linked profile. |
| `PATCH` | `/admin/users/:userId` | `ADMIN` | Change account state or reset a temporary password. |

`POST /admin/users` creates the account only. It accepts `email`, `role`, and an initial password; it does not accept a Student or Doctor profile. Email belongs to `User` only and must not be duplicated in Student/Doctor collections. The password is received over HTTPS, hashed before storage, and never returned in a response.

```json
{
  "email": "ada@example.edu",
  "role": "STUDENT",
  "password": "initial-temporary-password"
}
```

`PATCH /admin/users/:userId` may change `isActive` and reset the password. It must not change a user’s role after creation; role changes would break profile and appointment ownership. Deactivation is used instead of deleting a user with clinical scheduling history.

### Student profiles

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/students` | `STUDENT` | Create the caller’s student profile once. |
| `GET` | `/students/me` | `STUDENT` | Get the caller’s student profile. |
| `PATCH` | `/students/me` | `STUDENT` | Update permitted contact/profile fields. |
| `GET` | `/students/:studentId` | `ADMIN`, assigned `DOCTOR` | Get a student profile. A doctor needs an appointment with that student. |

The student profile is linked to the caller’s account from the JWT; `userId` is never submitted in the request. Students may update `phoneNumber`, `address`, and `emergencyContact`; administrators control `studentNumber` and the linked account.

### Doctor profiles

| Method | Path | Access | Purpose |
| --- | --- | --- |
| `POST` | `/doctors` | `DOCTOR` | Create the caller’s doctor profile once. |
| `GET` | `/doctors` | Authenticated | List active doctors; filters: `specialization`, `page`, `limit`. |
| `GET` | `/doctors/me` | `DOCTOR` | Get the caller’s full doctor profile. |
| `PATCH` | `/doctors/me` | `DOCTOR` | Update permitted professional/profile fields. |
| `GET` | `/doctors/:doctorId` | Authenticated | Get a public doctor profile. |

The doctor profile is linked to the caller’s account from the JWT; `userId` is never submitted in the request. Doctors cannot alter `licenseNumber`; administrators must handle corrections through the user-management process.

---

## 3. Doctor availability and bookable slots

Availability is a doctor-owned weekly schedule. A slot has `dayOfWeek`, `startTime`, `endTime`, `slotDurationMinutes`, and `isActive`.

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `GET` | `/doctors/:doctorId/availability` | Authenticated | View a doctor’s weekly availability. |
| `POST` | `/doctors/me/availability` | `DOCTOR` | Add one availability window. |
| `PATCH` | `/doctors/me/availability/:availabilityId` | owning `DOCTOR` | Edit one availability window. |
| `DELETE` | `/doctors/me/availability/:availabilityId` | owning `DOCTOR` | Remove one availability window. |
| `GET` | `/doctors/:doctorId/available-slots` | Authenticated | Calculate open slots for a date. Required query: `date`; optional: `durationMinutes`. |

Example creation body:

```json
{
  "dayOfWeek": "MONDAY",
  "startTime": "09:00",
  "endTime": "17:00",
  "slotDurationMinutes": 30,
  "isActive": true
}
```

Availability windows for the same doctor must not overlap. Changes that would make an existing future appointment invalid must return `409`; the doctor must first reschedule or cancel the affected appointments.

---

## 4. Appointments

### Appointment model and lifecycle

An appointment stores `studentId`, `doctorId`, `scheduledAt`, `durationMinutes`, `type`, `priority`, `reason`, optional `notes`, `status`, and optional `recurringAppointmentId`.

Allowed status values and transitions:

```text
BOOKED → CHECKED_IN → COMPLETED
BOOKED → CANCELLED
BOOKED → NO_SHOW
CHECKED_IN → CANCELLED (doctor/admin only, exceptional case)
```

This phase supports normal scheduled bookings only: `type` is always `BOOKING` and `priority` is always `0`. The server sets these values; it does not accept a client-provided priority.

### Endpoints

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/appointments` | `STUDENT`, `DOCTOR` | Create a one-off appointment. A student creates only for themself; a doctor may create for a student. |
| `GET` | `/appointments` | `STUDENT`, `DOCTOR`, `ADMIN` | List appointments within the caller’s scope. Filters: `status`, `from`, `to`, `studentId`, `doctorId`, `page`, `limit`. |
| `GET` | `/appointments/:appointmentId` | participant or `ADMIN` | Get appointment details. |
| `PATCH` | `/appointments/:appointmentId` | participant or `ADMIN` | Reschedule/update mutable details while the appointment is `BOOKED`. |
| `PATCH` | `/appointments/:appointmentId/check-in` | participant or `DOCTOR` | Mark the appointment checked in. |
| `PATCH` | `/appointments/:appointmentId/complete` | assigned `DOCTOR` | Mark it completed. |
| `PATCH` | `/appointments/:appointmentId/cancel` | participant or `ADMIN` | Cancel a future booked appointment. |
| `PATCH` | `/appointments/:appointmentId/no-show` | assigned `DOCTOR` | Mark a booked appointment as no-show. |

Student request example:

```json
{
  "doctorId": "…",
  "scheduledAt": "2026-08-10T09:00:00.000Z",
  "durationMinutes": 30,
  "type": "BOOKING",
  "reason": "General checkup"
}
```

Doctors creating for a student additionally provide `studentId`. Every create/reschedule is checked against active availability and existing overlapping appointments for the doctor. A student may not hold overlapping active appointments; appointments in `CANCELLED`, `COMPLETED`, and `NO_SHOW` do not block a new booking.

---

## 5. Recurring appointments

A recurring appointment is a doctor-owned scheduling instruction, not a single appointment. It has `doctorId`, `studentId`, `frequency` (`WEEKLY` or `MONTHLY`), `dayOfWeek` or `dayOfMonth`, `startTime`, `durationMinutes`, `startDate`, `endDate`, `isActive`, and `nextOccurrenceAt`.

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/doctors/me/recurring-appointments` | `DOCTOR` | Create a recurring schedule for a student. |
| `GET` | `/doctors/me/recurring-appointments` | `DOCTOR` | List the caller’s recurring schedules. |
| `PATCH` | `/doctors/me/recurring-appointments/:recurringAppointmentId` | owning `DOCTOR` | Edit or pause a schedule. |
| `DELETE` | `/doctors/me/recurring-appointments/:recurringAppointmentId` | owning `DOCTOR` | Cancel a schedule and prevent future generation. |
| `GET` | `/students/me/recurring-appointments` | `STUDENT` | View only the caller’s recurring schedules. |
| `GET` | `/students/me/recurring-appointments/:recurringAppointmentId` | `STUDENT` | View one of the caller’s schedules. |

Students have no create, update, pause, or delete route for recurring schedules.

When a doctor creates or changes a schedule, the server validates the recurrence against availability and conflicts before saving. A background job creates each child appointment inside a rolling booking horizon (for example, 90 days). Each child stores `recurringAppointmentId`; a unique index on `(recurringAppointmentId, scheduledAt)` prevents duplicate generation. If a future occurrence becomes unavailable, it is recorded as a skipped occurrence and the doctor is notified rather than silently double-booking it.

---

## 6. Reviews

Only the student in a **completed** appointment may create its review. There is exactly one review per appointment; the review stores `appointmentId`, `studentId`, `doctorId`, `rating` (1–5), `comment`, and timestamps.

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/appointments/:appointmentId/review` | appointment `STUDENT` | Create the appointment’s review after completion. |
| `GET` | `/doctors/:doctorId/reviews` | Authenticated | List doctor reviews; filters: `page`, `limit`. |
| `GET` | `/students/me/reviews` | `STUDENT` | List the caller’s own reviews. |

The server must reject a review with `409` unless the appointment is `COMPLETED`, belongs to the caller, and has no existing review. Doctor rating aggregates are computed from reviews (or maintained transactionally), never accepted from the client.

---

## 7. Notifications

Notifications are created internally by system events: account creation, appointment creation/reschedule/cancellation/status changes, recurring-schedule changes, and review availability. Clients do not receive a public endpoint to create arbitrary notifications.

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `GET` | `/notifications/me` | Authenticated | List the caller’s notifications; filters: `unreadOnly`, `type`, `page`, `limit`. |
| `PATCH` | `/notifications/:notificationId/read` | notification owner | Mark one notification as read. |
| `PATCH` | `/notifications/me/read-all` | Authenticated | Mark all of the caller’s notifications as read. |
| `GET` | `/notification-preferences/me` | Authenticated | Get notification preferences. |
| `PATCH` | `/notification-preferences/me` | Authenticated | Update channel preferences. |

Preference example:

```json
{
  "inApp": true,
  "email": true,
  "appointmentReminders": true,
  "recurringAppointmentUpdates": true
}
```

Notification preferences control delivery channels, not the creation of critical in-app audit notifications.

---

## 8. Request and response examples

The following objects are the canonical shapes for the endpoints above. Read endpoints return `200`; creation endpoints return `201` unless stated otherwise.

### Account management

`POST /admin/users` request:

```json
{
  "email": "ada@example.edu",
  "role": "STUDENT",
  "password": "initial-temporary-password"
}
```

Response:

```json
{
  "id": "userId",
  "email": "ada@example.edu",
  "role": "STUDENT",
  "isActive": true,
  "createdAt": "2026-07-15T10:00:00.000Z"
}
```

`GET /admin/users` response:

```json
{
  "data": [{ "id": "userId", "email": "ada@example.edu", "role": "STUDENT", "isActive": true, "profile": { "fullName": "Ada Okafor" } }],
  "pagination": { "page": 1, "limit": 20, "total": 1 }
}
```

`PATCH /admin/users/:userId` request and response:

```json
{ "isActive": false }
```

```json
{ "id": "userId", "email": "ada@example.edu", "role": "STUDENT", "isActive": false, "updatedAt": "2026-07-15T11:00:00.000Z" }
```

### Profiles

`POST /students` request:

```json
{
  "fullName": "Ada Okafor",
  "studentNumber": "MSE/2001/024",
  "phoneNumber": "+2348045678906",
  "dateOfBirth": "2005-03-15",
  "emergencyContact": { "name": "Garuba Abdusalam", "phoneNumber": "+2347065432188" },
  "address": "Asherifa Estate, Modakeke"
}
```

Response `201`:

```json
{ "id": "studentId", "userId": "userId", "studentNumber": "MSE/2001/024", "fullName": "Ada Okafor", "email": "ada@example.edu", "phoneNumber": "+2348045678906" }
```

`PATCH /students/me` request:

```json
{
  "phoneNumber": "+2348045678910",
  "emergencyContact": { "name": "Garuba Abdusalam", "phoneNumber": "+2347065432188" },
  "address": "New address"
}
```

`GET /students/me`, `GET /students/:studentId`, and a successful student update response:

```json
{
  "id": "studentId",
  "userId": "userId",
  "studentNumber": "MSE/2001/024",
  "fullName": "Ada Okafor",
  "email": "ada@example.edu",
  "phoneNumber": "+2348045678910",
  "emergencyContact": { "name": "Garuba Abdusalam", "phoneNumber": "+2347065432188" },
  "address": "New address"
}
```

`PATCH /doctors/me` request:

```json
{ "phoneNumber": "+2348000000000", "bio": "General practitioner", "qualifications": "MBBS" }
```

`POST /doctors` request:

```json
{ "fullName": "Dr. James Smith", "specialization": "General Medicine", "phoneNumber": "+2348000000000", "qualifications": "MBBS", "licenseNumber": "LIC123456", "bio": "General practitioner" }
```

`POST /doctors` returns `201`; `GET /doctors/me`, `GET /doctors/:doctorId`, and a successful doctor update return `200`, using this object:

```json
{ "id": "doctorId", "fullName": "Dr. James Smith", "specialization": "General Medicine", "phoneNumber": "+2348000000000", "qualifications": "MBBS", "bio": "General practitioner", "rating": 4.8 }
```

`GET /doctors` response:

```json
{
  "data": [{ "id": "doctorId", "fullName": "Dr. James Smith", "specialization": "General Medicine", "rating": 4.8 }],
  "pagination": { "page": 1, "limit": 20, "total": 1 }
}
```

### Availability

`POST /doctors/me/availability` request and response:

```json
{ "dayOfWeek": "MONDAY", "startTime": "09:00", "endTime": "17:00", "slotDurationMinutes": 30, "isActive": true }
```

```json
{ "id": "availabilityId", "doctorId": "doctorId", "dayOfWeek": "MONDAY", "startTime": "09:00", "endTime": "17:00", "slotDurationMinutes": 30, "isActive": true }
```

`PATCH /doctors/me/availability/:availabilityId` request:

```json
{ "startTime": "10:00", "endTime": "16:00", "isActive": true }
```

It returns the updated availability object. `DELETE /doctors/me/availability/:availabilityId` returns `204` without a body.

`GET /doctors/:doctorId/availability` response:

```json
{ "data": [{ "id": "availabilityId", "dayOfWeek": "MONDAY", "startTime": "09:00", "endTime": "17:00", "slotDurationMinutes": 30, "isActive": true }] }
```

`GET /doctors/:doctorId/available-slots?date=2026-08-10&durationMinutes=30` response:

```json
{ "doctorId": "doctorId", "date": "2026-08-10", "slots": [{ "start": "2026-08-10T09:00:00.000Z", "end": "2026-08-10T09:30:00.000Z" }] }
```

### Appointments

`POST /appointments` request from a student:

```json
{ "doctorId": "doctorId", "scheduledAt": "2026-08-10T09:00:00.000Z", "durationMinutes": 30, "type": "BOOKING", "reason": "General checkup" }
```

A doctor creating for a student includes `studentId`. Create response, single appointment response, and successful status/action response:

```json
{
  "id": "appointmentId",
  "studentId": "studentId",
  "doctorId": "doctorId",
  "scheduledAt": "2026-08-10T09:00:00.000Z",
  "durationMinutes": 30,
  "type": "BOOKING",
  "priority": 0,
  "reason": "General checkup",
  "status": "BOOKED",
  "recurringAppointmentId": null,
  "createdAt": "2026-07-15T10:00:00.000Z"
}
```

`GET /appointments` response is a paginated list of the object above. `PATCH /appointments/:appointmentId` request:

```json
{ "scheduledAt": "2026-08-10T10:00:00.000Z", "durationMinutes": 30, "reason": "Updated reason" }
```

The check-in, complete, and no-show endpoints have no request body. Cancel uses:

```json
{ "cancellationReason": "Unable to attend" }
```

### Recurring appointments

`POST /doctors/me/recurring-appointments` request:

```json
{ "studentId": "studentId", "frequency": "WEEKLY", "dayOfWeek": "MONDAY", "startTime": "10:00", "durationMinutes": 30, "startDate": "2026-08-03", "endDate": "2026-12-31" }
```

Create response, single recurring-schedule response, and list item:

```json
{ "id": "recurringAppointmentId", "doctorId": "doctorId", "studentId": "studentId", "frequency": "WEEKLY", "dayOfWeek": "MONDAY", "startTime": "10:00", "durationMinutes": 30, "startDate": "2026-08-03", "endDate": "2026-12-31", "isActive": true, "nextOccurrenceAt": "2026-08-03T10:00:00.000Z" }
```

`PATCH /doctors/me/recurring-appointments/:recurringAppointmentId` accepts mutable schedule fields, including `{ "isActive": false }` to pause it, and returns the updated object. `DELETE` returns `204` without a body. Both list routes return `{ "data": [<recurring schedule>] }`, with pagination where needed.

### Reviews

`POST /appointments/:appointmentId/review` request and response:

```json
{ "rating": 5, "comment": "Clear and helpful consultation." }
```

```json
{ "id": "reviewId", "appointmentId": "appointmentId", "studentId": "studentId", "doctorId": "doctorId", "rating": 5, "comment": "Clear and helpful consultation.", "createdAt": "2026-08-10T10:00:00.000Z" }
```

Both review list endpoints return a paginated `data` list of this review object.

### Notifications

`GET /notifications/me` response:

```json
{
  "data": [{ "id": "notificationId", "type": "APPOINTMENT_BOOKED", "title": "Appointment booked", "message": "Your appointment is confirmed.", "isRead": false, "createdAt": "2026-07-15T10:00:00.000Z" }],
  "pagination": { "page": 1, "limit": 20, "total": 1 }
}
```

The read endpoints return:

```json
{ "message": "Notifications marked as read" }
```

`GET /notification-preferences/me` response and `PATCH /notification-preferences/me` request:

```json
{ "inApp": true, "email": true, "appointmentReminders": true, "recurringAppointmentUpdates": true }
```

---

## Data-consistency rules to implement

1. **Account/profile separation:** `POST /admin/users` creates only the `User`. `POST /students` and `POST /doctors` create at most one matching profile for the authenticated user. Enforce unique `User.email`, `Student.studentNumber`, `Doctor.licenseNumber`, and one profile per user. Do not allow a user without the required profile to book an appointment, manage availability, or create recurring schedules.
2. **No duplicate bookings:** use a transaction plus a database-level conflict guard for doctor time ranges. A simple unique index on start time is insufficient for overlapping durations; validate range overlap while holding a transaction/session or model slots explicitly.
3. **Availability is not an appointment rewrite:** do not silently alter existing bookings when availability changes. Reject conflicting availability edits, as defined above.
4. **Recurring children remain traceable:** generated appointments retain their `recurringAppointmentId`. Cancelling a recurring schedule stops future generation but does not erase past or already-booked child appointments; the doctor must handle those explicitly.
5. **Review eligibility is server-enforced:** the UI must not be the only gate. Enforce appointment ownership, `COMPLETED` status, and unique `appointmentId` at the database level.
6. **Use soft deactivation, not user deletion:** deleting a user with appointments breaks references and history. Inactive users cannot log in or create new appointments, while existing records remain intact.

## Confirmed product decisions

1. Administrators create accounts with email, role, and password only. The password is hashed at rest and is never returned by the API.
2. Recurring schedules use the clinic time zone `Africa/Lagos` (WAT, UTC+01:00). The implementation must still choose and configure the booking-generation horizon; 90 days is an example, not a fixed decision.
3. Emergency and urgent appointment types are deferred with queue management. This focused phase supports normal `BOOKING` appointments only.
