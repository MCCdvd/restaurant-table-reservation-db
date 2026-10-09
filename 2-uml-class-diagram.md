# 2. UML Analysis Class Diagram

## Complete Class Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLASS DIAGRAM                                 │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│      Restaurant          │
├──────────────────────────┤
│ - restaurant_id: String  │
│ - name: String           │
│ - address: String        │
│ - phone: String          │
│ - email: String          │
└──────────────────────────┘
         │
         │ 1..*
         │ contains
         │
┌──────────────────────────┐         ┌──────────────────────────┐
│       Table              │────1────│   Reservation            │
├──────────────────────────┤  hosts  ├──────────────────────────┤
│ - table_id: String       │         │ - reservation_id: String │
│ - capacity: Integer      │         │ - qr_code: String        │
│ - location: String       │         │ - customer_name: String  │
│ - status: TableStatus    │         │ - date: Date             │
│ + getAvailability()      │         │ - time: Time             │
│ + updateStatus()         │         │ - diners: Integer        │
└──────────────────────────┘         │ - status: ReservStatus   │
         ▲                           │ - created_at: DateTime   │
         │                           │ - cancelled_at: DateTime │
         │                           │ + validate()             │
         │                           │ + cancel()               │
         │                           └──────────────────────────┘
         │                                    │
         │                                    │ makes
         │                                    │
         │                           ┌────────▼──────────────┐
         │                           │      Customer         │
         │                           ├───────────────────────┤
         │                           │ - customer_id: String │
         │                           │ - name: String        │
         │                           │ - phone: String       │
         │                           │ - email: String       │
         │                           │ + getReservations()   │
         │                           └───────────────────────┘
         │
         │                           ┌───────────────────────┐
         │                           │      QRCode           │
         │                           ├───────────────────────┤
         │ 1                         │ - qr_id: String       │
         │ ◇───generates─────────────│ - code_value: String  │
         │                           │ - generated_at: Time  │
         │                           │ - expires_at: Time    │
         │                           │ + validate()          │
         │                           │ + decode()            │
         │                           └───────────────────────┘
         │
         │
         ▼
┌──────────────────────────────┐
│   ReservationManager         │
├──────────────────────────────┤
│ - kvStore: KeyValueDB        │
│ - qrGenerator: QRGenerator   │
├──────────────────────────────┤
│ + setReservation()           │
│ + getReservation()           │
│ + deleteReservation()        │
│ + findAvailableTable()       │
│ + validateReservation()      │
│ + generateQRCode()           │
│ + logActivity()              │
└──────────────────────────────┘
```

## Class Descriptions

### Restaurant
- **Purpose**: Represents the restaurant entity
- **Attributes**:
  - `restaurant_id`: Unique identifier
  - `name`: Restaurant name
  - `address`: Physical location
  - `phone`: Contact number
  - `email`: Contact email
- **Relationships**: Contains multiple tables (1..*)

### Table
- **Purpose**: Represents a dining table
- **Attributes**:
  - `table_id`: Unique table identifier
  - `capacity`: Number of seats
  - `location`: Physical location in restaurant
  - `status`: Current status (AVAILABLE, RESERVED, MAINTENANCE)
- **Methods**:
  - `getAvailability()`: Check if table is free
  - `updateStatus()`: Change table status
- **Relationships**: Hosts one reservation per date

### Reservation
- **Purpose**: Represents a table reservation
- **Attributes**:
  - `reservation_id`: Unique identifier
  - `qr_code`: QR code for check-in
  - `customer_name`: Name of the person who made the reservation
  - `date`: Reservation date
  - `time`: Reservation time
  - `diners`: Number of people
  - `status`: PENDING, CONFIRMED, CANCELLED, COMPLETED
  - `created_at`: Creation timestamp
  - `cancelled_at`: Cancellation timestamp (if applicable)
- **Methods**:
  - `validate()`: Check reservation validity
  - `cancel()`: Cancel the reservation
- **Relationships**: Made by one customer, uses one table

### Customer
- **Purpose**: Represents a restaurant customer
- **Attributes**:
  - `customer_id`: Unique identifier
  - `name`: Customer name
  - `phone`: Phone number
  - `email`: Email address
- **Methods**:
  - `getReservations()`: Retrieve all reservations by this customer

### QRCode
- **Purpose**: Represents a QR code for reservation
- **Attributes**:
  - `qr_id`: Unique QR code identifier
  - `code_value`: Encoded QR code value
  - `generated_at`: Creation timestamp
  - `expires_at`: Expiration timestamp
- **Methods**:
  - `validate()`: Check if QR code is valid
  - `decode()`: Extract reservation ID from QR code

### ReservationManager
- **Purpose**: Main service managing all reservation operations
- **Attributes**:
  - `kvStore`: Key-value database instance
  - `qrGenerator`: QR code generator service
- **Methods**:
  - `setReservation()`: Create new reservation
  - `getReservation()`: Retrieve reservation by QR code
  - `deleteReservation()`: Cancel reservation
  - `findAvailableTable()`: Find suitable table for diners
  - `validateReservation()`: Validate reservation data
  - `generateQRCode()`: Create QR code
  - `logActivity()`: Log all operations for audit

## Enumerations

### TableStatus
```
enum TableStatus {
  AVAILABLE,      // Table is free and can be reserved
  RESERVED,       // Table has an active reservation
  MAINTENANCE,    // Table is under maintenance
  DIRTY           // Table needs cleaning
}
```

### ReservationStatus
```
enum ReservationStatus {
  PENDING,        // Reservation created but not confirmed
  CONFIRMED,      // Reservation is confirmed
  CHECKED_IN,     // Customer has checked in
  COMPLETED,      // Reservation completed
  CANCELLED,      // Reservation was cancelled
  NO_SHOW         // Customer did not show up
}
```

## Relationship Summary

| From | To | Type | Cardinality | Description |
|------|-----|------|-------------|-------------|
| Restaurant | Table | Contains | 1..* | A restaurant has multiple tables |
| Table | Reservation | Hosts | 1..1 | Each table can host one reservation per date |
| Reservation | Customer | Made By | 1..1 | Each reservation is made by one customer |
| Reservation | QRCode | Generates | 1..1 | Each reservation generates one QR code |
| ReservationManager | All | Manages | - | Central service manages all operations |
