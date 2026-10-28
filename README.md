# Vehicle Inspection & Maintenance Dashboard (Full-Stack)

A modern full-stack web application with a **React (Vite + Tailwind CSS)** frontend and an **Express (TypeScript)** backend, configured to deploy seamlessly together on **Vercel**.

The backend securely connects to the **Google Sheets API** using a Google Service Account via server-side environment variables, parses and formats the fleet inspection records, and exposes REST endpoints. The frontend displays real-time KPI metrics, analytics charts (Recharts), and an interactive data table (TanStack Table) with sorting, filtering, pagination, and a detailed modal view with Google Drive certificate links.

---

## 🌟 Features

- **Google Sheets API v4 Integration**:
  - Secure server-side service account authentication via JWT.
  - Automatically parses rows into typed `InspectionRecord` entities.
  - Formats vehicle types, plate numbers, dates, partner contacts, and Google Drive certificate attachments.
  - Built-in in-memory caching (60s TTL) with instant manual cache-busting (`?forceRefresh=true`).
- **RESTful Express API**:
  - `GET /api/inspections`: Returns formatted records, summary statistics, and supports search, filtering, and pagination.
  - `GET /api/inspections/stats`: Returns aggregated fleet statistics.
  - `GET /api/health`: Service health and uptime monitoring.
- **Modern Responsive React Frontend**:
  - Built with **React 18**, **Vite**, **TypeScript**, and **Tailwind CSS**.
  - **KPI Metric Cards**: Total requests, Pre-onboarding vs existing vehicle count, reviews needed.
  - **Fleet Analytics Charts (Recharts)**: Vehicle models distribution, inspection type breakdown, processing status.
  - **TanStack Table (v8)**:
    - Sort by column (ascending/descending)
    - Full-text search across partner, plate number, city, comments
    - Dropdown filters for Vehicle Model, Inspection Type, and Review Status
    - Interactive pagination (10, 25, 50, 100 per page)
    - Clickable certificate badges (Road Worthiness, Hackney Permit, Insurance, Speed Limit, Vehicle License) opening Google Drive files in new tabs
  - **Record Detail Modal**: Inspect all fields, driver details, partner contact info, and notes for any record.
- **Vercel Full-Stack Ready**:
  - Standardized `vercel.json` config.
  - Serverless entry point in `api/index.ts` routing `/api/*` requests to Express.
  - Static Vite build output directed to `client/dist`.

---

## 📁 Project Architecture

```
assignment/
├── api/
│   └── index.ts                 # Vercel Serverless Function entry point
├── client/                      # React (Vite + Tailwind CSS + TypeScript)
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── AnalyticsCharts.tsx
│   │   │   ├── DataTable.tsx
│   │   │   ├── Header.tsx
│   │   │   ├── RecordModal.tsx
│   │   │   └── StatsCards.tsx
│   │   ├── types/
│   │   │   └── inspection.ts
│   │   ├── App.tsx
│   │   ├── index.css
│   │   └── main.tsx
│   ├── index.html
│   ├── package.json
│   ├── postcss.config.js
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   └── vite.config.ts
├── server/                      # Express (TypeScript)
│   ├── src/
│   │   ├── config/
│   │   │   └── google.ts        # Google Sheets Service Account Auth
│   │   ├── controllers/
│   │   │   └── sheetController.ts
│   │   ├── routes/
│   │   │   └── sheetRoutes.ts
│   │   ├── services/
│   │   │   └── googleSheetsService.ts
│   │   ├── types/
│   │   │   └── inspection.ts
│   │   ├── app.ts               # Express application configuration
│   │   └── index.ts             # Express local server entry point
│   ├── package.json
│   └── tsconfig.json
├── .env                         # Server environment variables (Ignored in Git)
├── .env.example                 # Example template for environment variables
├── .gitignore
├── package.json                 # Root package with dev/build scripts
├── README.md
└── vercel.json                  # Vercel deployment configuration
```

---

## 🔑 Environment Variables Setup

Create a `.env` file in the project root (and in `server/`) based on `.env.example`:

```env
# Server Configuration
PORT=5000
NODE_ENV=development

# Google Sheets Configuration
GOOGLE_SHEET_ID=1gyuhg7Gx84acLtbZed506b4wOW8R31Cmn2p6VZVFxWs
GOOGLE_SHEET_NAME=Inspection Request

# Google Service Account Credentials (Server-side Only)
GOOGLE_PROJECT_ID=dashboard-512907
GOOGLE_CLIENT_EMAIL=sheets-qa@dashboard-512907.iam.gserviceaccount.com
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"

# Optional: Local fallback path to JSON key file
GOOGLE_APPLICATION_CREDENTIALS=./dashboard-512907-27495dea56a2.json
```

> **Security Note:** Never expose `GOOGLE_PRIVATE_KEY` on the client. It is strictly kept on the server/Vercel serverless environment.

---

## 🚀 Getting Started (Local Development)

### 1. Install Dependencies
Run from the root directory:
```bash
npm run install:all
```
Or install in each directory:
```bash
npm install
cd server && npm install
cd ../client && npm install
cd ..
```

### 2. Start Both Server & Client
Run from root:
```bash
npm run dev
```

- **Frontend**: `http://localhost:5173`
- **Backend API**: `http://localhost:5000`
- **API Health Check**: `http://localhost:5000/api/health`
- **API Inspections Data**: `http://localhost:5000/api/inspections`

Vite is pre-configured with a reverse proxy for `/api`, so client requests to `/api/inspections` automatically route to `http://localhost:5000/api/inspections` without CORS issues.

---

## ☁️ Deploying to Vercel

1. **Push your code to GitHub / GitLab / Bitbucket**.
2. **Import the repository into Vercel**.
3. **Configure Environment Variables** in Vercel Project Settings:
   - `GOOGLE_SHEET_ID`: `1gyuhg7Gx84acLtbZed506b4wOW8R31Cmn2p6VZVFxWs`
   - `GOOGLE_SHEET_NAME`: `Inspection Request`
   - `GOOGLE_PROJECT_ID`: `dashboard-512907`
   - `GOOGLE_CLIENT_EMAIL`: `sheets-qa@dashboard-512907.iam.gserviceaccount.com`
   - `GOOGLE_PRIVATE_KEY`: paste the private key (including BEGIN/END headers, quotes, and newlines).
4. **Deploy**:
   - Vercel automatically runs `npm run build` which builds the Vite frontend into `client/dist`.
   - Vercel detects `api/index.ts` and deploys it as a Serverless Function handling all `/api/*` routes.
   - All other routes are served statically from `client/dist`.

---

## 📡 API Reference

### 1. `GET /api/inspections`
Returns formatted sheet records and aggregate summary.

**Query Parameters:**
- `search` (string): Text query across partner name, phone, plate number, city, notes.
- `inspectionType` (string): Filter by inspection type (e.g. `Pre-Onboarding Inspection`).
- `vehicleType` (string): Filter by vehicle model (e.g. `Coaster`, `Hiace`, `Sienna`).
- `status` (string): Filter by status (`Sent`, `Manual Review — No Centre Match`, `Pending`).
- `page` (number) & `limit` (number): Optional server-side pagination.
- `forceRefresh` (boolean): Pass `true` to invalidate server cache and fetch live from Google Sheets API.

**Response Structure:**
```json
{
  "success": true,
  "sheetInfo": {
    "title": "Vehicle Inspection and Maintenance Records",
    "sheetName": "Inspection Request",
    "totalRows": 868,
    "lastFetchedAt": "2026-10-27T19:25:00.000Z"
  },
  "stats": {
    "totalRecords": 868,
    "byInspectionType": { ... },
    "byVehicleType": { ... },
    "byStatus": { ... },
    "totalVehiclesInspected": 1150
  },
  "data": [
    {
      "id": 1,
      "rowNumber": 863,
      "timestamp": "20/10/2026 01:06:36",
      "inspectionType": "Pre-Onboarding Inspection",
      "vehicleType": "GX460",
      "vehiclesCount": 1,
      "partnerName": "Chidera Luke Ezeh",
      "partnerEmail": "chideraluke2@gmail.com",
      "partnerPhone": "08142356862",
      "location": "Lekki, Lagos",
      "partnershipStatus": "New Partner",
      "plateNumber": "AKD-245KX",
      "status": "Pending",
      "documents": {
        "vehicleLicense": "https://drive.google.com/open?id=...",
        "roadWorthiness": "https://drive.google.com/open?id=...",
        "hackneyPermit": "https://drive.google.com/open?id=...",
        "insurance": "https://drive.google.com/open?id=...",
        "speedLimit": "https://drive.google.com/open?id=..."
      }
    }
  ]
}
```

### 2. `GET /api/inspections/stats`
Returns aggregated analytics and breakdown metrics.

### 3. `GET /api/health`
Health check endpoint reporting API uptime and status.
