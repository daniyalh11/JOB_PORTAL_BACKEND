 🚀 Job Importer Backend

A scalable job import system built with Node.js, Express, MongoDB, Redis, and BullMQ, designed to fetch jobs from multiple external XML APIs, queue them for background processing, and track detailed import history logs.

📁 Project Structure
/server
├── src
│   ├── config/           # Environment variables, Redis, MongoDB configs
│   ├── jobs/             # Bull/BullMQ job definitions and queue setup
│   ├── workers/          # Worker process logic
│   ├── models/           # Mongoose models (Job, ImportLog)
│   ├── services/         # Business logic (fetching, transforming, inserting)
│   ├── controllers/      # Route controllers for API endpoints
│   ├── routes/           # Express routes
│   ├── utils/            # Helper functions (XML → JSON, logging, etc.)
│   └── app.js            # Express app initialization
│
├── .env.example          # Example environment variables
├── package.json
├── README.md             # (this file)
└── server.js             # Entry point

🧠 Overview
Core Features

XML to JSON Job Fetching

Integrates with multiple external job feed APIs.

Converts XML responses to JSON.

Supports multiple job categories and regions.

Queue-Based Background Processing (Redis + BullMQ)

Each fetched job is queued for insertion or update in MongoDB.

Concurrency controlled by environment variables.

Retry logic and error handling for failed jobs.

Import History Tracking

Logs details of every import run:

timestamp

totalFetched

newJobs

updatedJobs

failedJobs (with reasons)

Stored in import_logs collection.

Cron-Based Auto Imports

Cron job runs every hour to fetch and queue new jobs.

API Endpoints for Monitoring

View import logs and job statistics.

⚙️ Tech Stack
Component	Technology
Runtime	Node.js (Express.js)
Database	MongoDB (Mongoose ORM)
Queue	BullMQ
Queue Store	Redis
Scheduler	node-cron
Data Parsing	xml2js
Environment Config	dotenv
Logging	Winston or console logs
🔧 Installation & Setup
1️⃣ Clone the repository
git clone https://github.com/<your-username>/<repo-name>.git
cd server

2️⃣ Install dependencies
npm install

3️⃣ Create .env file

Create a .env file in the /server directory based on .env.example:

PORT=5000
MONGO_URI=mongodb+srv://<your-mongo-uri>
REDIS_URL=redis://localhost:6379

# Optional
CRON_SCHEDULE=*/60 * * * *  # Runs every 60 mins
QUEUE_CONCURRENCY=5

4️⃣ Run Redis & MongoDB

If running locally:

# Start Redis
redis-server

# Start MongoDB (if local)
mongod


Or use managed services:

Redis Cloud

MongoDB Atlas

5️⃣ Start the server
Development:
npm run dev

Production:
npm start


Server will start on:

http://localhost:5000

🧩 API Endpoints
🗂️ Job Routes
Method	Endpoint	Description
GET	/api/jobs	Fetch all imported jobs
GET	/api/jobs/:id	Fetch single job
POST	/api/jobs/import	Manually trigger import
📜 Import Log Routes
Method	Endpoint	Description
GET	/api/import-logs	Get all import logs
GET	/api/import-logs/:id	Get details of a specific import
🧱 Architecture Flow
XML Feed APIs → Fetch Service → Queue (BullMQ / Redis) → Worker → MongoDB → Import Log

Flow Explanation

Fetch Service

Fetches XML job feeds from configured APIs.

Converts them to JSON and queues jobs for processing.

Queue (BullMQ)

Each job is pushed into Redis.

Worker consumes jobs with defined concurrency.

Worker

Checks if a job exists (by unique ID or title).

Inserts new or updates existing records.

Logs failures.

Import Logger

After every import cycle, logs summary stats in import_logs collection.

📅 Cron Job (Auto Import)

Set up in /src/services/cron.service.js
Runs automatically every hour (configurable via CRON_SCHEDULE in .env).

📊 Collections Schema
jobs
{
  title: String,
  company: String,
  location: String,
  description: String,
  type: String,
  url: String,
  updatedAt: Date
}

import_logs
{
  fileName: String,
  totalFetched: Number,
  totalImported: Number,
  newJobs: Number,
  updatedJobs: Number,
  failedJobs: [
    { jobId: String, reason: String }
  ],
  createdAt: { type: Date, default: Date.now }
}

🧠 Design Decisions

BullMQ over Bull → modern API, TypeScript-ready, better for distributed workloads.

MongoDB for flexible schema → ideal for varying job feed formats.

XML parsing using xml2js → robust for nested structures.

Cron + Worker separation → scalable horizontally; can later evolve to microservices.

Import logging → ensures auditability and transparency.

🧰 Useful Scripts
npm run dev       # Start dev server
npm run worker    # Start queue worker
npm run lint      # Run ESLint
