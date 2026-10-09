# 3. Pseudocode Implementation

## ReservationSystem Class

### FUNCTION 1: SET RESERVATION

```pseudocode
FUNCTION SetReservation(date: Date, diners: Integer, customer_name: String) 
    RETURNS QRCode
BEGIN
    // Step 1: Find an available table that matches capacity
    availableTable = FIND_AVAILABLE_TABLE(date, diners)
    
    IF availableTable IS NULL THEN
        THROW Exception("No available table for specified date and capacity")
    END IF
    
    // Step 2: Generate unique reservation ID and QR code
    reservation_id = GENERATE_UUID()
    qr_code_value = GENERATE_QR_CODE(reservation_id)
    
    // Step 3: Create reservation object
    reservation = {
        reservation_id: reservation_id,
        table_id: availableTable.table_id,
        date: date,
        customer_name: customer_name,
        diners: diners,
        time: GET_DEFAULT_TIME(date),
        status: "CONFIRMED",
        qr_code: qr_code_value,
        created_at: NOW(),
        special_requests: ""
    }
    
    // Step 4: Store reservation in key-value database
    // Primary storage
    keyValueStore.SET("reservation:" + reservation_id, reservation)
    
    // Step 5: Update availability index
    // This prevents double-booking on the same date/table
    keyValueStore.SET("availability:" + date + ":table:" + availableTable.table_id, 
                     reservation_id)
    
    // Step 6: Create customer index for quick lookup
    customer_key = "customer:" + NORMALIZE_NAME(customer_name) + ":" + date
    keyValueStore.SET(customer_key, reservation_id)
    
    // Step 7: Create date-based index for restaurant view
    date_index_key = "date:" + date + ":reservations"
    APPEND_TO_LIST(keyValueStore, date_index_key, reservation_id)
    
    // Step 8: Update table status
    keyValueStore.SET("table:" + availableTable.table_id + ":status:" + date, 
                     "RESERVED")
    
    // Step 9: Log the action
    LOG_RESERVATION_ACCESS(reservation_id, "CREATED", NOW())
    
    // Step 10: Return QR code to customer
    PRINT "QR Code Generated: " + qr_code_value
    RETURN qr_code_value
    
END FUNCTION
```

**Time Complexity**: O(1) - Direct key lookups
**Space Complexity**: O(1) - Fixed size reservation object

---

### FUNCTION 2: GET RESERVATION

```pseudocode
FUNCTION GetReservation(qr_code: String) 
    RETURNS Reservation
BEGIN
    // Step 1: Validate QR code format
    IF NOT VALIDATE_QR_CODE_FORMAT(qr_code) THEN
        THROW Exception("Invalid QR code format")
    END IF
    
    // Step 2: Extract reservation ID from QR code
    // The QR code contains encoded reservation ID
    reservation_id = DECODE_QR_CODE(qr_code)
    
    // Step 3: Query the key-value store
    lookup_key = "reservation:" + reservation_id
    reservation = keyValueStore.GET(lookup_key)
    
    // Step 4: Verify reservation exists
    IF reservation IS NULL THEN
        THROW Exception("Reservation not found with provided QR code")
    END IF
    
    // Step 5: Check if reservation is still valid (not expired/cancelled)
    IF reservation.status == "CANCELLED" THEN
        THROW Exception("Reservation has been cancelled")
    END IF
    
    IF reservation.date < TODAY() THEN
        THROW Exception("Reservation date has passed")
    END IF
    
    // Step 6: Retrieve associated table details
    table = keyValueStore.GET("table:" + reservation.table_id)
    
    // Step 7: Build response with complete information
    response = {
        reservation_id: reservation.reservation_id,
        customer_name: reservation.customer_name,
        table_id: reservation.table_id,
        table_capacity: table.capacity,
        table_location: table.location,
        date: reservation.date,
        time: reservation.time,
        diners: reservation.diners,
        status: reservation.status,
        created_at: reservation.created_at,
        special_requests: reservation.special_requests
    }
    
    // Step 8: Log the query for audit purposes
    LOG_RESERVATION_ACCESS(reservation_id, "ACCESSED", NOW())
    
    // Step 9: Return reservation details
    PRINT "Reservation found for " + response.customer_name
    RETURN response
    
END FUNCTION
```

**Time Complexity**: O(1) - Direct key lookup
**Space Complexity**: O(1) - Fixed response size

---

### FUNCTION 3: DELETE RESERVATION

```pseudocode
FUNCTION DeleteReservation(qr_code: String) 
    RETURNS Boolean
BEGIN
    // Step 1: Validate QR code
    IF NOT VALIDATE_QR_CODE_FORMAT(qr_code) THEN
        THROW Exception("Invalid QR code format")
    END IF
    
    // Step 2: Extract and retrieve reservation
    reservation_id = DECODE_QR_CODE(qr_code)
    lookup_key = "reservation:" + reservation_id
    reservation = keyValueStore.GET(lookup_key)
    
    // Step 3: Verify reservation exists
    IF reservation IS NULL THEN
        THROW Exception("Reservation not found")
    END IF
    
    // Step 4: Prevent deletion of past reservations
    IF reservation.date < TODAY() THEN
        THROW Exception("Cannot delete past reservations")
    END IF
    
    // Step 5: Mark reservation as cancelled instead of hard delete
    // This preserves audit trail
    reservation.status = "CANCELLED"
    reservation.cancelled_at = NOW()
    keyValueStore.SET(lookup_key, reservation)
    
    // Step 6: Free up the table
    availability_key = "availability:" + reservation.date + ":table:" + 
                      reservation.table_id
    keyValueStore.DELETE(availability_key)
    
    // Step 7: Update table status
    table_status_key = "table:" + reservation.table_id + ":status:" + 
                      reservation.date
    keyValueStore.SET(table_status_key, "AVAILABLE")
    
    // Step 8: Remove from customer index
    customer_key = "customer:" + NORMALIZE_NAME(reservation.customer_name) + 
                  ":" + reservation.date
    keyValueStore.DELETE(customer_key)
    
    // Step 9: Log cancellation
    LOG_RESERVATION_CANCELLATION(reservation_id, NOW(), "QR Code Scan")
    
    // Step 10: Return success
    PRINT "Reservation " + reservation_id + " has been cancelled"
    RETURN TRUE
    
END FUNCTION
```

**Time Complexity**: O(1) - Direct key operations
**Space Complexity**: O(1) - No additional data structures

---

## Helper Functions

### VALIDATE_QR_CODE_FORMAT
```pseudocode
FUNCTION VALIDATE_QR_CODE_FORMAT(qr_code: String) RETURNS Boolean
BEGIN
    // Check if QR code matches expected format: QR + 8 hex characters
    RETURN qr_code MATCHES PATTERN "^QR[0-9A-F]{8}$"
END FUNCTION
```

### DECODE_QR_CODE
```pseudocode
FUNCTION DECODE_QR_CODE(qr_code: String) RETURNS String
BEGIN
    // Extract reservation ID from QR code mapping
    mapping_key = "qrcode:mapping:" + qr_code
    reservation_id = keyValueStore.GET(mapping_key)
    
    IF reservation_id IS NULL THEN
        THROW Exception("QR code mapping not found")
    END IF
    
    RETURN reservation_id
END FUNCTION
```

### FIND_AVAILABLE_TABLE
```pseudocode
FUNCTION FIND_AVAILABLE_TABLE(date: Date, diners: Integer) 
    RETURNS Table
BEGIN
    // Retrieve all tables from restaurant configuration
    allTables = GET_ALL_TABLES()
    
    // Sort tables by capacity (best fit algorithm)
    SORT allTables BY capacity ASCENDING
    
    FOR EACH table IN allTables DO
        // Check capacity matches
        IF table.capacity >= diners THEN
            // Check if table is already reserved on this date
            availability_key = "availability:" + date + ":table:" + table.table_id
            existing_reservation = keyValueStore.GET(availability_key)
            
            // If no reservation exists, table is available
            IF existing_reservation IS NULL THEN
                RETURN table
            END IF
        END IF
    END FOR
    
    // No available table found
    RETURN NULL
    
END FUNCTION
```

### NORMALIZE_NAME
```pseudocode
FUNCTION NORMALIZE_NAME(name: String) RETURNS String
BEGIN
    // Convert name to lowercase and replace spaces with underscores
    normalized = name.LOWER_CASE().REPLACE(" ", "_")
    RETURN normalized
END FUNCTION
```

### GENERATE_UUID
```pseudocode
FUNCTION GENERATE_UUID() RETURNS String
BEGIN
    // Generate universally unique identifier using UUID v4
    return UUID_V4().TO_STRING()
END FUNCTION
```

### GENERATE_QR_CODE
```pseudocode
FUNCTION GENERATE_QR_CODE(reservation_id: String) RETURNS String
BEGIN
    // Generate QR code value from reservation ID
    // Format: QR + first 8 characters of hex-encoded reservation ID
    hex_part = HEX_ENCODE(reservation_id.SUBSTRING(0, 8))
    qr_value = "QR" + hex_part.SUBSTRING(0, 8)
    
    // Store bidirectional mapping for retrieval
    keyValueStore.SET("qrcode:mapping:" + qr_value, reservation_id)
    keyValueStore.SET("reservation:" + reservation_id + ":qr", qr_value)
    
    RETURN qr_value
END FUNCTION
```

### LOG_RESERVATION_ACCESS
```pseudocode
FUNCTION LOG_RESERVATION_ACCESS(reservation_id: String, action: String, timestamp: DateTime)
BEGIN
    log_entry = {
        reservation_id: reservation_id,
        action: action,
        timestamp: timestamp,
        ip_address: GET_CLIENT_IP()
    }
    keyValueStore.APPEND("logs:access", log_entry)
END FUNCTION
```

### LOG_RESERVATION_CANCELLATION
```pseudocode
FUNCTION LOG_RESERVATION_CANCELLATION(reservation_id: String, 
                                     timestamp: DateTime, method: String)
BEGIN
    log_entry = {
        reservation_id: reservation_id,
        action: "CANCELLED",
        method: method,
        timestamp: timestamp,
        ip_address: GET_CLIENT_IP()
    }
    keyValueStore.APPEND("logs:cancellation", log_entry)
END FUNCTION
```

### GET_ALL_TABLES
```pseudocode
FUNCTION GET_ALL_TABLES() RETURNS List<Table>
BEGIN
    tables = NEW List<Table>()
    
    // Retrieve all 5 tables from configuration
    FOR i = 1 TO 5 DO
        table = keyValueStore.GET("table:" + i)
        IF table IS NOT NULL THEN
            tables.ADD(table)
        END IF
    END FOR
    
    RETURN tables
END FUNCTION
```

### GET_DEFAULT_TIME
```pseudocode
FUNCTION GET_DEFAULT_TIME(date: Date) RETURNS Time
BEGIN
    // Return default reservation time (7:00 PM)
    RETURN Time(19, 00, 00)
END FUNCTION
```

---

## Algorithm Complexity Analysis

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| SetReservation | O(1) | O(1) | Direct key insertions, no loops |
| GetReservation | O(1) | O(1) | Direct key lookup |
| DeleteReservation | O(1) | O(1) | Direct key operations |
| FindAvailableTable | O(n) | O(1) | n = number of tables (5) |
| ValidateQRCode | O(1) | O(1) | Regex matching |
| DecodeQRCode | O(1) | O(1) | Direct mapping lookup |

**Overall Complexity**: Optimal for real-world restaurant operations
