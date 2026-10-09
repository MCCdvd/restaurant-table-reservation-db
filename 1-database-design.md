# 1. Key-Value Pair Examples & Database Design

## Restaurant Scenario
**Date**: October 14th, 2024

### Restaurant Tables Configuration
```json
{
  "table:1": {
    "table_id": "table:1",
    "capacity": 4,
    "location": "Window"
  },
  "table:2": {
    "table_id": "table:2",
    "capacity": 2,
    "location": "Bar"
  },
  "table:3": {
    "table_id": "table:3",
    "capacity": 6,
    "location": "Main Hall"
  },
  "table:4": {
    "table_id": "table:4",
    "capacity": 4,
    "location": "Corner"
  },
  "table:5": {
    "table_id": "table:5",
    "capacity": 8,
    "location": "Patio"
  }
}
```

## Reservations on October 14th, 2024

### Reservation 1: John Doe
```json
"reservation:UUID-001": {
  "reservation_id": "UUID-001",
  "table_id": "table:1",
  "date": "2024-10-14",
  "customer_name": "John Doe",
  "diners": 4,
  "time": "19:00",
  "status": "CONFIRMED",
  "qr_code": "QR00A1B2",
  "created_at": "2024-10-14T10:30:00Z",
  "special_requests": ""
}
```

### Reservation 2: Jane Smith
```json
"reservation:UUID-002": {
  "reservation_id": "UUID-002",
  "table_id": "table:3",
  "date": "2024-10-14",
  "customer_name": "Jane Smith",
  "diners": 6,
  "time": "19:30",
  "status": "CONFIRMED",
  "qr_code": "QR00C3D4",
  "created_at": "2024-10-14T11:15:00Z",
  "special_requests": "Vegetarian menu required"
}
```

## Availability Index (Date-Table Combinations)

### Available Tables (No Reservations)
```json
"availability:2024-10-14:table:1": "UUID-001",
"availability:2024-10-14:table:2": null,
"availability:2024-10-14:table:3": "UUID-002",
"availability:2024-10-14:table:4": null,
"availability:2024-10-14:table:5": null
```

## Customer Index (Quick Lookup by Name)

```json
"customer:john_doe:2024-10-14": "UUID-001",
"customer:jane_smith:2024-10-14": "UUID-002"
```

## Table Status by Date

```json
"table:1:status:2024-10-14": "RESERVED",
"table:2:status:2024-10-14": "AVAILABLE",
"table:3:status:2024-10-14": "RESERVED",
"table:4:status:2024-10-14": "AVAILABLE",
"table:5:status:2024-10-14": "AVAILABLE"
```

## QR Code Mapping

```json
"qrcode:mapping:QR00A1B2": "UUID-001",
"qrcode:mapping:QR00C3D4": "UUID-002"
```

## Daily Reservations Index

```json
"date:2024-10-14:reservations": ["UUID-001", "UUID-002"]
```

## Database Size Analysis
- **Total Keys for October 14th**: ~20 keys
- **Storage per reservation**: ~0.5 KB
- **Storage per table**: ~0.2 KB
- **Scalability**: Easily handles 1M+ reservations
