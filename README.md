# RMS — Java Edition

Recruitment Management System rebuilt with Spring Boot (Java) + Next.js frontend.
Runs fully locally — no cloud services required.

---

## Tech Stack

| Layer       | Technology                              |
|-------------|-----------------------------------------|
| Frontend    | Next.js 14, React, Tailwind CSS (unchanged) |
| Backend     | Spring Boot 3.x, Maven                 |
| ORM         | Spring Data JPA (Hibernate)             |
| Database    | MySQL (local)                           |
| Auth        | BCrypt password hashing, no JWT         |
| Language    | Java 21 (not compatible with Java 25)       |

---

## Architecture

```
Browser (Next.js)
      │
      │  HTTP requests to localhost:8080
      ▼
Spring Boot Controllers   ← Layer 1: receives requests, calls service
      │
Spring Boot Services      ← Business logic, validation, ownership checks
      │
Spring Data JPA Repos     ← Layer 2: talks to MySQL
      │
MySQL (local)
```

User ID is stored in localStorage after login and passed as a
query parameter (`?applicantId=...`) on all protected requests.

---

## Step 1 — Install MySQL

### Windows

1. Download MySQL Installer from https://dev.mysql.com/downloads/installer/
2. Run the installer → choose "Developer Default"
3. Set root password to `root` (or update `application.properties`)
4. Complete installation — MySQL Service starts automatically

Verify it's running:
```cmd
mysql -u root -p
```
Type your password. You should see the MySQL prompt `mysql>`. Type `exit` to quit.

### macOS (Homebrew)
```bash
brew install mysql
brew services start mysql
mysql_secure_installation   # set root password to: root
```

### Linux (Ubuntu/Debian)
```bash
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo mysql -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';"
```

---

## Step 2 — Create the Database

MySQL and Spring Boot will auto-create the `rms_db` database and all tables
on first startup (via `createDatabaseIfNotExist=true` in application.properties).

If you prefer to create it manually:
```sql
CREATE DATABASE rms_db;
```

---

## Step 3 — Install Java 21

Download from https://adoptium.net (Eclipse Temurin — free LTS)
Only Java 21 works with this project.
(Lombok compatible with Java 21 only)

Verify:
```cmd
java -version
```

---

## Step 4 — Install Maven

Download from https://maven.apache.org/download.cgi
Extract and add `bin/` to your PATH.

Verify:
```cmd
mvn -version
```

Alternatively, use the Maven Wrapper included in most Spring Boot projects
(no install needed): use `./mvnw` instead of `mvn` on Mac/Linux,
or `mvnw.cmd` on Windows.

---

## Step 5 — Run the Backend

```cmd
cd backend
mvn spring-boot:run
```

On first run, Maven downloads all dependencies (~2 minutes).
Subsequent runs are fast.

You should see:
```
Started RmsApplication in 3.4 seconds
```

Backend is now running at http://localhost:8080

---

## Step 6 — Run the Frontend

Copy the `.env.local` file into your frontend folder, then:

```cmd
cd frontend
npm install   (first time only)
npm run dev
```

Frontend is now running at http://localhost:3000

---

## API Endpoints

Since there is no JWT, the caller's ID is passed as a query parameter.

### Auth
```
POST /api/auth/applicant/register    body: { name, email, password, phone }
POST /api/auth/recruiter/register    body: { name, email, password, companyName }
POST /api/auth/login                 body: { email, password, role }
```

### Jobs
```
GET    /api/jobs                          ?keyword=&location=&skill=
GET    /api/jobs/{id}
GET    /api/jobs/recruiter/my             ?recruiterId=
POST   /api/jobs                          ?recruiterId=   body: { title, description, skillsRequired, location }
PUT    /api/jobs/{id}                     ?recruiterId=
DELETE /api/jobs/{id}                     ?recruiterId=
```

### Applications
```
POST   /api/applications                  ?applicantId=   body: { jobId, coverLetter }
GET    /api/applications/my               ?applicantId=
GET    /api/applications/job/{jobId}      ?recruiterId=
PUT    /api/applications/{id}/status      ?recruiterId=   body: { applicationStatus }
DELETE /api/applications/{id}             ?applicantId=
```

### Profiles
```
GET /api/applicants/profile    ?applicantId=
PUT /api/applicants/profile    ?applicantId=   body: { name, phone, profileDetails }
GET /api/recruiters/profile    ?recruiterId=
PUT /api/recruiters/profile    ?recruiterId=   body: { name, companyName }
```

---

## Project Structure

```
rms-java/
├── README.md
│
├── backend/
│   ├── pom.xml
│   └── src/main/
│       ├── java/com/rms/
│       │   ├── RmsApplication.java          ← entry point
│       │   ├── config/
│       │   │   ├── SecurityConfig.java      ← disables default auth, provides BCrypt
│       │   │   ├── CorsConfig.java          ← allows localhost:3000
│       │   │   └── GlobalExceptionHandler.java ← clean JSON error responses
│       │   ├── entity/
│       │   │   ├── Applicant.java
│       │   │   ├── Recruiter.java
│       │   │   ├── Job.java
│       │   │   └── Application.java
│       │   ├── repository/
│       │   │   ├── ApplicantRepository.java
│       │   │   ├── RecruiterRepository.java
│       │   │   ├── JobRepository.java
│       │   │   └── ApplicationRepository.java
│       │   ├── service/
│       │   │   ├── AuthService.java
│       │   │   ├── JobService.java
│       │   │   ├── ApplicationService.java
│       │   │   └── ProfileService.java
│       │   ├── controller/
│       │   │   ├── AuthController.java
│       │   │   ├── JobController.java
│       │   │   ├── ApplicationController.java
│       │   │   └── ProfileController.java
│       │   └── dto/
│       │       ├── AuthDTOs.java
│       │       ├── JobDTOs.java
│       │       ├── ApplicationDTOs.java
│       │       └── ProfileDTOs.java
│       └── resources/
│           └── application.properties
│
└── frontend/                              ← same design, updated API calls
    ├── .env.local                         ← NEXT_PUBLIC_API_URL=http://localhost:8080/api
    ├── lib/
    │   ├── axios.ts                       ← points to localhost:8080, no JWT
    │   └── auth.ts                        ← uses localStorage instead of cookies
    └── app/auth/
        ├── login/page.tsx
        └── register/page.tsx
```

---

## Key Differences from NestJS Version

| Feature           | NestJS Version           | Java Version               |
|-------------------|--------------------------|----------------------------|
| Auth              | JWT tokens in cookies    | User ID in localStorage    |
| Password hashing  | bcrypt (12 rounds)       | BCrypt (12 rounds)         |
| Protected routes  | JWT guard + role guard   | ID passed as query param   |
| Database          | PostgreSQL (Supabase)    | MySQL (local)              |
| File upload       | Cloudinary               | Not implemented            |
| Admin dashboard   | Yes                      | Removed                    |
| Architecture      | 3-layer                  | 2-layer + service where needed |
