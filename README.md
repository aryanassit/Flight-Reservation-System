# SkyRes Pro — Flight Reservation System

A fully functional flight reservation system built as a single HTML file with an in-browser SQLite database (sql.js). No backend server, no installation, no dependencies to install — just open the file in a browser and it works.

---

## What It Does

This system covers all core operations of an airline reservation workflow:

- Manage flights, routes, and seat capacity across 3 cabin classes
- Register passengers with loyalty tier profiles
- Book seats with a visual seat map
- Handle overbooking with an automated upgrade priority system
- Track loyalty points and member benefits
- Run live SQL queries against the database

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, Vanilla JavaScript |
| Database | SQLite via [sql.js](https://github.com/sql-js/sql.js) (in-browser) |
| SQL Engine | sql.js 1.10.2 (WebAssembly port of SQLite) |
| Hosting | None required — runs locally in browser |

---

## How to Run

1. Download `flight_reservation_system.html`
2. Open it in any modern browser (Chrome, Firefox, Edge)
3. The database initializes automatically and loads demo data
4. Start using the system immediately

> **Note:** Requires an internet connection on first load to fetch sql.js from the CDN. After that it runs fully in the browser.

---

## Database Schema

The system uses 4 tables:

### `flights`
Stores all flight route and capacity information.

```sql
CREATE TABLE flights (
  flight_id     INTEGER PRIMARY KEY AUTOINCREMENT,
  flight_number VARCHAR(10) NOT NULL,
  aircraft_type VARCHAR(50),
  origin        CHAR(3) NOT NULL,        -- IATA code e.g. DEL
  destination   CHAR(3) NOT NULL,        -- IATA code e.g. BOM
  departure_dt  DATETIME NOT NULL,
  arrival_dt    DATETIME NOT NULL,
  eco_capacity  INTEGER DEFAULT 120,
  biz_capacity  INTEGER DEFAULT 24,
  first_cap     INTEGER DEFAULT 8,
  status        VARCHAR(20) DEFAULT 'Scheduled',
  CHECK (status IN ('Scheduled','Boarding','Departed','Arrived','Cancelled'))
);
```

### `passengers`
Stores passenger profiles and loyalty program membership.

```sql
CREATE TABLE passengers (
  passenger_id  INTEGER PRIMARY KEY AUTOINCREMENT,
  full_name     VARCHAR(100) NOT NULL,
  email         VARCHAR(100) UNIQUE NOT NULL,
  passport_no   VARCHAR(20) UNIQUE,
  nationality   VARCHAR(50),
  loyalty_tier  VARCHAR(20) DEFAULT 'Bronze',
  loyalty_pts   INTEGER DEFAULT 0,
  created_at    DATETIME DEFAULT CURRENT_TIMESTAMP,
  CHECK (loyalty_tier IN ('Bronze','Silver','Gold','Platinum'))
);
```

### `bookings`
Links passengers to flights with seat and fare information.

```sql
CREATE TABLE bookings (
  booking_id    INTEGER PRIMARY KEY AUTOINCREMENT,
  booking_ref   VARCHAR(10) UNIQUE NOT NULL,
  passenger_id  INTEGER NOT NULL REFERENCES passengers(passenger_id),
  flight_id     INTEGER NOT NULL REFERENCES flights(flight_id),
  fare_class    VARCHAR(20) NOT NULL,
  seat_number   VARCHAR(5),
  fare_amount   DECIMAL(10,2),
  booking_dt    DATETIME DEFAULT CURRENT_TIMESTAMP,
  status        VARCHAR(20) DEFAULT 'Confirmed',
  CHECK (status IN ('Confirmed','Pending','Cancelled','Waitlisted')),
  CHECK (fare_class IN ('Economy','Business','First'))
);
```

### `upgrade_log`
Audit trail for every cabin upgrade performed.

```sql
CREATE TABLE upgrade_log (
  log_id        INTEGER PRIMARY KEY AUTOINCREMENT,
  booking_id    INTEGER REFERENCES bookings(booking_id),
  from_class    VARCHAR(20),
  to_class      VARCHAR(20),
  upgraded_at   DATETIME DEFAULT CURRENT_TIMESTAMP,
  reason        VARCHAR(200)
);
```

### Indices

```sql
CREATE INDEX idx_bookings_flight     ON bookings(flight_id);
CREATE INDEX idx_bookings_passenger  ON bookings(passenger_id);
CREATE INDEX idx_bookings_status     ON bookings(status);
CREATE INDEX idx_passengers_loyalty  ON passengers(loyalty_tier, loyalty_pts);
```

---

## Features

### Dashboard
- Live metrics: total flights, confirmed bookings, average load factor, overbooked flight count
- Flight load factor table with visual progress bars
- Recent bookings list with status badges

### Flights Module
- Add new flights with IATA origin/destination codes, aircraft type, departure/arrival times
- Set capacity independently for Economy, Business, and First class
- Cancel flights
- View all flights with current status

### New Booking
- Select passenger and flight from dropdowns
- Choose fare class (Economy / Business / First)
- Visual seat map — Economy in 3-3 layout, Business in 2-2, First in 1-1
- Click to select a specific seat or auto-assign
- System automatically marks booking as `Waitlisted` if flight is full
- Loyalty points awarded on confirmation — multiplied by tier (Bronze 1x, Silver 1.5x, Gold 2x, Platinum 3x)

### All Bookings
- Filter by status (Confirmed / Pending / Cancelled / Waitlisted)
- Filter by fare class
- Cancel individual bookings

### Passengers
- Register new passengers with passport, nationality, and loyalty details
- View full registry sorted by loyalty points
- See confirmed booking count per passenger

### Overbooking Management
- Per-flight overbooking status table
- Upgrade priority queue sorted by loyalty tier then fare amount
- One-click manual upgrade or auto-upgrade (picks highest priority passenger automatically)
- Full upgrade history log with reason and timestamp

### Loyalty Program
- Tier metrics: Platinum / Gold / Silver / Bronze member counts
- Benefits table showing lounge access, extra baggage, points multiplier per tier
- Points leaderboard with visual progress bars

### SQL Console
- Free-form SQL editor — run any query against the live database
- 6 preset queries:
  - Seat availability per flight
  - Loyalty ranking
  - All confirmed bookings
  - Overbooked flights
  - Revenue by fare class
  - Upgrade history
- Results displayed as a formatted table

---

## Booking Status Flow

```
New booking
    |
    ├── Flight has capacity → Confirmed
    |
    └── Flight is full → Waitlisted
                              |
                              └── Cancellation occurs → auto-promote to Confirmed
```

Bookings can also be manually set to:
- `Pending` — payment not yet completed
- `Cancelled` — passenger or airline cancelled

---

## Overbooking Logic

When a flight exceeds its total seat capacity:

1. New bookings are automatically marked `Waitlisted`
2. Upgrade queue sorts Economy passengers by:
   - Loyalty tier (Platinum first, then Gold, Silver, Bronze)
   - Fare amount (higher fare breaks ties within the same tier)
3. Top-ranked passenger gets upgraded to Business class
4. Upgrade is logged in `upgrade_log` with reason and timestamp

---

## Loyalty Tier Benefits

| Tier | Min Points | Upgrade Priority | Lounge Access | Extra Baggage | Points Multiplier |
|---|---|---|---|---|---|
| Bronze | 0 | 4th | None | 0 kg | 1x |
| Silver | 5,000 | 3rd | Domestic | 5 kg | 1.5x |
| Gold | 20,000 | 2nd | All lounges | 10 kg | 2x |
| Platinum | 50,000 | 1st | Platinum + guest | 20 kg | 3x |

---

## Demo Data

Clicking **Reload demo data** seeds the database with:

- 5 flights across Indian domestic and international routes (DEL, BOM, BLR, CCU, HYD, LHR)
- 8 passengers across all loyalty tiers
- 12 bookings with a mix of Confirmed, Waitlisted, Pending, and Cancelled statuses

---

## Key SQL Queries Reference

**Load factor per flight:**
```sql
SELECT f.flight_number,
  ROUND(100.0 * COUNT(CASE WHEN b.status='Confirmed' THEN 1 END)
  / (f.eco_capacity + f.biz_capacity + f.first_cap), 1) AS load_pct
FROM flights f
LEFT JOIN bookings b ON f.flight_id = b.flight_id
GROUP BY f.flight_id
HAVING load_pct > 80;
```

**Overbooked flights:**
```sql
SELECT f.flight_number, COUNT(b.booking_id) - (f.eco_capacity+f.biz_capacity+f.first_cap) AS overbooked_by
FROM flights f JOIN bookings b ON f.flight_id = b.flight_id
WHERE b.status IN ('Confirmed','Waitlisted')
GROUP BY f.flight_id
HAVING COUNT(b.booking_id) > (f.eco_capacity+f.biz_capacity+f.first_cap);
```

**Upgrade priority queue:**
```sql
SELECT p.full_name, p.loyalty_tier, b.fare_amount
FROM bookings b JOIN passengers p ON b.passenger_id = p.passenger_id
WHERE b.status = 'Confirmed' AND b.fare_class = 'Economy'
ORDER BY CASE p.loyalty_tier WHEN 'Platinum' THEN 1 WHEN 'Gold' THEN 2 WHEN 'Silver' THEN 3 ELSE 4 END,
         b.fare_amount DESC;
```

---

## Limitations

Since this uses an in-browser SQLite database:

- **Data is not persistent** — refreshing the browser resets all data
- **Single user only** — no multi-user concurrency support
- **No real authentication** — no login or role-based access
- **No seat locking** — two users could theoretically book the same seat simultaneously (not relevant for single-user use)

For a production system, replace sql.js with a backend using PostgreSQL or MySQL, add a REST API layer (Node.js / Django / Spring Boot), and connect a proper frontend framework.

---

## Project Structure

```
flight_reservation_system.html
├── <style>          CSS — layout, components, badges, seat map
├── HTML             8 tab panels (dashboard, flights, book, bookings,
│                    passengers, overbooking, loyalty, sql console)
└── <script>
    ├── DB init      Creates 4 tables + indices on load
    ├── seedData()   Inserts demo flights, passengers, bookings
    ├── CRUD ops     addFlight, addPassenger, makeBooking, cancelBooking
    ├── Render fns   One function per tab to query and render the UI
    ├── Overbooking  autoUpgrade, upgradePassenger, priority queue
    └── SQL console  runSQL() with formatted table output
```

---

## Author

Built as a full-stack academic project demonstrating relational database design, SQL query writing, and frontend integration using a live in-browser SQLite engine.
