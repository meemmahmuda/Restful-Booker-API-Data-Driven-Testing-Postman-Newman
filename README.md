# Restful-Booker API – Data-Driven Testing (Postman + Newman)

Data-driven API test suite built with **Postman** and executed from the command line with **Newman**. The same request flow is run once per row of a data file, and an HTML report is generated with the **htmlextra** reporter.

**API under test:** [Restful-Booker](https://restful-booker.herokuapp.com) – a public hotel booking API built for practicing API testing.

---

## What this project covers

For every row in the data file (6 iterations), the collection runs this flow:

| # | Request | Method | Endpoint | Purpose |
|---|---------|--------|----------|---------|
| 1 | CreateBooking | `POST` | `/booking` | Create a booking from the data row |
| 2 | CreateToken | `POST` | `/auth` | Get an auth token (`admin` user) |
| 3 | UpdateBooking | `PUT` | `/booking/{id}` | Fully update the booking |
| 4 | PartialUpdateBooking | `PATCH` | `/booking/{id}` | Update first name and last name only |

### Assertions

- **Status code** is `201`
- **Response time** is below `500ms`
- **Response size** is below `1000B`
- **Response body values** (first name, last name, total price, deposit paid, check-in, check-out, additional needs) match the data that was sent
- **Data types** – first/last name are strings, total price is a number, deposit paid is a boolean
- **Token** is returned correctly
- **Updated values** are reflected after `PUT` and `PATCH`

---

## Latest run summary

Run date: 21 Oct 2026

| Metric | Result |
|--------|--------|
| Iterations | 6 |
| Total requests | 24 (0 failed) |
| Total assertions | 222 |
| Failed assertions | 13 |
| Skipped | 0 |
| Total run duration | 10.2s |
| Average response time | 335ms |
| Total data received | 3.29KB |

Full report: [**Download Newman HTML Report**](./newman) (download and open in a browser).

### Failure analysis

All 24 requests completed successfully (HTTP `200`). The 13 failed assertions come from just two checks:

| Failed check | Count | Reason |
|--------------|-------|--------|
| `Status code is 201` (CreateBooking) | 6 | API returns `200`, not `201` |
| `Status code is 201` (CreateToken) | 6 | API returns `200`, not `201` |
| `Response time below 500ms` (CreateBooking, iteration 1) | 1 | Took 1246ms (likely a Heroku cold start) |

These look like incorrect expectations in the tests rather than API bugs: Restful-Booker responds with `200` for both `POST /booking` and `POST /auth`. Updating the expected status code to `200` would clear 12 of the 13 failures. The response-time failure is intermittent and depends on the hosting server.

### Observation about the test data

In iteration 3 the data has a check-out date (`2025-03-06`) earlier than the check-in date (`2025-05-03`), and the API still accepted it. This suggests the API does not validate date order – a good candidate for a negative test case.

---

## Project structure

> Adjust the file names below to match your repository.

```
.
├── collection/
│   └── DDT.postman_collection.json
├── environment/
│   └── DDT.postman_environment.json
├── data/
│   └── data.csv                # or data.json
├── reports/
│   └── DDT-2026-10-21-19-06-30-082-0.html
└── README.md
```

---

## Prerequisites

- [Node.js](https://nodejs.org/) (LTS)
- [Newman](https://www.npmjs.com/package/newman)
- [newman-reporter-htmlextra](https://www.npmjs.com/package/newman-reporter-htmlextra)

```bash
npm install -g newman newman-reporter-htmlextra
```

---

## How to run

```bash
newman run collection/DDT.postman_collection.json \
  -e environment/DDT.postman_environment.json \
  -d data/data.csv \
  -r htmlextra \
  --reporter-htmlextra-export reports/report.html
```

| Flag | Meaning |
|------|---------|
| `-e` | Environment file |
| `-d` | Data file – one iteration per row |
| `-r htmlextra` | Use the htmlextra HTML reporter |
| `--reporter-htmlextra-export` | Output path of the report |

### Run in Postman (GUI)

1. Import the collection and environment files.
2. Open the collection and click **Run**.
3. Select the data file and set iterations to match the number of rows.
4. Click **Run DDT**.

---

## Data file format

Each row supplies the values for one iteration (booking details, plus the new names used for the update requests). Example of the booking payload built from one row:

```json
{
  "firstname": "Rubel",
  "lastname": "Hosain",
  "totalprice": 501,
  "depositpaid": 1,
  "bookingdates": {
    "checkin": "2025-03-28",
    "checkout": "2025-04-03"
  },
  "additionalneeds": "Lunch"
}
```

---

## Tech stack

- Postman
- Newman
- newman-reporter-htmlextra
- JavaScript (Postman test scripts)

---

## Author

**Your Name** – [GitHub](https://github.com/your-username)
