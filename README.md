# Concurrent Ticket Booking System

A production-ready Node.js/Express.js application demonstrating a concurrent ticket booking system with **seat locking mechanism** to prevent double-booking using **atomic database updates** and **distributed locks**.

## Features

✨ **Seat Locking Mechanism**
- Atomic seat locking using MongoDB's `findOneAndUpdate` operation
- Prevents race conditions and double-booking in concurrent environments
- Configurable lock expiry time (default: 5 minutes)

🔒 **Concurrency Control**
- MongoDB transactions for booking operations
- ACID compliance for multi-document operations
- Atomic updates to prevent data inconsistencies

🎬 **Show Management**
- Movie show creation and management
- Real-time seat availability tracking
- Seat status: available, locked, booked

💳 **Booking System**
- Multi-seat booking with atomic transaction support
- Booking reference generation
- User booking history
- Payment status tracking

## Project Structure

```
concurrent-ticket-booking/
├── models/
│   ├── Seat.js           # Seat model with lock tracking
│   ├── Show.js           # Movie show information
│   └── Booking.js        # Booking records with transaction support
├── controllers/
│   ├── seatController.js          # Seat locking & management logic
│   └── bookingController.js       # Booking & transaction handling
├── routes/
│   ├── seatRoutes.js              # Seat API endpoints
│   └── bookingRoutes.js           # Booking API endpoints
├── app.js                # Express server setup
├── package.json          # Project dependencies
├── .env.example          # Environment variables template
└── README.md             # Project documentation
```

## Dependencies

- **express**: ^4.18.2 - Web framework
- **mongoose**: ^7.0.0 - MongoDB ODM with transaction support
- **dotenv**: ^16.0.3 - Environment variable management
- **body-parser**: ^1.20.2 - Request body parsing
- **cors**: ^2.8.5 - Cross-origin resource sharing

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/concurrent-ticket-booking.git
cd concurrent-ticket-booking

# Install dependencies
npm install

# Create .env file from template
cp .env.example .env

# Update .env with your configuration
# MONGODB_URI=mongodb://localhost:27017/ticket-booking
# PORT=3000
```

## Running the Application

```bash
# Start the server (development mode)
npm run dev

# Or production mode
npm start
```

The server will run on `http://localhost:3000`

## API Endpoints

### Seat Operations

**Lock a Seat**
- **POST** `/api/seats/lock`
- **Body**: `{ showId, seatNumber, userId }`
- **Response**: Locked seat details

**Unlock a Seat**
- **POST** `/api/seats/unlock`
- **Body**: `{ showId, seatNumber, userId }`
- **Response**: Unlocked seat details

**Get Available Seats**
- **GET** `/api/seats/available/:showId`
- **Response**: List of available seats

**Get All Seats**
- **GET** `/api/seats/all/:showId`
- **Response**: All seats with their status

### Booking Operations

**Create Booking**
- **POST** `/api/bookings/create`
- **Body**: `{ showId, seatNumbers, userId, email, phone }`
- **Response**: Booking details with reference number

**Get Booking**
- **GET** `/api/bookings/:bookingId`
- **Response**: Booking details with seat information

**Get User Bookings**
- **GET** `/api/bookings/user/:userId`
- **Response**: All bookings for a user

## How Concurrency is Handled

### Seat Locking (Atomic Operation)
```javascript
const lockedSeat = await Seat.findOneAndUpdate(
  {
    showId,
    seatNumber,
    status: 'available' // Atomic check
  },
  {
    status: 'locked',
    lockedBy: userId,
    lockExpiry: new Date(Date.now() + 5 * 60 * 1000)
  },
  { new: true }
);
```
This ensures **only one user** can lock a seat at a time due to MongoDB's atomic operation guarantee.

### Booking Transaction (ACID Compliance)
```javascript
const session = await mongoose.startSession();
session.startTransaction();
try {
  // Multiple operations on locked seats
  // Create booking
  // Update seat status to booked
  // Update show inventory
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
}
```
If any operation fails, all changes are rolled back, ensuring data consistency.

## Testing Concurrent Bookings

```bash
# Test concurrent seat locking (simulate multiple users)
curl -X POST http://localhost:3000/api/seats/lock \
  -H "Content-Type: application/json" \
  -d '{
    "showId": "<show_id>",
    "seatNumber": "A1",
    "userId": "user1"
  }'

# Create booking with multiple seats
curl -X POST http://localhost:3000/api/bookings/create \
  -H "Content-Type: application/json" \
  -d '{
    "showId": "<show_id>",
    "seatNumbers": ["A1", "A2"],
    "userId": "user1",
    "email": "user@example.com",
    "phone": "1234567890"
  }'
```

## Key Technologies

- **Node.js/Express** - Server framework
- **MongoDB** - NoSQL database with transaction support
- **Mongoose** - ODM with schema validation
- **Atomic Operations** - MongoDB findOneAndUpdate for lock safety
- **MongoDB Sessions & Transactions** - ACID compliance

## Error Handling

The application handles:
- ✅ Double-booking prevention through atomic operations
- ✅ Lock expiration management
- ✅ Transaction rollback on failure
- ✅ Proper HTTP status codes and error messages
- ✅ Validation of required fields

## Environment Variables

```
MONGODB_URI=mongodb://localhost:27017/ticket-booking
PORT=3000
NODE_ENV=development
```

## License

MIT

## Author

akashsinhamahapatra78-cmd

---

**Note:** This is a demonstration project showcasing best practices for handling concurrency in ticket booking systems using Node.js and MongoDB.
