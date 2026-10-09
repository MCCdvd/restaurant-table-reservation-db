# Restaurant Table Reservation System - Complete Solution
PROMPT: Design a Table Reservation DB for a Restaurant

• Design the database for a table reservation system in a restaurant using a key-value
DB (depict also the di UML Analysis Class Diagram) .

• The restaurant has a fixed number of tables, each with a specified seating capacity,
and for each date, only one reservation is allowed per table.

• Functional requirements:
• Set a reservation for a specific day, given the number of diners and the name of
who reserved the table. It returns a QR code.
• Get a reservation by scanning a QR code for retrieving the info of the reservation
• Delete a reservation by using a QR code1. Provide an example of key-value pairs, considering the following scenario:
- The restaurant has 5 tables:
- Table 1: 4 seats, Table 2: 2 seats, Table 3: 6 seats, Table 4: 4 seats, Table 5: 8 seats
- On October 14th, 2024, the following reservations are made:
Table 1 is reserved by John Doe for 4 people.
Table 3 is reserved by Jane Smith for 6 people.

2. Using pseudodoce, implement the following methods:
function SetReservation(date, diners, customer_name);
function GetReservation(qr_code);
function DeleteReservation(qr_code).
## Overview
A comprehensive key-value database design for a restaurant table reservation system with QR code support.

## Contents
- **Database Design**: Key-value pair examples and schema
- **UML Analysis Class Diagram**: Complete class relationships
- **Pseudocode Implementation**: Three main functions with helpers
- **Data Models**: Detailed structure and storage patterns

## Quick Links
- [Database Design](./1-database-design.md)
- [UML Diagram](./2-uml-class-diagram.md)
- [Pseudocode Implementation](./3-pseudocode-implementation.md)
- [Key-Value Store Structure](./4-kv-structure.md)
- [Usage Examples](./5-usage-examples.md)

## Key Features
✅ O(1) Lookup Time
✅ Prevents Double-Booking
✅ QR Code Generation & Validation
✅ Audit Trail & Logging
✅ Scalable & Efficient

## Restaurant Configuration
- **Table 1**: 4 seats (Window)
- **Table 2**: 2 seats (Bar)
- **Table 3**: 6 seats (Main Hall)
- **Table 4**: 4 seats (Corner)
- **Table 5**: 8 seats (Patio)
