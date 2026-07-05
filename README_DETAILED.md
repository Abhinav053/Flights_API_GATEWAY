# API Gateway for Flights - Comprehensive Documentation

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Key Features](#key-features)
4. [Technology Stack](#technology-stack)
5. [Project Structure Explained](#project-structure-explained)
6. [Installation & Setup Guide](#installation--setup-guide)
7. [Environment Configuration](#environment-configuration)
8. [Database Setup](#database-setup)
9. [Authentication & Authorization](#authentication--authorization)
10. [API Endpoints Documentation](#api-endpoints-documentation)
11. [Running the Application](#running-the-application)
12. [Docker Setup & Deployment](#docker-setup--deployment)
13. [User Roles & Permissions](#user-roles--permissions)
14. [Database Models & Relationships](#database-models--relationships)
15. [Error Handling](#error-handling)
16. [Best Practices](#best-practices)
17. [Common Issues & Solutions](#common-issues--solutions)

---

## Project Overview

### What is this Project?

This is an **API Gateway** application built with Node.js and Express that serves as a central hub for managing user authentication, authorization, and routing requests to microservices. Specifically, it's designed for a **Flight Booking System** that routes requests to:

- **Flight Service** (handles flights data)
- **Booking Service** (handles booking operations)

### Purpose

The API Gateway acts as a single entry point for client applications, providing:
- User registration and authentication
- JWT-based token management
- Role-based access control (Admin, Customer, Flight Company)
- Request rate limiting for security
- Proxy routing to backend microservices
- Centralized error handling and logging

### Who Should Use This?

- Developers building microservice-based flight booking systems
- Teams needing a scalable authentication gateway
- Anyone implementing role-based access control in Node.js applications

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Applications                       │
│                   (Web, Mobile, Desktop)                     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (Port 3000)                   │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Express Application                       │    │
│  │  ┌────────────────────────────────────────────┐    │    │
│  │  │  Rate Limiter (2 min windows, 30 requests)│    │    │
│  │  └────────────────────────────────────────────┘    │    │
│  │                                                     │    │
│  │  ┌────────────────────────────────────────────┐    │    │
│  │  │  Authentication & Authorization Routes     │    │    │
│  │  │  • Signup                                   │    │    │
│  │  │  • Signin                                   │    │    │
│  │  │  • Role Management                          │    │    │
│  │  └────────────────────────────────────────────┘    │    │
│  │                                                     │    │
│  │  ┌────────────────────────────────────────────┐    │    │
│  │  │  Proxy Middleware                           │    │    │
│  │  │  • /flightsService → Flight Service         │    │    │
│  │  │  • /bookingService → Booking Service        │    │    │
│  │  └────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Database Layer (MySQL)                   │    │
│  │  • User Management                                   │    │
│  │  • Role Management                                   │    │
│  │  • User-Role Associations                            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
         │                              │
         ▼                              ▼
   ┌──────────────┐            ┌──────────────┐
   │Flight Service│            │Booking Service│
   │(Port 3001)   │            │(Port 3002)    │
   └──────────────┘            └──────────────┘
```

---

## Key Features

### 1. **Authentication System**
   - User registration with email and password
   - Secure password hashing using **bcrypt**
   - JWT token generation for session management
   - Token verification and validation
   - 3-day token expiry (configurable)

### 2. **Authorization & Role-Based Access Control**
   - Three user roles: **Admin**, **Customer**, **Flight Company**
   - Role assignment to users
   - Admin-only endpoints
   - Fine-grained permission control

### 3. **Security Features**
   - Password encryption with bcrypt (10 salt rounds)
   - JWT token-based authentication
   - Rate limiting (30 requests per 2 minutes per IP)
   - Request validation middleware
   - Error handling with custom AppError class

### 4. **Microservice Integration**
   - HTTP proxy to Flight Service
   - HTTP proxy to Booking Service
   - Dynamic service URL configuration
   - Request forwarding with proper routing

### 5. **Database Management**
   - SQL migrations for version control
   - Database seeders for initial data
   - ORM using Sequelize
   - MySQL compatibility
   - Model associations and relationships

### 6. **Logging & Monitoring**
   - Winston logger integration
   - Structured logging for debugging
   - Error tracking and logging
   - Request/response logging capabilities

---

## Technology Stack

### Backend Framework & Server
| Component | Purpose | Version |
|-----------|---------|---------|
| **Express.js** | Web server framework | 4.18.2 |
| **Node.js** | JavaScript runtime | Latest |
| **Nodemon** | Development auto-reload | 2.0.22 |

### Database & ORM
| Component | Purpose | Version |
|-----------|---------|---------|
| **MySQL2** | MySQL database driver | 3.2.4 |
| **Sequelize** | ORM for database operations | 6.31.1 |
| **Sequelize CLI** | Database migrations & seeders | 6.6.0 |

### Authentication & Security
| Component | Purpose | Version |
|-----------|---------|---------|
| **bcrypt** | Password hashing | 5.1.0 |
| **jsonwebtoken** | JWT token management | 9.0.0 |
| **express-rate-limit** | Rate limiting middleware | 6.7.0 |

### Configuration & Utilities
| Component | Purpose | Version |
|-----------|---------|---------|
| **dotenv** | Environment variable management | 16.0.3 |
| **http-proxy-middleware** | Request proxying | 2.0.6 |
| **http-status-codes** | HTTP status code constants | 2.2.0 |
| **Winston** | Logging library | 3.8.2 |

---

## Project Structure Explained

```
API_Gateway_Flights/
│
├── 📁 src/                          # Source code directory
│   ├── index.js                     # Application entry point
│   │
│   ├── 📁 config/                   # Configuration files
│   │   ├── config.json              # Database configuration (dev/test/prod)
│   │   ├── index.js                 # Config exports
│   │   ├── logger-config.js         # Winston logger setup
│   │   └── server-config.js         # Server configuration (PORT, JWT, etc.)
│   │
│   ├── 📁 controllers/              # Request handlers
│   │   ├── index.js                 # Controller exports
│   │   ├── info-controller.js       # Basic info endpoint
│   │   └── user-controller.js       # User signup/signin/role management
│   │
│   ├── 📁 middlewares/              # Custom middleware functions
│   │   ├── index.js                 # Middleware exports
│   │   └── auth-request-middlewares.js  # Authentication validators
│   │
│   ├── 📁 models/                   # Database models (Sequelize)
│   │   ├── index.js                 # Model exports & initialization
│   │   ├── user.js                  # User model with associations
│   │   ├── role.js                  # Role model with enum values
│   │   └── user_role.js             # Junction table (many-to-many)
│   │
│   ├── 📁 migrations/               # Database schema changes
│   │   ├── 20230601152546-create-user.js       # Create users table
│   │   ├── 20230603070701-create-role.js       # Create roles table
│   │   └── 20230603072634-create-user-role.js  # Create user_roles table
│   │
│   ├── 📁 seeders/                  # Initial data insertion
│   │   └── 20230603072341-add-roles.js  # Seed initial roles
│   │
│   ├── 📁 repositories/             # Database access layer (Data Access Object)
│   │   ├── index.js                 # Repository exports
│   │   ├── crud-repository.js       # Base CRUD operations
│   │   ├── user-repository.js       # User-specific queries
│   │   └── role-repository.js       # Role-specific queries
│   │
│   ├── 📁 services/                 # Business logic layer
│   │   ├── index.js                 # Service exports
│   │   └── user-service.js          # User operations & auth logic
│   │
│   ├── 📁 routes/                   # API route definitions
│   │   ├── index.js                 # Main router
│   │   └── v1/                      # API version 1
│   │       ├── index.js             # V1 route aggregator
│   │       └── user-routes.js       # User authentication routes
│   │
│   └── 📁 utils/                    # Utility functions
│       ├── index.js                 # Utils exports
│       ├── 📁 common/               # Common utilities
│       │   ├── auth.js              # JWT & password utilities
│       │   ├── enums.js             # Role enums
│       │   ├── error-response.js    # Error response formatter
│       │   ├── success-response.js  # Success response formatter
│       │   └── index.js             # Common utils exports
│       └── 📁 errors/               # Error classes
│           └── app-error.js         # Custom AppError class
│
├── .env                             # Environment variables (not in repo)
├── .gitignore                       # Git ignore rules
├── package.json                     # Project dependencies & scripts
├── Dockerfile                       # Docker containerization
└── README.md                        # Original README
```

### Directory Purposes Explained

#### **config/**
Handles all configuration setup:
- `config.json`: Database credentials for different environments
- `server-config.js`: Loads environment variables (PORT, JWT_SECRET, etc.)
- `logger-config.js`: Winston logger configuration for structured logs

#### **controllers/**
Acts as the request handler layer:
- Receives HTTP requests from routes
- Validates request data
- Calls appropriate services
- Formats and sends responses back
- Catches errors and returns error responses

#### **middlewares/**
Intercepts requests before they reach controllers:
- `validateAuthRequest`: Checks if email & password exist
- `checkAuth`: Validates JWT token in headers
- `isAdmin`: Verifies user has admin role

#### **models/**
Database schema definitions using Sequelize:
- Defines table structure
- Sets up validations
- Establishes relationships between tables
- Includes hooks for password encryption

#### **repositories/**
Data access abstraction layer:
- Contains all database queries
- Implements pagination, filtering
- Handles raw and ORM queries
- Separates business logic from database queries

#### **services/**
Core business logic layer:
- User signup/signin logic
- Role assignment logic
- Authentication/authorization checks
- Validation logic

#### **routes/**
Maps HTTP endpoints to controllers:
- Defines which controller handles which HTTP method/path
- Applies middleware to specific routes
- Structures API versions

#### **utils/**
Helper functions and utilities:
- `auth.js`: Password comparison, JWT creation/verification
- `enums.js`: Defines role constants
- `error-response.js`: Standardized error response format
- `success-response.js`: Standardized success response format

---

## Installation & Setup Guide

### Prerequisites
Ensure you have installed:
- **Node.js** (v14 or higher)
- **npm** (comes with Node.js)
- **MySQL Server** (v5.7 or higher)
- **Git** (optional, for version control)

### Step 1: Clone or Download the Project
```bash
# If using git
git clone <repository-url>
cd API_Gateway_Flights

# Or extract the ZIP file and navigate to the directory
cd API_Gateway_Flights
```

### Step 2: Install Dependencies
```bash
npm install
```

This will install all packages listed in `package.json`:
- Express, Sequelize, bcrypt, jsonwebtoken, etc.

### Step 3: Setup Environment Variables
Create a `.env` file in the root directory with:
```bash
PORT=3000
SALT_ROUNDS=10
JWT_EXPIRY=3d
JWT_SECRET=your_super_secret_key
FLIGHT_SERVICE=http://localhost:3001/api
BOOKING_SERVICE=http://localhost:3002/api
```

**Explanation of each variable:**
- `PORT`: Server listening port
- `SALT_ROUNDS`: Bcrypt salt rounds for password hashing (higher = more secure but slower)
- `JWT_EXPIRY`: Token expiration time (3d = 3 days)
- `JWT_SECRET`: Secret key for signing JWT tokens (keep it secure!)
- `FLIGHT_SERVICE`: URL of the flight microservice
- `BOOKING_SERVICE`: URL of the booking microservice

### Step 4: Configure Database
Edit `src/config/config.json` to match your MySQL setup:
```json
{
  "development": {
    "username": "root",
    "password": "your_password",
    "database": "flights_db",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": "test_password",
    "database": "flights_db_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "prod_user",
    "password": "prod_password",
    "database": "flights_db_prod",
    "host": "your_prod_host",
    "dialect": "mysql"
  }
}
```

---

## Environment Configuration

### Understanding env Variables

| Variable | Purpose | Example | Security |
|----------|---------|---------|----------|
| `PORT` | Server port | 3000 | Public |
| `SALT_ROUNDS` | Password hashing strength | 10 | Internal |
| `JWT_EXPIRY` | Token expiration | 3d, 24h, 7d | Internal |
| `JWT_SECRET` | Token signing secret | strong_random_string | **CRITICAL - Keep Secret!** |
| `FLIGHT_SERVICE` | Flight service URL | http://localhost:3001/api | Internal |
| `BOOKING_SERVICE` | Booking service URL | http://localhost:3002/api | Internal |

### Security Best Practices for .env

```bash
# ❌ WRONG - Never commit .env file
git add .env

# ✅ RIGHT - .env is in .gitignore
cat .gitignore  # Should contain ".env"

# ✅ Use environment-specific variables
# For production, set environment variables on the server
export JWT_SECRET="very_long_random_string_here"
export PORT=8080

# Never hardcode secrets in the code!
```

---

## Database Setup

### Step 1: Initialize Sequelize
```bash
cd src
npx sequelize init
```

This creates:
- `migrations/` folder (already exists, will be preserved)
- `seeders/` folder (already exists, will be preserved)
- `config/config.json` (already exists, will be updated)

### Step 2: Create Database
```bash
# Connect to MySQL
mysql -u root -p

# Create the database
CREATE DATABASE flights_db;
EXIT;
```

### Step 3: Run Migrations
```bash
npx sequelize db:migrate
```

**What this does:**
- Creates `users` table with email, password fields
- Creates `roles` table with role names (admin, customer, flight_company)
- Creates `user_roles` junction table for many-to-many relationship

**Migration Files Explained:**

**20230601152546-create-user.js:**
```javascript
// Creates users table with:
// - id (primary key, auto-increment)
// - email (unique, required)
// - password (required, hashed)
// - createdAt, updatedAt (timestamps)
```

**20230603070701-create-role.js:**
```javascript
// Creates roles table with:
// - id (primary key)
// - name (ENUM: 'admin', 'customer', 'flight_company')
// - createdAt, updatedAt
```

**20230603072634-create-user-role.js:**
```javascript
// Creates user_roles junction table with:
// - id (primary key)
// - userId (foreign key to users)
// - roleId (foreign key to roles)
// - createdAt, updatedAt
```

### Step 4: Seed Initial Roles
```bash
npx sequelize db:seed:all
```

**What gets seeded:**
- Admin role
- Customer role
- Flight Company role

These roles are now available for assignment to users.

### Database Schema Visualization

```
┌─────────────────────────────┐
│        users                │
├─────────────────────────────┤
│ id (PK)       | INT         │
│ email         | VARCHAR     │ (UNIQUE)
│ password      | VARCHAR     │ (hashed)
│ createdAt     | TIMESTAMP   │
│ updatedAt     | TIMESTAMP   │
└─────────────────────────────┘
         │
         │ many-to-many
         │ via user_roles
         │
┌─────────────────────────────┐
│      user_roles             │
├─────────────────────────────┤
│ id (PK)       | INT         │
│ userId (FK)   | INT         │ ──→ users.id
│ roleId (FK)   | INT         │ ──→ roles.id
│ createdAt     | TIMESTAMP   │
│ updatedAt     | TIMESTAMP   │
└─────────────────────────────┘
         │
         │ many-to-many
         │ via user_roles
         │
┌─────────────────────────────┐
│      roles                  │
├─────────────────────────────┤
│ id (PK)       | INT         │
│ name          | ENUM        │ (admin/customer/flight_company)
│ createdAt     | TIMESTAMP   │
│ updatedAt     | TIMESTAMP   │
└─────────────────────────────┘
```

---

## Authentication & Authorization

### How Authentication Works (Step by Step)

#### **Signup Flow**
```
1. User sends: POST /api/v1/user/signup
   Body: {email: "user@example.com", password: "123456"}
           ↓
2. Route → validateAuthRequest Middleware
   - Checks if email exists
   - Checks if password exists
   - If missing, returns 400 error
           ↓
3. Route → UserController.signup
   - Calls UserService.create()
           ↓
4. UserService.create()
   - Validates email format
   - Checks if email already exists
   - Hashes password using bcrypt
   - Creates user in database
   - Assigns 'customer' role by default
           ↓
5. Response: 201 Created
   {
     "success": true,
     "data": {
       "id": 1,
       "email": "user@example.com",
       "createdAt": "2023-06-03T12:30:00Z"
     },
     "error": null
   }
```

#### **Signin Flow**
```
1. User sends: POST /api/v1/user/signin
   Body: {email: "user@example.com", password: "123456"}
           ↓
2. Route → validateAuthRequest Middleware
   - Validates email & password exist
           ↓
3. Route → UserController.signin
   - Calls UserService.signin()
           ↓
4. UserService.signin()
   - Fetches user by email
   - If not found, throws 404 error
   - Compares plain password with hashed password using bcrypt
   - If no match, throws 400 error
   - Generates JWT token with user id & email
           ↓
5. Response: 201 Created
   {
     "success": true,
     "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
     "error": null
   }
```

#### **Protected Request Flow**
```
1. User sends: POST /api/v1/user/role
   Header: x-access-token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
           ↓
2. Route → checkAuth Middleware
   - Extracts token from x-access-token header
   - Calls UserService.isAuthenticated(token)
   - Verifies token signature using JWT_SECRET
   - Extracts user id from token
   - Fetches user from database
   - Sets req.user = userId
           ↓
3. Route → isAdmin Middleware
   - Calls UserService.isAdmin(req.user)
   - Fetches user and admin role
   - Checks if user has admin role
   - If not, returns 401 Unauthorized
           ↓
4. Route → UserController.addRoleToUser
   - (Only reached if user is authenticated and admin)
   - Adds role to another user
           ↓
5. Response: Success with updated user
```

### Password Hashing Process

```
User Password: "mySecurePassword123"
                    ↓
           bcrypt.hashSync()
           (10 salt rounds)
                    ↓
Hashed Password: "$2b$10$N9qo8uLOickgx2ZMRZoMye$"
                    ↓
Stored in Database
```

**Why multiple salt rounds?**
- 1 round ≈ 2 hashes
- 10 rounds ≈ 2^10 = 1024 hashes
- Makes password cracking exponentially harder
- Trade-off: More rounds = more secure but slower

### JWT Token Structure

```
JWT Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwi...

Structure:
┌──────────────────────────────────────────────────────┐
│ Header.Payload.Signature                             │
├──────────────────────────────────────────────────────┤
│ Decoded Header:                                      │
│ {                                                    │
│   "alg": "HS256",                                   │
│   "typ": "JWT"                                      │
│ }                                                    │
├──────────────────────────────────────────────────────┤
│ Decoded Payload:                                     │
│ {                                                    │
│   "id": 1,                    ← user id             │
│   "email": "user@example.com",                       │
│   "iat": 1685793000,          ← issued at           │
│   "exp": 1685966800           ← expires at          │
│ }                                                    │
├──────────────────────────────────────────────────────┤
│ Signature: HMAC-SHA256(header + payload, secret)    │
│ Verifies token wasn't tampered with                 │
└──────────────────────────────────────────────────────┘
```

---

## API Endpoints Documentation

### Base URL
```
http://localhost:3000/api
```

### Authentication Endpoints

#### **1. User Signup**
Create a new user account.

**Request:**
```http
POST /v1/user/signup
Content-Type: application/json

{
  "email": "newuser@example.com",
  "password": "securePassword123"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "newuser@example.com",
    "updatedAt": "2023-06-03T10:30:00.000Z",
    "createdAt": "2023-06-03T10:30:00.000Z"
  },
  "error": null
}
```

**Error Cases:**
```json
// Email already exists
{
  "success": false,
  "data": null,
  "error": "Validation error: Email must be unique"
}

// Invalid email format
{
  "success": false,
  "data": null,
  "error": "Validation error: Email must be valid"
}

// Missing email or password
{
  "success": false,
  "data": null,
  "error": "Email not found in the incoming request in the correct form"
}
```

---

#### **2. User Signin**
Authenticate a user and get JWT token.

**Request:**
```http
POST /v1/user/signin
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwi...",
  "error": null
}
```

**Send this token in future requests:**
```http
POST /v1/user/role
x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Error Cases:**
```json
// User not found
{
  "success": false,
  "data": null,
  "error": "No user found for the given email"
}

// Wrong password
{
  "success": false,
  "data": null,
  "error": "Invalid password"
}
```

---

#### **3. Add Role to User**
Assign a role to an existing user. **Admin only.**

**Request:**
```http
POST /v1/user/role
Content-Type: application/json
x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "id": 2,
  "role": "admin"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 2,
    "email": "otheruser@example.com",
    "createdAt": "2023-06-03T10:30:00.000Z",
    "updatedAt": "2023-06-03T10:35:00.000Z",
    "role": [
      {
        "id": 1,
        "name": "admin",
        "createdAt": "2023-06-03T08:00:00.000Z",
        "updatedAt": "2023-06-03T08:00:00.000Z",
        "User_Roles": {
          "createdAt": "2023-06-03T10:35:00.000Z",
          "updatedAt": "2023-06-03T10:35:00.000Z"
        }
      }
    ]
  },
  "error": null
}
```

**Error Cases:**
```json
// Not authenticated
{
  "statusCode": 400,
  "message": "Missing JWT token"
}

// Not admin
{
  "message": "User not authorized for this action"
}

// User not found
{
  "success": false,
  "data": null,
  "error": "No user found for the given id"
}

// Role doesn't exist
{
  "success": false,
  "data": null,
  "error": "No user found for the given role"
}
```

**Available Roles:**
- `admin` - Full system access
- `customer` - Can book flights
- `flight_company` - Can manage flights

---

#### **4. Info Endpoint**
Basic health check endpoint.

**Request:**
```http
GET /v1/info
```

**Response (200 OK):**
```json
{
  "message": "API Gateway is running"
}
```

---

### Proxy Endpoints

#### **5. Flight Service Proxy**
Forward requests to Flight Service.

**Request:**
```http
GET /flightsService/flights
x-access-token: [your_token]
```

Routes to: `http://localhost:3001/api/flights`

---

#### **6. Booking Service Proxy**
Forward requests to Booking Service.

**Request:**
```http
POST /bookingService/bookings
x-access-token: [your_token]
Body: {...}
```

Routes to: `http://localhost:3002/api/bookings`

---

### Response Format

All API responses follow a standard format:

**Success Response:**
```json
{
  "success": true,
  "data": { /* actual data */ },
  "error": null
}
```

**Error Response:**
```json
{
  "success": false,
  "data": null,
  "error": {
    "statusCode": 400,
    "message": "Error description",
    "explanation": ["Detailed explanation 1", "Detailed explanation 2"]
  }
}
```

---

## Running the Application

### Development Mode
```bash
npm run dev
```

This uses **Nodemon** which:
- Auto-restarts server when files change
- Perfect for development
- Watches all files in `src/` directory

**Output:**
```
> nodemon src/index.js

[nodemon] 2.0.22
[nodemon] to restart at any time, type `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,json
http://localhost:3001/api
Successfully started the server on PORT : 3000
```

### Production Mode
```bash
node src/index.js
```

No auto-restart. Better performance.

### Testing the API

**Using cURL:**
```bash
# Signup
curl -X POST http://localhost:3000/api/v1/user/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Signin
curl -X POST http://localhost:3000/api/v1/user/signin \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Add Role (replace TOKEN with actual JWT)
curl -X POST http://localhost:3000/api/v1/user/role \
  -H "Content-Type: application/json" \
  -H "x-access-token: TOKEN" \
  -d '{"id":2,"role":"admin"}'
```

**Using Postman:**
1. Create new request
2. Set method to POST
3. Enter URL: `http://localhost:3000/api/v1/user/signup`
4. Go to Body → raw → JSON
5. Complete! Send request

---

## Docker Setup & Deployment

### What is Docker?
Docker containerizes your application so it runs the same everywhere (dev, staging, production).

### Prerequisites
- Install [Docker Desktop](https://www.docker.com/products/docker-desktop)
- Docker should be running

### Building the Docker Image

**Step 1: Build the image**
```bash
docker build -t api-gateway .
```

**Step 2: Create Docker network**
```bash
docker network create micro-net
```

This network allows containers to communicate with each other.

**Step 3: Create volume for node_modules**
```bash
docker volume create api-gateway-node-modules
```

Volumes persist data between container restarts.

**Step 4: Run the container**
```bash
docker run -it --init \
  -p 3001:3001 \
  --name=api_gateway \
  --network micro-net \
  -v "$(pwd)":/developer/nodejs/api-gateway \
  -v api-gateway-node-modules:/developer/nodejs/api-gateway/node_modules \
  api-gateway:latest
```

**Parameter Explanation:**
| Parameter | Meaning |
|-----------|---------|
| `-it` | Interactive terminal |
| `--init` | Use init process (prevents zombie processes) |
| `-p 3001:3001` | Map port 3001 (host:container) |
| `--name` | Container name |
| `--network` | Connect to network |
| `-v` | Mount volume (host:container path) |

**Access the running container:**
```bash
docker exec -it api_gateway bash
# Now you can run commands inside the container
```

**Stop the container:**
```bash
docker stop api_gateway
```

**Remove the container:**
```bash
docker rm api_gateway
```

### Docker Compose (Optional)
Create `docker-compose.yml` for easier management:

```yaml
version: '3.8'
services:
  api-gateway:
    build: .
    container_name: api_gateway
    ports:
      - "3001:3001"
    volumes:
      - .:/developer/nodejs/api-gateway
      - api-gateway-node-modules:/developer/nodejs/api-gateway/node_modules
    networks:
      - micro-net
    environment:
      - NODE_ENV=development
      - PORT=3001

volumes:
  api-gateway-node-modules:

networks:
  micro-net:
    driver: bridge
```

Then run:
```bash
docker-compose up
```

---

## User Roles & Permissions

### Three Role System

#### **1. Admin**
- **Permissions:**
  - Create, read, update, delete users
  - Assign roles to users
  - Access all endpoints
  - Manage system settings
  
- **Identifier:** "admin"

- **Example Signup →**
  ```
  Signup as admin → needs manual role assignment
  POST /api/v1/user/role with admin token
  {id: new_user_id, role: "admin"}
  ```

#### **2. Customer**
- **Default role** assigned on signup
- **Permissions:**
  - View available flights
  - Make bookings
  - View own bookings
  - Update profile
  - Cannot access admin endpoints
  
- **Identifier:** "customer"

#### **3. Flight Company**
- **Permissions:**
  - Add flights
  - Update flight information
  - View flight bookings
  - Manage flight schedules
  - Cannot create bookings for other companies
  
- **Identifier:** "flight_company"

### Role Assignment Flow

```
Admin User (id=1)
      ↓
Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSw...
      ↓
POST /api/v1/user/role
{
  "id": 2,
  "role": "flight_company"
}
      ↓
checkAuth Middleware ✓ (Token valid)
      ↓
isAdmin Middleware ✓ (User 1 is admin)
      ↓
addRoleToUser ✓
      ↓
User 2 now has role: flight_company
```

---

## Database Models & Relationships

### User Model
```javascript
User {
  id: Integer (Primary Key, Auto-increment)
  email: String (Unique, Required, Valid email format)
  password: String (Required, 3-50 chars, hashed with bcrypt)
  createdAt: Timestamp (Auto-generated)
  updatedAt: Timestamp (Auto-generated)
  
  // Associations
  roles: [Role] (Many-to-many through User_Roles)
}
```

**Methods:**
- `addRole(role)` - Add a role to user
- `hasRole(role)` - Check if user has a role
- `getRoles()` - Get all roles

### Role Model
```javascript
Role {
  id: Integer (Primary Key)
  name: ENUM('admin', 'customer', 'flight_company')
  createdAt: Timestamp
  updatedAt: Timestamp
  
  // Associations
  users: [User] (Many-to-many through User_Roles)
}
```

### User_Role Model (Junction Table)
```javascript
User_Role {
  userId: Integer (Foreign Key to User)
  roleId: Integer (Foreign Key to Role)
  createdAt: Timestamp
  updatedAt: Timestamp
}
```

### Relationship Diagram

```
User → User_Role ← Role

One user can have multiple roles
One role can be assigned to multiple users

Example:
User(id=1, email=admin@example.com)
         ↓
         User_Role(userId=1, roleId=1)
         User_Role(userId=1, roleId=3)
         ↓
    Role(id=1, name=admin)
    Role(id=3, name=flight_company)

Result: User 1 has both admin and flight_company roles
```

---

## Error Handling

### Error Hierarchy

```
AppError extends Error {
  - message: String (user-friendly message)
  - statusCode: HTTP status code
  - explanation: Array<String> (detailed explanations)
}
```

### Common HTTP Status Codes

| Code | Meaning | Example |
|------|---------|---------|
| 201 | Created | User signup success |
| 400 | Bad Request | Missing email field |
| 401 | Unauthorized | Invalid JWT token |
| 404 | Not Found | User doesn't exist |
| 429 | Too Many Requests | Rate limited |
| 500 | Server Error | Database connection failed |

### Error Examples

**Missing Required Field:**
```json
{
  "statusCode": 400,
  "message": "Something went wrong while authenticating user",
  "explanation": ["Email not found in the incoming request in the correct form"]
}
```

**Token Expired:**
```json
{
  "statusCode": 400,
  "message": "JWT token expired",
  "explanation": "Your session has expired. Please login again."
}
```

**Database Validation Error:**
```json
{
  "statusCode": 400,
  "message": "Validation error",
  "explanation": [
    "Email must be unique",
    "Password must be between 3-50 characters"
  ]
}
```

### How to Handle Errors in Your Code

```javascript
try {
  // Your async operation
  const user = await userService.create(data);
} catch(error) {
  if(error instanceof AppError) {
    // Handle our custom errors
    console.log(error.message);
    console.log(error.statusCode);
    console.log(error.explanation);
  } else {
    // Handle unexpected errors
    console.log('Unexpected error:', error);
  }
}
```

---

## Best Practices

### Code Organization Best Practices Used

✅ **Layered Architecture**
- Controllers don't talk directly to database
- Services handle business logic
- Repositories handle data access
- Clean separation of concerns

✅ **DRY Principle (Don't Repeat Yourself)**
- `CrudRepository` base class for common operations
- Shared error handling
- Reusable middleware

✅ **Error Handling**
- Custom `AppError` class
- Consistent error responses
- Proper HTTP status codes

✅ **Security**
- Passwords hashed with bcrypt
- JWT tokens for stateless auth
- Rate limiting on all routes
- Input validation with middleware

✅ **Environment Management**
- Environment-specific database configs
- Sensitive data in .env
- .env in .gitignore

✅ **Configuration**
- Centralized configuration in `config/`
- Easy to switch between environments
- Logging configuration centralized

### Security Recommendations

❌ **Never do this:**
```javascript
// WRONG: Storing plain passwords
password: "myPassword123"

// WRONG: Hardcoding secrets
const JWT_SECRET = "abc123"

// WRONG: Committing .env file
git add .env
```

✅ **Always do this:**
```javascript
// RIGHT: Hash passwords
password: bcrypt.hashSync(plainPassword, saltRounds)

// RIGHT: Use environment variables
const JWT_SECRET = process.env.JWT_SECRET

// RIGHT: .env in .gitignore
echo ".env" >> .gitignore
```

### Performance Best Practices

1. **Database Indexing**
   ```sql
   -- Create index on frequently queried fields
   CREATE INDEX idx_user_email ON users(email);
   ```

2. **Pagination** (implement in services)
   ```javascript
   async function getUsers(page, limit) {
     return User.findAll({
       offset: (page - 1) * limit,
       limit: limit
     });
   }
   ```

3. **Caching** (consider Redis for sessions)
   ```javascript
   // Don't fetch user from DB every time
   // Cache after first fetch
   ```

4. **Query Optimization**
   - Use `include` in Sequelize to avoid N+1 queries
   - Select only needed fields
   - Use pagination

---

## Common Issues & Solutions

### Issue 1: "Cannot find module 'express'"
**Cause:** Dependencies not installed

**Solution:**
```bash
npm install
```

---

### Issue 2: "ECONNREFUSED: Connection refused to 127.0.0.1:3306"
**Cause:** MySQL server not running

**Solution (Windows):**
```bash
# Start MySQL service
net start MySQL80

# Or through Services app
# Search → Services → MySQL80 → Start
```

**Solution (Mac):**
```bash
mysql.server start
```

**Solution (Linux):**
```bash
sudo systemctl start mysql
```

---

### Issue 3: "Database 'flights_db' doesn't exist"
**Cause:** Database not created

**Solution:**
```bash
mysql -u root -p
CREATE DATABASE flights_db;
EXIT;

npx sequelize db:migrate
npx sequelize db:seed:all
```

---

### Issue 4: "listen EADDRINUSE: address already in use :::3000"
**Cause:** Port 3000 already in use

**Solution:**
```bash
# Windows: Find and kill process on port 3000
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Mac/Linux:
lsof -i :3000
kill -9 <PID>

# Or use a different port in .env
PORT=3001
```

---

### Issue 5: "Invalid JWT token"
**Cause:** Token expired or corrupted

**Solution:**
```bash
# Get a new token by signing in again
POST /api/v1/user/signin
```

---

### Issue 6: "bcrypt not compiling on Windows"
**Cause:** Missing build tools

**Solution:**
```bash
# Install windows-build-tools globally
npm install --global windows-build-tools

# Then reinstall bcrypt
npm rebuild bcrypt
```

---

## Advanced Topics

### Extending with New Endpoints

**Step 1: Create a migration** (if adding new table)
```bash
npx sequelize model:generate --name Flight --attributes code:string,airline:string
npx sequelize db:migrate
```

**Step 2: Create model** (`src/models/flight.js`)

**Step 3: Create repository** (`src/repositories/flight-repository.js`)

**Step 4: Create service** (`src/services/flight-service.js`)

**Step 5: Create controller** (`src/controllers/flight-controller.js`)

**Step 6: Create routes** (`src/routes/v1/flight-routes.js`)

**Step 7: Register routes** (in `src/routes/v1/index.js`)

---

### Implementing Request Logging

Add to `src/index.js`:
```javascript
const logger = require('winston');

app.use((req, res, next) => {
  logger.info(`${req.method} ${req.path}`, {
    ip: req.ip,
    timestamp: new Date()
  });
  next();
});
```

---

### Database Backup & Restore

**Backup:**
```bash
mysqldump -u root -p flights_db > backup.sql
```

**Restore:**
```bash
mysql -u root -p flights_db < backup.sql
```

---

## Conclusion

This API Gateway provides a **production-ready foundation** for building microservice-based applications with:
- ✅ Secure authentication & authorization
- ✅ Clean, maintainable code architecture
- ✅ Comprehensive error handling
- ✅ Easy to extend and customize
- ✅ Docker-ready for deployment

### Next Steps

1. ✅ Set up the project locally
2. ✅ Create and seed the database
3. ✅ Test the authentication endpoints
4. ✅ Connect the microservices (Flight, Booking)
5. ✅ Deploy using Docker

---

## Support & Resources

- **Express.js Documentation:** https://expressjs.com/
- **Sequelize ORM:** https://sequelize.org/
- **JWT Introduction:** https://jwt.io/introduction
- **bcrypt Security:** https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- **Docker Guide:** https://docs.docker.com/

---

**Created:** 2023
**Version:** 1.0.0
**Author:** Sanket

---

*Last Updated: March 2026*
