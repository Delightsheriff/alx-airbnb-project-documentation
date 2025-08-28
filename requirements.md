# Airbnb Clone Backend – Technical & Functional Requirements

## 📌 1. User Authentication

### **Objective**

Provide secure user registration, login, and session management for guests, hosts, and admins.

### **API Endpoints**

- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/auth/profile`

### **Input / Output**

#### `POST /api/auth/register`

**Input (JSON):**

```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "role": "guest|host"
}
```

**Output (Success):**

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "role": "guest",
  "token": "jwt_token_here"
}
```

**Output (Error):**

```json
{
  "error": "Email already exists"
}
```

### **Validation Rules**

- Email must be unique and RFC-5322 compliant.
- Password must be ≥ 8 chars, at least one uppercase, one number, one special char.
- Role must be either `guest` or `host`.

### **Performance Criteria**

- Authentication responses ≤ 200ms under normal load.
- Passwords stored using salted bcrypt (min cost factor 12).
- Tokens expire in 24h, refresh supported.

---

## 📌 2. Property Management

### **Objective**

Enable hosts to create, update, and delete property listings with images and availability details.

### **API Endpoints**

- `POST /api/properties`
- `GET /api/properties?location=&price=&guests=`
- `PUT /api/properties/:id`
- `DELETE /api/properties/:id`

### **Input / Output**

#### `POST /api/properties`

**Input (JSON):**

```json
{
  "title": "Cozy Apartment",
  "description": "2-bed apartment with city view",
  "location": "New York",
  "price": 120,
  "guests": 4,
  "amenities": ["wifi", "air_conditioning"]
}
```

**Output (Success):**

```json
{
  "id": "uuid",
  "title": "Cozy Apartment",
  "status": "active"
}
```

### **Validation Rules**

- Title ≤ 150 chars.
- Price > 0 and ≤ 10,000.
- Guests between 1–20.
- Location must match supported geocoding format.
- Only authenticated **hosts** can create listings.

### **Performance Criteria**

- Listing search must respond ≤ 500ms (with pagination).
- Property images stored in cloud (e.g., AWS S3, Cloudinary).
- Cache frequently accessed listings (Redis).

---

## 📌 3. Booking System

### **Objective**

Allow guests to book properties with date validation, prevent double bookings, and manage booking status.

### **API Endpoints**

- `POST /api/bookings`
- `GET /api/bookings/:id`
- `PUT /api/bookings/:id/cancel`
- `GET /api/bookings?userId=`

### **Input / Output**

#### `POST /api/bookings`

**Input (JSON):**

```json
{
  "propertyId": "uuid",
  "checkIn": "2025-09-10",
  "checkOut": "2025-09-15",
  "guests": 2
}
```

**Output (Success):**

```json
{
  "id": "uuid",
  "status": "pending",
  "totalAmount": 600,
  "paymentLink": "https://stripe.com/pay/123"
}
```

**Output (Error):**

```json
{
  "error": "Property not available for selected dates"
}
```

### **Validation Rules**

- Dates must follow ISO-8601 (`YYYY-MM-DD`).
- `checkIn < checkOut`.
- Guests ≤ property capacity.
- Booking linked to authenticated **guest**.
- Prevent overlapping bookings for same property.

### **Performance Criteria**

- Booking availability check must run ≤ 200ms.
- Double-booking prevention must be ACID-compliant at DB transaction level.
- Bookings indexed by `propertyId` and `checkIn/checkOut`.

---
