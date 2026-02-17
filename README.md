# Doctor Appointment Scheduling System - API Endpoints

## Base URL
```
http://localhost:3000/api
```

---

## Authentication Endpoints

### 1. Register User
**POST** `/auth/register`

**Description:** Create a new user account

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "hashedPassword123",
  "role": "STUDENT"
}
```

**Response (201):**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "STUDENT",
  "createdAt": "2026-02-11T10:00:00Z"
}
```

---

### 2. Login User
**POST** `/auth/login`

**Description:** Authenticate user and return JWT token

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "hashedPassword123"
}
```

**Response (200):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "email": "john@example.com",
    "role": "STUDENT"
  }
}
```

---

### 3. Logout User
**POST** `/auth/logout`

**Description:** Invalidate user session/token

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "message": "Logged out successfully"
}
```

---

## Student Endpoints

### 4. Create Student Profile
**POST** `/students`

**Description:** Create a student profile linked to user account

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "userId": 1,
  "studentNumber": "STU2026001",
  "fullName": "John Doe",
  "phoneNumber": "+1234567890",
  "email": "john@example.com",
  "dateOfBirth": "2005-03-15",
  "emergencyContact": "Jane Doe",
  "emergencyPhone": "+0987654321",
  "address": "123 Main St, City, State"
}
```

**Response (201):**
```json
{
  "id": 1,
  "userId": 1,
  "studentNumber": "STU2026001",
  "fullName": "John Doe",
  "createdAt": "2026-02-11T10:00:00Z"
}
```

---

### 5. Get Student Profile
**GET** `/students/:studentId`

**Description:** Retrieve student profile information

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "userId": 1,
  "studentNumber": "STU2026001",
  "fullName": "John Doe",
  "phoneNumber": "+1234567890",
  "email": "john@example.com",
  "dateOfBirth": "2005-03-15",
  "emergencyContact": "Jane Doe",
  "emergencyPhone": "+0987654321",
  "address": "123 Main St, City, State",
  "createdAt": "2026-02-11T10:00:00Z",
  "updatedAt": "2026-02-11T10:00:00Z"
}
```

---

### 6. Update Student Profile
**PUT** `/students/:studentId`

**Description:** Update student profile information

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "fullName": "John Updated",
  "phoneNumber": "+1234567890",
  "emergencyContact": "Jane Doe",
  "emergencyPhone": "+0987654321",
  "address": "456 Oak Ave, City, State"
}
```

**Response (200):**
```json
{
  "id": 1,
  "studentNumber": "STU2026001",
  "fullName": "John Updated",
  "updatedAt": "2026-02-11T11:00:00Z"
}
```

---

### 7. Get Student Appointments
**GET** `/students/:studentId/appointments`

**Description:** Retrieve all appointments for a student

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `status` (optional): Filter by status (BOOKED, CANCELLED, COMPLETED, NO_SHOW, CHECKED_IN)
- `page` (optional): Page number (default: 1)
- `limit` (optional): Results per page (default: 10)

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "studentId": 1,
      "doctorId": 5,
      "scheduledAt": "2026-02-15T14:00:00Z",
      "durationMinutes": 30,
      "reason": "General Checkup",
      "status": "BOOKED",
      "createdAt": "2026-02-11T10:00:00Z",
      "doctor": {
        "id": 5,
        "fullName": "Dr. Smith",
        "specialization": "General Medicine"
      }
    }
  ],
  "total": 5,
  "page": 1,
  "limit": 10
}
```

---

## Doctor Endpoints

### 8. Get All Doctors
**GET** `/doctors`

**Description:** Retrieve list of all doctors with filters

**Query Parameters:**
- `specialization` (optional): Filter by specialization
- `isAvailable` (optional): Filter by availability status
- `page` (optional): Page number (default: 1)
- `limit` (optional): Results per page (default: 10)

**Response (200):**
```json
{
  "data": [
    {
      "id": 5,
      "fullName": "Dr. James Smith",
      "specialization": "General Medicine",
      "phoneNumber": "+1111111111",
      "bio": "Experienced physician with 10+ years",
      "qualifications": "MD, Board Certified",
      "rating": 4.8,
      "isAvailable": true,
      "createdAt": "2026-01-01T10:00:00Z"
    }
  ],
  "total": 15,
  "page": 1,
  "limit": 10
}
```

---

### 9. Get Doctor Profile
**GET** `/doctors/:doctorId`

**Description:** Retrieve detailed doctor profile

**Response (200):**
```json
{
  "id": 5,
  "userId": 2,
  "fullName": "Dr. James Smith",
  "specialization": "General Medicine",
  "phoneNumber": "+1111111111",
  "bio": "Experienced physician with 10+ years of practice",
  "qualifications": "MD, Board Certified",
  "licenseNumber": "LIC123456",
  "rating": 4.8,
  "isAvailable": true,
  "createdAt": "2026-01-01T10:00:00Z"
}
```

---

### 10. Create Doctor Availability
**POST** `/doctors/:doctorId/availability`

**Description:** Add available time slots for a doctor

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "dayOfWeek": "MONDAY",
  "startTime": "09:00",
  "endTime": "17:00",
  "isActive": true
}
```

**Response (201):**
```json
{
  "id": 1,
  "doctorId": 5,
  "dayOfWeek": "MONDAY",
  "startTime": "09:00",
  "endTime": "17:00",
  "isActive": true,
  "createdAt": "2026-02-11T10:00:00Z"
}
```

---

### 11. Get Doctor Availability
**GET** `/doctors/:doctorId/availability`

**Description:** Retrieve all availability slots for a doctor

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "doctorId": 5,
      "dayOfWeek": "MONDAY",
      "startTime": "09:00",
      "endTime": "17:00",
      "isActive": true
    },
    {
      "id": 2,
      "doctorId": 5,
      "dayOfWeek": "TUESDAY",
      "startTime": "10:00",
      "endTime": "18:00",
      "isActive": true
    }
  ]
}
```

---

### 12. Update Doctor Availability
**PUT** `/doctors/:doctorId/availability/:availabilityId`

**Description:** Update an availability slot

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "startTime": "09:30",
  "endTime": "17:30",
  "isActive": true
}
```

**Response (200):**
```json
{
  "id": 1,
  "doctorId": 5,
  "dayOfWeek": "MONDAY",
  "startTime": "09:30",
  "endTime": "17:30",
  "isActive": true
}
```

---

### 13. Delete Doctor Availability
**DELETE** `/doctors/:doctorId/availability/:availabilityId`

**Description:** Remove an availability slot

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "message": "Availability deleted successfully"
}
```

---

### 14. Get Doctor Appointments
**GET** `/doctors/:doctorId/appointments`

**Description:** Retrieve all appointments for a doctor

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `status` (optional): Filter by status
- `date` (optional): Filter by specific date (YYYY-MM-DD)
- `page` (optional): Page number
- `limit` (optional): Results per page

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "studentId": 1,
      "doctorId": 5,
      "scheduledAt": "2026-02-15T14:00:00Z",
      "durationMinutes": 30,
      "reason": "General Checkup",
      "status": "BOOKED",
      "student": {
        "id": 1,
        "fullName": "John Doe",
        "studentNumber": "STU2026001"
      }
    }
  ],
  "total": 8,
  "page": 1
}
```

---

## Department Management Endpoints

### 15. Get All Departments
**GET** `/departments`

**Description:** Retrieve all departments

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "name": "General Medicine",
      "description": "General medical services",
      "location": "Building A, Floor 2",
      "phoneNumber": "+1234567890"
    }
  ]
}
```

---

### 16. Create Department
**POST** `/departments`

**Description:** Create a new department (admin only)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "name": "Cardiology",
  "description": "Heart and cardiovascular services",
  "location": "Building B, Floor 3",
  "phoneNumber": "+1234567890"
}
```

**Response (201):**
```json
{
  "id": 2,
  "name": "Cardiology",
  "description": "Heart and cardiovascular services",
  "location": "Building B, Floor 3"
}
```

---

## Medical Records Endpoints

### 17. Get Student Medical Records
**GET** `/students/:studentId/medical-record`

**Description:** Retrieve student's medical history

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "studentId": 1,
  "bloodType": "O_POSITIVE",
  "allergies": "Penicillin, Shellfish",
  "chronicConditions": "Asthma, Hypertension",
  "currentMedications": "Albuterol inhaler, Lisinopril",
  "medicalHistory": "Previous appendectomy in 2020",
  "lastUpdated": "2026-02-10T10:00:00Z"
}
```

---

### 18. Create/Update Medical Record
**POST** `/students/:studentId/medical-record`

**Description:** Create or update student medical record

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "bloodType": "O_POSITIVE",
  "allergies": "Penicillin, Shellfish",
  "chronicConditions": "Asthma",
  "currentMedications": "Albuterol inhaler",
  "medicalHistory": "Appendectomy in 2020"
}
```

**Response (201/200):**
```json
{
  "id": 1,
  "studentId": 1,
  "bloodType": "O_POSITIVE",
  "allergies": "Penicillin, Shellfish",
  "lastUpdated": "2026-02-11T10:00:00Z"
}
```

---

## Consultation Notes Endpoints

### 19. Add Consultation Note
**POST** `/appointments/:appointmentId/consultation-note`

**Description:** Add doctor's consultation notes (after appointment)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "symptoms": "Fever, cough, sore throat",
  "diagnosis": "Common cold with mild bronchitis",
  "treatment": "Rest, fluids, paracetamol",
  "notes": "Follow up if symptoms persist",
  "followUpDate": "2026-02-22T10:00:00Z"
}
```

**Response (201):**
```json
{
  "id": 1,
  "appointmentId": 1,
  "doctorId": 5,
  "symptoms": "Fever, cough, sore throat",
  "diagnosis": "Common cold with mild bronchitis",
  "treatment": "Rest, fluids, paracetamol",
  "followUpDate": "2026-02-22T10:00:00Z",
  "createdAt": "2026-02-15T14:30:00Z"
}
```

---

### 20. Get Consultation Note
**GET** `/appointments/:appointmentId/consultation-note`

**Description:** Retrieve consultation notes for an appointment

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "appointmentId": 1,
  "doctorId": 5,
  "symptoms": "Fever, cough, sore throat",
  "diagnosis": "Common cold",
  "treatment": "Rest, fluids, paracetamol",
  "notes": "Follow up if symptoms persist",
  "followUpDate": "2026-02-22T10:00:00Z",
  "createdAt": "2026-02-15T14:30:00Z"
}
```

---

## Prescription Endpoints

### 21. Add Prescription
**POST** `/appointments/:appointmentId/prescriptions`

**Description:** Add prescription for a student

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "medicineName": "Amoxicillin",
  "dosage": "500mg",
  "frequency": "Three times daily",
  "duration": "7 days",
  "instructions": "Take with food",
  "notes": "Complete the full course"
}
```

**Response (201):**
```json
{
  "id": 1,
  "appointmentId": 1,
  "doctorId": 5,
  "studentId": 1,
  "medicineName": "Amoxicillin",
  "dosage": "500mg",
  "frequency": "Three times daily",
  "duration": "7 days",
  "issuedAt": "2026-02-15T14:30:00Z",
  "expiresAt": "2026-02-22T14:30:00Z"
}
```

---

### 22. Get Student Prescriptions
**GET** `/students/:studentId/prescriptions`

**Description:** Get all prescriptions for a student

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `active` (optional): Show only active/unexpired prescriptions
- `page` (optional): Page number
- `limit` (optional): Results per page

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "medicineName": "Amoxicillin",
      "dosage": "500mg",
      "frequency": "Three times daily",
      "duration": "7 days",
      "issuedAt": "2026-02-15T14:30:00Z",
      "expiresAt": "2026-02-22T14:30:00Z",
      "doctor": {
        "id": 5,
        "fullName": "Dr. James Smith"
      }
    }
  ],
  "total": 5,
  "page": 1
}
```

---

## Review & Rating Endpoints

### 23. Add Review
**POST** `/appointments/:appointmentId/review`

**Description:** Add review for doctor after appointment completion

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "rating": 5,
  "comment": "Doctor was very professional and helpful"
}
```

**Response (201):**
```json
{
  "id": 1,
  "appointmentId": 1,
  "doctorId": 5,
  "studentId": 1,
  "rating": 5,
  "comment": "Doctor was very professional and helpful",
  "createdAt": "2026-02-15T15:00:00Z"
}
```

---

### 24. Get Doctor Reviews
**GET** `/doctors/:doctorId/reviews`

**Description:** Get all reviews for a doctor

**Query Parameters:**
- `page` (optional): Page number
- `limit` (optional): Results per page
- `sortBy` (optional): newest, oldest, highest, lowest

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "rating": 5,
      "comment": "Very professional",
      "studentName": "John Doe",
      "createdAt": "2026-02-15T15:00:00Z"
    }
  ],
  "total": 42,
  "averageRating": 4.7,
  "page": 1
}
```

---

### 25. Get Student Reviews
**GET** `/students/:studentId/reviews`

**Description:** Get reviews written by a student

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "rating": 5,
      "comment": "Great doctor",
      "doctor": {
        "id": 5,
        "fullName": "Dr. James Smith"
      },
      "createdAt": "2026-02-15T15:00:00Z"
    }
  ]
}
```

---

## Notification Endpoints

### 26. Get User Notifications
**GET** `/users/:userId/notifications`

**Description:** Get all notifications for a user

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `status` (optional): PENDING, SENT, FAILED, OPENED
- `type` (optional): EMAIL, SMS, IN_APP
- `unreadOnly` (optional): Show only unread notifications

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "type": "EMAIL",
      "title": "Appointment Reminder",
      "message": "Your appointment is in 1 hour",
      "status": "SENT",
      "appointmentId": 1,
      "sentAt": "2026-02-15T13:00:00Z",
      "openedAt": null,
      "createdAt": "2026-02-15T13:00:00Z"
    }
  ],
  "total": 10,
  "unreadCount": 3
}
```

---

### 27. Get Notification Preferences
**GET** `/users/:userId/notification-preferences`

**Description:** Get user's notification preferences

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "userId": 1,
  "emailNotifications": true,
  "smsNotifications": false,
  "inAppNotifications": true,
  "appointmentReminder": true,
  "reminderMinutesBefore": 60
}
```

---

### 28. Update Notification Preferences
**PUT** `/users/:userId/notification-preferences`

**Description:** Update user's notification settings

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "emailNotifications": true,
  "smsNotifications": true,
  "inAppNotifications": true,
  "appointmentReminder": true,
  "reminderMinutesBefore": 30
}
```

**Response (200):**
```json
{
  "id": 1,
  "userId": 1,
  "emailNotifications": true,
  "smsNotifications": true,
  "inAppNotifications": true,
  "appointmentReminder": true,
  "reminderMinutesBefore": 30
}
```

---

### 29. Mark Notification as Read
**PATCH** `/notifications/:notificationId/read`

**Description:** Mark a notification as opened/read

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "status": "OPENED",
  "openedAt": "2026-02-15T13:05:00Z"
}
```

---

## Recurring Appointments Endpoints

### 30. Create Recurring Appointment
**POST** `/recurring-appointments`

**Description:** Set up recurring appointments

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "studentId": 1,
  "doctorId": 5,
  "recurrenceType": "WEEKLY",
  "frequency": 1,
  "startDate": "2026-02-15T10:00:00Z",
  "endDate": "2026-06-15T10:00:00Z",
  "totalOccurrences": 12,
  "reason": "Chronic condition follow-up",
  "notes": "Monthly checkups"
}
```

**Recurrence Types:**
- `NONE` - One time only
- `DAILY` - Every day
- `WEEKLY` - Every week
- `MONTHLY` - Every month
- `YEARLY` - Every year

**Response (201):**
```json
{
  "id": 1,
  "studentId": 1,
  "doctorId": 5,
  "recurrenceType": "WEEKLY",
  "frequency": 1,
  "startDate": "2026-02-15T10:00:00Z",
  "endDate": "2026-06-15T10:00:00Z",
  "totalOccurrences": 12,
  "nextOccurrenceDate": "2026-02-22T10:00:00Z",
  "reason": "Chronic condition follow-up",
  "isActive": true
}
```

---

### 31. Get Recurring Appointments
**GET** `/students/:studentId/recurring-appointments`

**Description:** Get recurring appointments for a student

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "doctorId": 5,
      "doctor": {
        "fullName": "Dr. James Smith",
        "specialization": "General Medicine"
      },
      "recurrenceType": "WEEKLY",
      "frequency": 1,
      "nextOccurrenceDate": "2026-02-22T10:00:00Z",
      "isActive": true
    }
  ]
}
```

---

### 32. Update Recurring Appointment
**PUT** `/recurring-appointments/:recurringId`

**Description:** Update recurring appointment settings

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "frequency": 2,
  "endDate": "2026-08-15T10:00:00Z",
  "isActive": true
}
```

**Response (200):**
```json
{
  "id": 1,
  "frequency": 2,
  "endDate": "2026-08-15T10:00:00Z",
  "isActive": true,
  "nextOccurrenceDate": "2026-03-01T10:00:00Z"
}
```

---

### 33. Cancel Recurring Appointment
**DELETE** `/recurring-appointments/:recurringId`

**Description:** Stop recurring appointments

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "message": "Recurring appointment cancelled"
}
```

---

## Holiday & Schedule Endpoints

### 34. Get Department Holidays
**GET** `/departments/:departmentId/holidays`

**Description:** Get holidays for a department

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "title": "New Year",
      "description": "New Year holiday",
      "startDate": "2026-01-01",
      "endDate": "2026-01-01",
      "allDay": true
    }
  ]
}
```

---

### 35. Add Department Holiday
**POST** `/departments/:departmentId/holidays`

**Description:** Add holiday/closure for department

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "title": "Spring Break",
  "description": "Clinic closed",
  "startDate": "2026-03-15",
  "endDate": "2026-03-22",
  "allDay": true
}
```

**Response (201):**
```json
{
  "id": 1,
  "departmentId": 1,
  "title": "Spring Break",
  "startDate": "2026-03-15",
  "endDate": "2026-03-22"
}
```

---

## File Attachment Endpoints

### 36. Upload Attachment
**POST** `/attachments/upload`

**Description:** Upload medical documents or prescriptions

**Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Request Body:**
```
- file: <binary file>
- appointmentId: 1 (optional)
- prescriptionId: 1 (optional)
- type: MEDICAL_REPORT | PRESCRIPTION | LABS | OTHER
```

**Response (201):**
```json
{
  "id": 1,
  "fileName": "lab_results.pdf",
  "fileUrl": "https://storage.example.com/files/lab_results.pdf",
  "fileSize": 254321,
  "fileType": "application/pdf",
  "appointmentId": 1,
  "uploadedBy": 5,
  "createdAt": "2026-02-15T14:30:00Z"
}
```

---

### 37. Get Appointment Attachments
**GET** `/appointments/:appointmentId/attachments`

**Description:** Get all attachments for an appointment

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "fileName": "lab_results.pdf",
      "fileUrl": "https://storage.example.com/files/lab_results.pdf",
      "fileType": "application/pdf",
      "uploadedBy": 5,
      "createdAt": "2026-02-15T14:30:00Z"
    }
  ]
}
```

---

### 38. Delete Attachment
**DELETE** `/attachments/:attachmentId`

**Description:** Remove an attachment

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "message": "Attachment deleted successfully"
}
```

---

## Audit Log Endpoints

### 39. Get Audit Logs
**GET** `/admin/audit-logs`

**Description:** Get system action history

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `userId` (optional): Filter by user
- `action` (optional): Filter by action type
- `entityType` (optional): Filter by entity type (User, Appointment, Doctor, etc.)
- `startDate` (optional): From date
- `endDate` (optional): To date
- `page` (optional): Page number
- `limit` (optional): Results per page

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "userId": 1,
      "action": "appointment_booked",
      "entityType": "Appointment",
      "entityId": 5,
      "oldValue": null,
      "newValue": "{\"status\": \"BOOKED\"}",
      "description": "Student booked appointment",
      "ipAddress": "192.168.1.1",
      "createdAt": "2026-02-15T14:30:00Z"
    }
  ],
  "total": 150,
  "page": 1
}
```

---

### 40. Get User Activity
**GET** `/admin/users/:userId/activity-log`

**Description:** Get detailed activity log for a specific user

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "userId": 1,
  "userName": "John Doe",
  "activities": [
    {
      "id": 1,
      "action": "appointment_booked",
      "entityType": "Appointment",
      "description": "Booked appointment with Dr. Smith",
      "timestamp": "2026-02-15T14:30:00Z"
    }
  ],
  "total": 25
}
```

---

## Appointment Endpoints

### 41. Get Available Time Slots
**GET** `/appointments/available-slots`

**Description:** Get available appointment slots for a doctor on a specific date

**Query Parameters:**
- `doctorId` (required): Doctor ID
- `date` (required): Date in format YYYY-MM-DD
- `durationMinutes` (optional): Duration of appointment (default: 30)

**Response (200):**
```json
{
  "doctorId": 5,
  "date": "2026-02-15",
  "slots": [
    {
      "startTime": "09:00",
      "endTime": "09:30"
    },
    {
      "startTime": "09:30",
      "endTime": "10:00"
    },
    {
      "startTime": "14:00",
      "endTime": "14:30"
    }
  ]
}
```

---

### 42. Create Appointment
**POST** `/appointments`

**Description:** Book an appointment

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "studentId": 1,
  "doctorId": 5,
  "scheduledAt": "2026-02-15T14:00:00Z",
  "durationMinutes": 30,
  "type": "BOOKING",
  "priority": 0,
  "reason": "General Checkup",
  "notes": "Follow-up from previous visit"
}
```

**Appointment Types:**
- `BOOKING` - Routine checkup, can come anytime
- `NORMAL_SICKNESS` - Urgent but not emergency, want to come soon
- `EMERGENCY` - Critical, needs immediate attention

**Priority Levels:**
- `0` - Normal
- `1` - Urgent (NORMAL_SICKNESS)
- `2` - Emergency

**Response (201):**
```json
{
  "id": 1,
  "studentId": 1,
  "doctorId": 5,
  "scheduledAt": "2026-02-15T14:00:00Z",
  "durationMinutes": 30,
  "type": "BOOKING",
  "priority": 0,
  "reason": "General Checkup",
  "status": "BOOKED",
  "queuePosition": null,
  "createdAt": "2026-02-11T10:00:00Z"
}
```

---

### 43. Get Appointment Details
**GET** `/appointments/:appointmentId`

**Description:** Retrieve full appointment details

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "studentId": 1,
  "doctorId": 5,
  "scheduledAt": "2026-02-15T14:00:00Z",
  "durationMinutes": 30,
  "type": "BOOKING",
  "priority": 0,
  "reason": "General Checkup",
  "notes": "Follow-up from previous visit",
  "status": "BOOKED",
  "queuePosition": null,
  "checkedInAt": null,
  "cancelledAt": null,
  "completedAt": null,
  "createdAt": "2026-02-11T10:00:00Z",
  "updatedAt": "2026-02-11T10:00:00Z",
  "student": {
    "id": 1,
    "fullName": "John Doe"
  },
  "doctor": {
    "id": 5,
    "fullName": "Dr. James Smith"
  }
}
```

---

### 44. Update Appointment
**PUT** `/appointments/:appointmentId`

**Description:** Update appointment details (before scheduled time)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "reason": "Updated reason",
  "notes": "Updated notes"
}
```

**Response (200):**
```json
{
  "id": 1,
  "reason": "Updated reason",
  "notes": "Updated notes",
  "updatedAt": "2026-02-11T11:00:00Z"
}
```

---

### 45. Check In Appointment
**PATCH** `/appointments/:appointmentId/check-in`

**Description:** Mark student as checked in

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "status": "CHECKED_IN",
  "checkedInAt": "2026-02-15T14:00:00Z",
  "updatedAt": "2026-02-15T14:00:00Z"
}
```

---

### 46. Complete Appointment
**PATCH** `/appointments/:appointmentId/complete`

**Description:** Mark appointment as completed

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body (optional):**
```json
{
  "notes": "Patient responded well to treatment"
}
```

**Response (200):**
```json
{
  "id": 1,
  "status": "COMPLETED",
  "completedAt": "2026-02-15T14:30:00Z",
  "updatedAt": "2026-02-15T14:30:00Z"
}
```

---

### 47. Cancel Appointment
**PATCH** `/appointments/:appointmentId/cancel`

**Description:** Cancel an appointment

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "cancellationReason": "Student is sick"
}
```

**Response (200):**
```json
{
  "id": 1,
  "status": "CANCELLED",
  "cancelledAt": "2026-02-11T10:30:00Z",
  "cancellationReason": "Student is sick",
  "updatedAt": "2026-02-11T10:30:00Z"
}
```

---

### 48. Mark as No-Show
**PATCH** `/appointments/:appointmentId/no-show`

**Description:** Mark appointment as no-show (student didn't attend)

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "status": "NO_SHOW",
  "updatedAt": "2026-02-15T14:30:00Z"
}
```

---

## Doctor Status & Waiting Queue Endpoints

### 49. Get Doctor Current Status
**GET** `/doctors/:doctorId/status`

**Description:** Get doctor's current status and waiting queue information

**Response (200):**
```json
{
  "doctorId": 5,
  "fullName": "Dr. James Smith",
  "currentStatus": "ATTENDING",
  "currentPatientId": 1,
  "estimatedWaitTime": 45,
  "waitingQueueCount": 3,
  "updatedAt": "2026-02-15T14:00:00Z"
}
```

**Status Values:**
- `FREE` - Doctor is available
- `ATTENDING` - Doctor is with a patient
- `ON_BREAK` - Doctor is on break
- `OFFLINE` - Doctor is offline

---

### 50. Update Doctor Status
**PATCH** `/doctors/:doctorId/status`

**Description:** Update doctor's current status (use by doctor only)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "status": "ATTENDING",
  "currentPatientId": 1,
  "estimatedWaitTime": 45
}
```

**Response (200):**
```json
{
  "doctorId": 5,
  "currentStatus": "ATTENDING",
  "currentPatientId": 1,
  "estimatedWaitTime": 45,
  "updatedAt": "2026-02-15T14:00:00Z"
}
```

---

### 51. Get Doctor Waiting Queue
**GET** `/doctors/:doctorId/queue`

**Description:** Get all patients waiting for a specific doctor

**Query Parameters:**
- `limit` (optional): Results per page (default: 20)
- `includeServed` (optional): Include already served patients (default: false)

**Response (200):**
```json
{
  "doctorId": 5,
  "totalWaiting": 3,
  "estimatedWaitTime": 45,
  "queue": [
    {
      "queuePosition": 1,
      "appointmentId": 2,
      "studentId": 3,
      "studentName": "Sarah Johnson",
      "type": "EMERGENCY",
      "priority": 2,
      "arrivedAt": "2026-02-15T13:45:00Z",
      "estimatedWaitTime": 10
    },
    {
      "queuePosition": 2,
      "appointmentId": 3,
      "studentId": 4,
      "studentName": "Mike Wilson",
      "type": "NORMAL_SICKNESS",
      "priority": 1,
      "arrivedAt": "2026-02-15T13:50:00Z",
      "estimatedWaitTime": 40
    },
    {
      "queuePosition": 3,
      "appointmentId": 4,
      "studentId": 5,
      "studentName": "Lisa Davis",
      "type": "BOOKING",
      "priority": 0,
      "arrivedAt": "2026-02-15T14:00:00Z",
      "estimatedWaitTime": 70
    }
  ]
}
```

---

### 52. Add Patient to Waiting Queue
**POST** `/doctors/:doctorId/queue`

**Description:** Add a patient to the waiting queue (called when student arrives)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "appointmentId": 2,
  "studentId": 3,
  "type": "EMERGENCY",
  "priority": 2
}
```

**Response (201):**
```json
{
  "queueId": 1,
  "queuePosition": 1,
  "doctorId": 5,
  "appointmentId": 2,
  "studentId": 3,
  "type": "EMERGENCY",
  "priority": 2,
  "estimatedWaitTime": 10,
  "status": "WAITING",
  "arrivedAt": "2026-02-15T13:45:00Z"
}
```

---

### 53. Get Appointment Queue Position
**GET** `/appointments/:appointmentId/queue-position`

**Description:** Get student's current position in the waiting queue

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "appointmentId": 3,
  "studentId": 4,
  "doctorId": 5,
  "queuePosition": 2,
  "type": "NORMAL_SICKNESS",
  "priority": 1,
  "estimatedWaitTime": 40,
  "totalWaitingAhead": 1,
  "arrivedAt": "2026-02-15T13:50:00Z"
}
```

---

### 54. Mark Patient as Being Served
**PATCH** `/doctors/:doctorId/queue/:queueId/serve`

**Description:** Mark patient as being served (doctor calls next patient)

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "queueId": 1,
  "appointmentId": 2,
  "studentId": 3,
  "doctorId": 5,
  "status": "SERVING",
  "servedAt": "2026-02-15T13:55:00Z",
  "updatedAt": "2026-02-15T13:55:00Z"
}
```

---

### 55. Get Queue Statistics
**GET** `/doctors/:doctorId/queue-stats`

**Description:** Get queue statistics for a doctor

**Response (200):**
```json
{
  "doctorId": 5,
  "currentDate": "2026-02-15",
  "totalSeenToday": 12,
  "totalWaiting": 3,
  "averageWaitTime": 35,
  "longestWait": 60,
  "appointmentsByType": {
    "EMERGENCY": 1,
    "NORMAL_SICKNESS": 1,
    "BOOKING": 1
  },
  "appointmentsByPriority": {
    "EMERGENCY": 1,
    "URGENT": 1,
    "NORMAL": 1
  }
}
```

---

## Admin Endpoints

### 56. Get All Appointments
**GET** `/admin/appointments`

**Description:** Retrieve all appointments in the system

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `status` (optional): Filter by status
- `type` (optional): Filter by appointment type (EMERGENCY, BOOKING, NORMAL_SICKNESS)
- `studentId` (optional): Filter by student
- `doctorId` (optional): Filter by doctor
- `startDate` (optional): Filter from date (YYYY-MM-DD)
- `endDate` (optional): Filter to date (YYYY-MM-DD)
- `page` (optional): Page number
- `limit` (optional): Results per page

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "studentId": 1,
      "doctorId": 5,
      "scheduledAt": "2026-02-15T14:00:00Z",
      "type": "BOOKING",
      "priority": 0,
      "status": "BOOKED",
      "student": { "fullName": "John Doe" },
      "doctor": { "fullName": "Dr. James Smith" }
    }
  ],
  "total": 142,
  "page": 1,
  "limit": 20
}
```

---

### 57. Get Dashboard Statistics
**GET** `/admin/statistics`

**Description:** Get system-wide statistics

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `startDate` (optional): From date (YYYY-MM-DD)
- `endDate` (optional): To date (YYYY-MM-DD)

**Response (200):**
```json
{
  "totalAppointments": 250,
  "bookedAppointments": 45,
  "completedAppointments": 180,
  "cancelledAppointments": 20,
  "noShowAppointments": 5,
  "totalStudents": 500,
  "totalDoctors": 15,
  "appointmentsByStatus": {
    "BOOKED": 45,
    "COMPLETED": 180,
    "CANCELLED": 20,
    "NO_SHOW": 5,
    "CHECKED_IN": 10,
    "WAITING": 3
  },
  "appointmentsByType": {
    "EMERGENCY": 20,
    "NORMAL_SICKNESS": 80,
    "BOOKING": 150
  },
  "appointmentsBySpecialization": {
    "General Medicine": 80,
    "Cardiology": 45,
    "Dermatology": 35
  },
  "doctorStatuses": {
    "FREE": 8,
    "ATTENDING": 5,
    "ON_BREAK": 1,
    "OFFLINE": 1
  },
  "averageRating": 4.6
}
```

---

### 58. Get All Users
**GET** `/admin/users`

**Description:** Retrieve all users with filters

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `role` (optional): Filter by role (STUDENT, DOCTOR, STAFF, ADMIN)
- `page` (optional): Page number
- `limit` (optional): Results per page

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "role": "STUDENT",
      "createdAt": "2026-01-15T10:00:00Z"
    }
  ],
  "total": 520,
  "page": 1,
  "limit": 20
}
```

---

### 59. Delete User
**DELETE** `/admin/users/:userId`

**Description:** Remove a user from the system

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "message": "User deleted successfully"
}
```

---

### 60. Get User Activity Log
**GET** `/admin/users/:userId/activity`

**Description:** Get activity history for a specific user

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "userId": 1,
  "activities": [
    {
      "action": "appointment_booked",
      "appointmentId": 1,
      "type": "BOOKING",
      "timestamp": "2026-02-11T10:00:00Z"
    },
    {
      "action": "appointment_completed",
      "appointmentId": 1,
      "timestamp": "2026-02-15T14:30:00Z"
    }
  ]
}
```

---

## Error Responses

### 400 Bad Request
```json
{
  "error": "Bad Request",
  "message": "Invalid input data",
  "details": {
    "field": "email",
    "issue": "Invalid email format"
  }
}
```

### 401 Unauthorized
```json
{
  "error": "Unauthorized",
  "message": "Authentication token is missing or invalid"
}
```

### 403 Forbidden
```json
{
  "error": "Forbidden",
  "message": "You don't have permission to access this resource"
}
```

### 404 Not Found
```json
{
  "error": "Not Found",
  "message": "Resource not found"
}
```

### 409 Conflict
```json
{
  "error": "Conflict",
  "message": "Time slot is already booked",
  "details": {
    "reason": "Another appointment exists at this time"
  }
}
```

### 500 Internal Server Error
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred"
}
```

---

## Rate Limiting

All endpoints implement rate limiting:
- **Standard users**: 100 requests per hour
- **Admin users**: 500 requests per hour

**Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1613067600
```

---

## Summary Table

| # | Endpoint | Method | Purpose |
|---|----------|--------|---------|
| 1 | `/auth/register` | POST | User registration |
| 2 | `/auth/login` | POST | User authentication |
| 3 | `/auth/logout` | POST | User logout |
| 4 | `/students` | POST | Create student profile |
| 5 | `/students/:id` | GET | Get student profile |
| 6 | `/students/:id` | PUT | Update student profile |
| 7 | `/students/:id/appointments` | GET | Get student appointments |
| 8 | `/doctors` | GET | List all doctors |
| 9 | `/doctors/:id` | GET | Get doctor details |
| 10 | `/doctors/:id/availability` | POST | Add availability |
| 11 | `/doctors/:id/availability` | GET | Get availability |
| 12 | `/doctors/:id/availability/:availId` | PUT | Update availability |
| 13 | `/doctors/:id/availability/:availId` | DELETE | Delete availability |
| 14 | `/doctors/:id/appointments` | GET | Get doctor appointments |
| 15 | `/departments` | GET | Get all departments |
| 16 | `/departments` | POST | Create department |
| 17 | `/students/:id/medical-record` | GET | Get medical records |
| 18 | `/students/:id/medical-record` | POST | Create/update medical record |
| 19 | `/appointments/:id/consultation-note` | POST | Add consultation note |
| 20 | `/appointments/:id/consultation-note` | GET | Get consultation note |
| 21 | `/appointments/:id/prescriptions` | POST | Add prescription |
| 22 | `/students/:id/prescriptions` | GET | Get prescriptions |
| 23 | `/appointments/:id/review` | POST | Add review |
| 24 | `/doctors/:id/reviews` | GET | Get doctor reviews |
| 25 | `/students/:id/reviews` | GET | Get student reviews |
| 26 | `/users/:id/notifications` | GET | Get notifications |
| 27 | `/users/:id/notification-preferences` | GET | Get notification preferences |
| 28 | `/users/:id/notification-preferences` | PUT | Update notification preferences |
| 29 | `/notifications/:id/read` | PATCH | Mark notification as read |
| 30 | `/recurring-appointments` | POST | Create recurring appointment |
| 31 | `/students/:id/recurring-appointments` | GET | Get recurring appointments |
| 32 | `/recurring-appointments/:id` | PUT | Update recurring appointment |
| 33 | `/recurring-appointments/:id` | DELETE | Cancel recurring appointment |
| 34 | `/departments/:id/holidays` | GET | Get department holidays |
| 35 | `/departments/:id/holidays` | POST | Add department holiday |
| 36 | `/attachments/upload` | POST | Upload attachment |
| 37 | `/appointments/:id/attachments` | GET | Get appointment attachments |
| 38 | `/attachments/:id` | DELETE | Delete attachment |
| 39 | `/admin/audit-logs` | GET | Get audit logs |
| 40 | `/admin/users/:id/activity-log` | GET | Get user activity log |
| 41 | `/appointments/available-slots` | GET | Get available slots |
| 42 | `/appointments` | POST | Book appointment |
| 43 | `/appointments/:id` | GET | Get appointment details |
| 44 | `/appointments/:id` | PUT | Update appointment |
| 45 | `/appointments/:id/check-in` | PATCH | Check in patient |
| 46 | `/appointments/:id/complete` | PATCH | Complete appointment |
| 47 | `/appointments/:id/cancel` | PATCH | Cancel appointment |
| 48 | `/appointments/:id/no-show` | PATCH | Mark as no-show |
| 49 | `/doctors/:id/status` | GET | Get doctor current status |
| 50 | `/doctors/:id/status` | PATCH | Update doctor status |
| 51 | `/doctors/:id/queue` | GET | Get doctor waiting queue |
| 52 | `/doctors/:id/queue` | POST | Add patient to queue |
| 53 | `/appointments/:id/queue-position` | GET | Get queue position |
| 54 | `/doctors/:id/queue/:queueId/serve` | PATCH | Mark patient being served |
| 55 | `/doctors/:id/queue-stats` | GET | Get queue statistics |
| 56 | `/admin/appointments` | GET | Get all appointments |
| 57 | `/admin/statistics` | GET | Get dashboard stats |
| 58 | `/admin/users` | GET | Get all users |
| 59 | `/admin/users/:id` | DELETE | Delete user |
| 60 | `/admin/users/:id/activity` | GET | Get user activity |

**Total Endpoints: 60**

---
