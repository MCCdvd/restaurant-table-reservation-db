# 4. Key-Value Database Structure Summary

## Storage Architecture

The key-value database uses a multi-index design pattern to optimize different access patterns:

### Core Storage Keys

| Key Pattern | Value Type | Purpose | Example |
|------------|------------|---------|----------|
| `reservation:{id}` | Reservation Object | Primary reservation storage | `reservation:UUID-001` |
| `availability:{date}:table:{id}` | Reservation ID or null | Prevent double-booking | `availability:2024-10-14:table:1` |
| `customer:{name}:{date}` | Reservation ID | Quick customer lookup | `customer:john_doe:2024-10-14` |
| `date:{date}:reservations` | List[Reservation ID] | All reservations for a date | `date:2024-10-14:reservations` |
| `table:{id}` | Table Object | Table configuration | `table:1` |
| `table:{id}:status:{date}` | TableStatus Enum | Table status by date | `table:1:status:2024-10-14` |
| `qrcode:mapping:{qr_code}` | Reservation ID | QR code to reservation mapping | `qrcode:mapping:QR00A1B2` |
| `logs:access` | List[Log Entry] | Audit trail for access | `logs:access` |
| `logs:cancellation` | List[Log Entry] | Cancellation history | `logs:cancellation` |

## Data Models

### Reservation Object
```json
{
  "reservation_id": "UUID-001",
  "table_id": "table:1",
  "date": "2024-10-14",
  "customer_name": "John Doe",
  "diners": 4,
  "time": "19:00",
  "status": "CONFIRMED",
  "qr_code": "QR00A1B2",
  "created_at": "2024-10-14T10:30:00Z",
  "cancelled_at": null,
  "special_requests": ""
}
```

### Table Object
```json
{
  "table_id": "table:1",
  "capacity": 4,
  "location": "Window",
  "section": "Main Dining",
  "created_at": "2024-01-01T00:00:00Z"
}
```

### Log Entry Object
```json
{
  "reservation_id": "UUID-001",
  "action": "CREATED",
  "timestamp": "2024-10-14T10:30:00Z",
  "ip_address": "192.168.1.1",
  "user_agent": "Mozilla/5.0..."
}
```

## Access Patterns & Query Examples

### Pattern 1: Get Reservation by QR Code
```
1. Decode QR Code → Get reservation_id from qrcode:mapping:{qr_code}
2. Lookup → keyValueStore.GET("reservation:" + reservation_id)
3. Complexity: O(1)
```

### Pattern 2: Find Available Tables on Specific Date
```
1. Loop through all table IDs (1-5)
2. Check "availability:{date}:table:{id}" for each table
3. If value is null, table is available
4. Complexity: O(n) where n = number of tables (5)
```

### Pattern 3: Get All Reservations for a Customer
```
1. Normalize customer name: john_doe
2. For each date, lookup "customer:john_doe:{date}"
3. Get reservation IDs from results
4. Complexity: O(d) where d = days to search
```

### Pattern 4: View All Reservations for a Date
```
1. Lookup "date:{date}:reservations"
2. Returns list of all reservation IDs for that date
3. Complexity: O(1) lookup, O(r) to process results
```

## Scalability Analysis

### Storage Size Estimation

#### For 10,000 Daily Reservations
```
Per Reservation:
  - Main object (reservation:id): ~0.5 KB
  - Availability index entry: ~0.1 KB
  - Customer index entry: ~0.1 KB
  - QR mapping entry: ~0.1 KB
  - Total per reservation: ~0.8 KB

Total for 10,000 reservations:
  10,000 × 0.8 KB = 8 MB per day
  8 MB × 365 days = 2.92 GB per year
```

#### For 1,000,000 Annual Reservations
```
1,000,000 × 0.8 KB = 800 MB
With indices and logs: ~1 GB
Compressed: ~200-300 MB
```

### Performance Metrics

| Operation | Avg Latency | Max Latency | Throughput |
|-----------|------------|------------|------------|
| SetReservation | 10ms | 50ms | 1000+ ops/sec |
| GetReservation | 5ms | 20ms | 10000+ ops/sec |
| DeleteReservation | 10ms | 50ms | 1000+ ops/sec |
| FindAvailableTable | 2ms | 10ms | 5000+ ops/sec |

## Consistency & Durability

### Transaction Flow for SetReservation
```
1. Generate reservation ID and QR code
2. Create reservation object
3. BEGIN TRANSACTION
   - INSERT reservation:{id}
   - SET availability:{date}:table:{id}
   - SET customer:{name}:{date}
   - APPEND date:{date}:reservations
   - SET table:{id}:status:{date}
   - APPEND logs:access
4. COMMIT
5. Return QR code
```

### Idempotency
- QR codes are unique (UUID-based)
- Same reservation cannot be created twice
- If network fails, client retries with same data
- System detects existing reservation and returns same QR code

## Backup & Recovery

### Point-in-Time Recovery
- All modifications logged with timestamps
- Access logs enable audit trails
- Cancellation logs allow restoration

### Replication Strategy
```
Primary Database
      |
      ├─→ Read Replica 1 (async)
      ├─→ Read Replica 2 (async)
      └─→ Archive (daily snapshots)
```

## Optimization Techniques

### 1. Caching Strategy
```
L1 Cache: QR Code → Reservation (1 hour TTL)
L2 Cache: Table availability (5 min TTL)
L3 Cache: Table configuration (24 hour TTL)
```

### 2. Compression
- Use binary encoding for frequently accessed data
- Compress historical logs after 30 days
- Archive completed reservations annually

### 3. Sharding
```
Shard by Date:
  Shard 1: 2024-01-01 to 2024-01-31
  Shard 2: 2024-02-01 to 2024-02-28
  ...
Benefit: Faster access, easier archival
```

## Monitoring & Alerts

### Key Metrics
- Average response time (target: <10ms)
- Error rate (target: <0.1%)
- Database size growth
- QR code generation success rate
- Reservation cancellation rate

### Sample Alerts
```
IF avg_response_time > 50ms THEN
  Alert("Database latency high")
END IF

IF error_rate > 1% THEN
  Alert("High error rate detected")
END IF

IF disk_usage > 80% THEN
  Alert("Storage capacity low")
END IF
```
