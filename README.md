# AuthCheck

A Node.js REST API that verifies a mobile number with an SMS one-time password before registering the user.

![Node.js](https://img.shields.io/badge/Node.js-339933?logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![Twilio](https://img.shields.io/badge/Twilio-F22F46?logo=twilio&logoColor=white)

## About

AuthCheck is a small Express backend for OTP-based sign-up. A client asks for a code to be sent to a mobile number, the server sends a 6-digit OTP by SMS through Twilio and keeps it in the user's session, and the sign-up request is only accepted when the submitted OTP matches. Registered users are stored in MongoDB and can be fetched by ID.

## Features

- Generates a random 6-digit numeric OTP and sends it by SMS using Twilio
- Stores the pending OTP and mobile number in an `express-session` session
- Creates the user only after the OTP matches, then clears the OTP from the session
- Stores a student-style profile in MongoDB (name, date of birth, gender, mobile and guardian numbers, address, school, class, board, subjects, referral code)
- Fetches a user by MongoDB ObjectId, with checks for invalid IDs and missing users

## Tech Stack

- **Runtime:** Node.js
- **Framework:** Express 4
- **Database:** MongoDB with Mongoose 8
- **SMS:** Twilio
- **Sessions:** express-session
- **Config:** dotenv
- **Dev:** nodemon

## Project Structure

```
AuthCheck/
├── app.js                     # Express app, session setup, MongoDB connection, server on port 3000
├── controllers/
│   └── authController.js      # Send OTP, sign up, get user by ID
├── models/
│   └── User.js                # Mongoose user schema
├── routes/
│   └── authRoutes.js          # Route definitions
├── utils/
│   ├── auth.js                # Twilio client: sendOTP and verifyOTP
│   └── generateOTP.js         # Random numeric OTP generator
├── .example.env               # Environment variable template
└── package.json
```

## Getting Started

### Prerequisites

- Node.js and npm
- A MongoDB instance (local or MongoDB Atlas)
- A Twilio account with a phone number that can send SMS

### Installation

```bash
git clone https://github.com/AryanDeswal/AuthCheck.git
cd AuthCheck
npm install
```

### Environment Variables

Copy `.example.env` to `.env` and fill in the values:

```bash
cp .example.env .env
```

| Variable      | Purpose                                                                                   |
| ------------- | ----------------------------------------------------------------------------------------- |
| `DB_URL`      | MongoDB connection string. Defaults to `mongodb://localhost:27017/rabloDB` if not set.     |
| `Account_SID` | Twilio Account SID                                                                        |
| `Auth_Token`  | Twilio Auth Token                                                                         |

`.env` is only loaded when `NODE_ENV` is not `production`.

> **Note:** The Twilio sender number (`from`) is hard-coded in `utils/auth.js` and the session secret is hard-coded in `app.js`. Change both to your own values before running the project.

### Run

```bash
npm start       # node app.js
npm run serve   # nodemon app.js (auto-reload)
```

The server listens on `http://localhost:3000`.

## API Reference

| Method | Route       | Description                                                                 |
| ------ | ----------- | --------------------------------------------------------------------------- |
| POST   | `/verify`   | Generate a 6-digit OTP, send it by SMS to `mobileNumber`, store it in session |
| POST   | `/signup`   | Check `otp` against the session OTP and create the user                     |
| GET    | `/user/:id` | Get a user by MongoDB ObjectId                                              |

## Usage

The OTP is kept in the session, so the client must send the session cookie from `/verify` along with the `/signup` request (for example, use the same Postman or browser session).

**1. Request an OTP**

```http
POST /verify
Content-Type: application/json

{ "mobileNumber": "+919876543210" }
```

Response: `200 { "status": "OTP Sent Successfully" }`, or `500 { "status": "Error Sending Otp" }`.

**2. Sign up with the OTP**

```http
POST /signup
Content-Type: application/json

{
  "otp": "123456",
  "firstName": "Jane",
  "lastName": "Doe",
  "DateOfBirth": "2008-05-14",
  "Gender": "Female",
  "mobileNumber": "+919876543210",
  "guardianNumber": "+919812345678",
  "houseNo": "12B",
  "city": "Delhi",
  "state": "Delhi",
  "pinCode": "110001",
  "referalCode": "",
  "school": "Example Public School",
  "class": "10",
  "Boards": "CBSE",
  "subjectYouStudy": ["Maths", "Science"]
}
```

Response: `200 { "userId": "<id>" }`, or `500 { "status": "Invalid OTP" }` when the OTP does not match.

**3. Get a user**

```http
GET /user/<id>
```

Response: `200 { "data": { ...user } }`, `400` for an invalid ID, or `404` if no user is found.
