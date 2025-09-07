# Timestamp and Timezone Standards

## Overview

This document outlines best practices for handling timestamps and timezones across different systems. **All storage must use ISO 8601 format in UTC**, while only the UI layer should localize timestamps to user timezones.

## 1. Standard Timestamp Format

### ISO 8601 Standard

The **ISO 8601** standard is the required format for all timestamp storage. It provides a clear, unambiguous, and sortable representation.

#### Format Structure
```
yyyy-MM-ddThh:mm:ssZ
```

Where:
- `yyyy` = Four-digit year
- `MM` = Two-digit month (01-12)
- `dd` = Two-digit day (01-31)
- `T` = Separator between date and time
- `hh` = Two-digit hour in 24-hour format (00-23)
- `mm` = Two-digit minute (00-59)
- `ss` = Two-digit second (00-59)
- `Z` = UTC timezone indicator

#### Examples
```
2024-12-19T15:30:45Z        # Basic format      (recommended)
2024-12-19T15:30:45.123Z    # With milliseconds (optional)
2024-12-19T15:30:45.123456Z # With microseconds (optional)
```

### Precision Beyond Seconds (Optional)

For systems requiring higher precision, fractional seconds can be added:

```
2024-12-19T15:30:45.123Z        # Milliseconds
2024-12-19T15:30:45.123456Z     # Microseconds
2024-12-19T15:30:45.123456789Z  # Nanoseconds
```

### Storage Requirements
1. **Always store in UTC**: Use the `Z` suffix
2. **Consistent format**: Use `yyyy-MM-ddThh:mm:ssZ` throughout your system
3. **Optional precision**: Add fractional seconds only when needed
4. **Validation**: Implement strict validation for incoming timestamps

## 2. Storage vs UI Layer Separation

### Storage Layer (Always UTC)
- **All timestamps stored in UTC** using ISO 8601 format
- **No timezone conversion** at storage level
- **Consistent across all systems** and databases

### UI Layer (Localization Only)
- **Convert UTC to user timezone** for display only
- **Never modify stored timestamps** based on user timezone
- **Handle DST transitions** in display logic only

## 3. Daylight Saving Time Considerations

### DST Challenges
1. **Spring forward**: 2:00 AM becomes 3:00 AM (lose 1 hour)
2. **Fall back**: 2:00 AM becomes 1:00 AM (gain 1 hour)
3. **Ambiguous times**: 1:30 AM occurs twice during fall transition
4. **Non-existent times**: 2:30 AM doesn't exist during spring transition

### DST Handling (UI Layer Only)
- **Storage remains in UTC** - no DST handling needed
- **UI layer handles DST** during display conversion
- **Use timezone-aware libraries** for accurate conversions

## 4. Implementation Code Examples

### Java
```java
// Generate UTC timestamp
import java.time.Instant;

// Current UTC timestamp
String utcTimestamp = Instant.now().toString(); // "2024-12-19T15:30:45.123Z"

// Parse UTC timestamp
Instant instant = Instant.parse("2024-12-19T15:30:45Z");
```

### .NET (C#)
```csharp
// Generate UTC timestamp
using System;

// Current UTC timestamp
string utcTimestamp = DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ");

// Parse UTC timestamp
DateTime utcDateTime = DateTime.Parse("2024-12-19T15:30:45Z").ToUniversalTime();
```

### Angular (Experience Layer)
```typescript
// Angular service for timezone handling
import { Injectable } from '@angular/core';
import { DatePipe } from '@angular/common';

@Injectable()
export class TimezoneService {
  constructor(private datePipe: DatePipe) {}

  // Convert UTC to user timezone for DISPLAY ONLY
  convertToUserTimezone(utcTimestamp: string, timezone: string): string {
    const date = new Date(utcTimestamp);
    return this.datePipe.transform(date, 'yyyy-MM-dd HH:mm:ss z', timezone) || '';
    // Note: Return value is only for display, never store it
  }

  // Get user's timezone
  getUserTimezone(): string {
    return Intl.DateTimeFormat().resolvedOptions().timeZone;
  }

  // Sample localized timezone conversions
  getLocalizedTime(utcTimestamp: string): { [key: string]: string } {
    const date = new Date(utcTimestamp);
    const timezones = {
      'UTC': 'UTC',
      'America/New_York': 'Eastern Time',
      'America/Chicago': 'Central Time', 
      'America/Denver': 'Mountain Time',
      'America/Los_Angeles': 'Pacific Time',
      'Europe/London': 'London',
      'Europe/Paris': 'Paris',
      'Asia/Tokyo': 'Tokyo',
      'Asia/Shanghai': 'Shanghai',
      'Asia/Kolkata': 'India Standard Time',
      'Australia/Sydney': 'Sydney'
    };

    const result: { [key: string]: string } = {};
    
    Object.entries(timezones).forEach(([tz, label]) => {
      result[label] = this.datePipe.transform(date, 'yyyy-MM-dd HH:mm:ss z', tz) || '';
    });

    return result;
  }
}
```

### Python
```python
from datetime import datetime, timezone

# Generate UTC timestamp
utc_timestamp = datetime.now(timezone.utc).isoformat()  # "2024-12-19T15:30:45.123456+00:00"

# Parse UTC timestamp
utc_datetime = datetime.fromisoformat("2024-12-19T15:30:45Z")
```

### Mule (MuleSoft)
```xml
<!-- Generate UTC timestamp -->
<set-variable variableName="utcTimestamp" value="#[now() as String {format: 'yyyy-MM-dd''T''HH:mm:ss''Z''}']" />
```

### PostgreSQL Database
```sql
-- Create table with UTC timestamp
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    data JSONB
);

-- Insert UTC timestamp
INSERT INTO events (event_timestamp, data) 
VALUES ('2024-12-19T15:30:45Z', '{"event": "user_login"}');
```

### MongoDB
```javascript
// Insert document with UTC timestamp
db.events.insertOne({
    event_timestamp: new Date(), // Automatically stored as UTC
    data: { event: "user_login" }
});
```

## 5. Implementation Checklist

### Data Storage
- [ ] Store all timestamps in UTC using `yyyy-MM-ddThh:mm:ssZ` format
- [ ] Add fractional seconds only when precision is required
- [ ] Implement strict validation for incoming timestamps
- [ ] Never store local time without UTC conversion

### UI Layer
- [ ] Convert UTC to user timezone for display only
- [ ] Use timezone-aware libraries for accurate conversions
- [ ] Handle DST transitions in display logic
- [ ] Show timezone indicators to users

### Testing
- [ ] Test DST transition periods
- [ ] Test with different timezone settings
- [ ] Validate timestamp format consistency
- [ ] Test timezone conversion accuracy

## 6. Common Pitfalls

### ❌ Bad: Storing Local Time
```javascript
// Don't store local time
const badTimestamp = "2024-12-19 15:30:45";
```

### ✅ Good: Storing UTC
```javascript
// Always store in UTC
const goodTimestamp = "2024-12-19T15:30:45Z";
```

### ❌ Bad: Converting at Storage Layer
```javascript
// Don't convert timezone before storing
const localTime = new Date().toLocaleString();
```

### ✅ Good: Converting at UI Layer Only
```javascript
// Convert only for display
const utcTime = "2024-12-19T15:30:45Z";
const displayTime = new Date(utcTime).toLocaleString();
```

---

*Last updated: December 19, 2024*
