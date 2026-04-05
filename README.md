# Flight-Reservation-System
``` ✈  Project Description — Flight Reservation System
 
This project focuses on designing and implementing a relational database system for a Flight Reservation System using MySQL. The system models core entities involved in airline operations, including airports, airlines, routes, flights, passengers, bookings, seats, payments, loyalty programs, and fare classes. Each table is designed with appropriate primary keys, foreign keys, and constraints to maintain data integrity and establish clear relationships between the components of the reservation system.
 
SQL queries are implemented to handle essential operations such as managing seat availability and booking status, processing flight cancellations, handling overbooking situations, prioritizing passenger upgrades based on fare class or loyalty tier, and generating payment records. These queries help minimize scheduling conflicts, automate upgrade decisions, and ensure accurate revenue tracking across flights.
 
The project demonstrates how a well-structured relational database can power a production-grade flight reservation platform — enabling real-time seat management, loyalty program integration, and seamless passenger experience from booking to boarding.
  
✈  Key Objectives
 
•	Design a schema to store flight information including routes, schedules, and seat capacities
•	Implement queries to handle seat availability, booking status, and flight cancellations
•	Handle overbooking situations and prioritize passenger upgrades based on loyalty programs or fare classes
•	Track payment records, refund statuses, and generate billing summaries per booking
•	Manage passenger profiles and link them to loyalty reward points and tier levels
•	Support multi-leg journey bookings with layover information across routes
  
✈  Project Information
 
Project Title	Flight Reservation System
Technology	MySQL (Relational Database)
Domain	Airlines / Travel Management
Key Entities	Airports, Airlines, Flights, Routes, Passengers, Bookings, Seats, Payments, Loyalty Programs, Fare Classes
Core Features	Seat booking, Overbooking management, Fare class upgrades, Loyalty points, Cancellations & Refunds
  
✈  Database Schema Overview
 
Table	Key Columns
Airports	airport_id (PK), airport_code, name, city, country
Airlines	airline_id (PK), airline_name, iata_code, country
Routes	route_id (PK), origin_airport_id (FK), destination_airport_id (FK), distance_km
Flights	flight_id (PK), airline_id (FK), route_id (FK), departure_time, arrival_time, status, total_seats
Passengers	passenger_id (PK), full_name, email, phone, passport_no, nationality
Fare_Classes	fare_class_id (PK), class_name (Economy/Business/First), base_multiplier
Seats	seat_id (PK), flight_id (FK), seat_number, fare_class_id (FK), status (Available/Booked/Reserved)
Bookings	booking_id (PK), passenger_id (FK), seat_id (FK), flight_id (FK), booking_date, booking_status
Payments	payment_id (PK), booking_id (FK), amount, payment_status, payment_method, payment_date
Loyalty_Programs	loyalty_id (PK), passenger_id (FK), tier (Bronze/Silver/Gold), points_balance
Cancellations	cancellation_id (PK), booking_id (FK), reason, refund_status, cancelled_at
