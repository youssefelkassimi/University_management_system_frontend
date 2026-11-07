# University Management System — Frontend

A React single-page application for managing a university: academic programs, departments, teachers, modules, enrollments, academic years, student records, grades, and student documents.

The frontend consumes a REST API (Spring Boot backend expected at `http://localhost:8080`) and uses JWT-based authentication with role-specific dashboards.

## Table of contents

- [Features](#features)
- [Tech stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting started](#getting-started)
- [Available scripts](#available-scripts)
- [Roles and routes](#roles-and-routes)
- [Authentication](#authentication)
- [API integration](#api-integration)
- [Project structure](#project-structure)
- [Building for production](#building-for-production)
- [Troubleshooting](#troubleshooting)

## Features

### Administrator (`ROLE_ADMIN`)

- Dashboard and administrator profile.
- Manage academic programs (filières), including activation and deactivation.
- Manage departments and teachers.
- Manage modules, academic years, and enrollments.
- Set the current academic year.
- View and add students.
- Notes management section (placeholder for school-wide grade administration).

### Teacher (`ROLE_TEACHER`)

- View and edit profile, including profile photo upload.
- View assigned modules.
- Enter and edit module grades by session (`NORMAL` / retake session).
- View grade statistics for a module.
- Search and manage individual student notes (edit and delete).

### Head of Program (`ROLE_CHEF_FILIERE`)

- View and edit profile.
- View the program(s) they manage.
- View program modules and their statistics.
- Manage student notes for their modules.
- Browse students enrolled in their program with pagination.

### Student (`ROLE_STUDENT`)

- View and edit personal profile.
- View grades.
- Download a school certificate (PDF).
- Download an academic transcript (bulletin, PDF).

## Tech stack

| Concern | Technology |
| --- | --- |
| UI framework | React 18 |
| Build tool | Vite 4 |
| Routing | React Router DOM 6 |
| HTTP client | Axios |
| UI components | React Bootstrap 5, Bootstrap 5, MUI (Material UI) |
| Icons | MUI Icons, React Icons |
| Charts | Chart.js, React Chart.js 2 |
| Styling | Bootstrap CSS, Emotion, component CSS |

## Prerequisites

- [Node.js](https://nodejs.org/) 18 or newer
- npm (included with Node.js)
- A running backend API at `http://localhost:8080` (the frontend has no mock server)

## Getting started

1. Clone the repository and install dependencies:

   ```bash
   git clone <repository-url>
   cd University_management_system_frontend
   npm install
   ```

2. Start the backend API so it is reachable at `http://localhost:8080`.

3. Start the development server:

   ```bash
   npm run dev
   ```

4. Open the URL printed by Vite (usually `http://localhost:5173`). The root route redirects to `/login`.

5. Log in with an account created in the backend. After authentication you are redirected to the dashboard for your role.

## Available scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Vite development server with hot reload. |
| `npm run build` | Create a production build in `dist/`. |
| `npm run preview` | Serve the production build locally for verification. |
| `npm run lint` | Run ESLint over `.js` and `.jsx` files (requires an ESLint configuration). |

> There is currently no automated test suite configured.

## Roles and routes

| Role | Base area | Main routes |
| --- | --- | --- |
| Public | `/login` | `/login` |
| Administrator | `/admin` | `/admin/profile`, `/admin/dashboard`, `/admin/filiere`, `/admin/departement`, `/admin/enseignant`, `/admin/module`, `/admin/inscription`, `/admin/annee-universitaire`, `/admin/etudiants`, `/admin/notes` |
| Teacher | `/teacher` | `/teacher/profile`, `/teacher/modules`, `/teacher/modules/:moduleId/notes`, `/teacher/individual-notes` |
| Head of program | `/chef-filiere` | `/chef-filiere/profile`, `/chef-filiere/filiere`, `/chef-filiere/modules`, `/chef-filiere/modules/:moduleId/notes`, `/chef-filiere/notes`, `/chef-filiere/students` |
| Student | `/student` | `/student/profile`, `/student/notes`, `/student/documents` |

Unknown routes redirect to `/`, which in turn redirects to `/login`.

Routes are guarded by `PrivateRoute`, which checks for an access token and, when required, the user's role.

## Authentication

- Login is performed against `POST http://localhost:8080/api/auth/login`.
- The backend returns `accessToken`, `refreshToken`, `userId`, and `userRole`.
- These values are stored in `localStorage` and restored on page reload by `AuthContext`.
- API requests send the token in the `Authorization: Bearer <accessToken>` header.
- Logout clears the stored tokens and user information and returns to `/login`.

`AuthContext` exposes:

```js
const { user, accessToken, refreshToken, login, logout, loading } = useAuth();
```

## API integration

The base URL is currently hard-coded to `http://localhost:8080` across the codebase. Key endpoints used by the frontend include:

| Area | Endpoints |
| --- | --- |
| Authentication | `POST /api/auth/login` |
| Academic years | `/api/annees-universitaires`, `/api/annees-universitaires/{id}`, `/api/annees-universitaires/{id}/set-current` |
| Departments | `/api/departements`, `/api/departements/add`, `/api/departements/{id}` |
| Programs | `/api/filieres`, `/api/filieres/active`, `/api/filieres/{id}`, `/api/filieres/{id}/activate`, `/api/filieres/{id}/deactivate`, `/api/filieres/{enseignantId}` |
| Teachers | `/api/enseignants`, `/api/enseignants/{id}`, `/api/enseignants/user/{userId}`, `/api/enseignants/{id}/photo`, `/api/admin/teacher` |
| Modules | `/api/modules`, `/api/modules/{id}`, `/api/modules/enseignant/{enseignantId}`, `/api/modules/filiere/{filiereId}` |
| Enrollments | `/api/inscriptions`, `/api/inscriptions/{id}` |
| Students | `/api/students`, `/api/students/{userId}`, `/api/students/{codeApj}`, `/api/students/filiere/{filiereId}`, `/api/students/module/{moduleId}`, `/api/admin/student` |
| Grades | `/api/notes/insert`, `/api/notes/{id}`, `/api/notes/student/{studentId}/notes`, `/api/notes/student/{studentId}/module/{moduleId}`, `/api/notes/module/{moduleId}/statistics` |
| Documents | `/api/students/{codeApj}/certificate`, `/api/students/{studentId}/bulletin` |

All protected requests require a valid JWT access token.

To point the app at a different backend, replace the hard-coded URLs (for example with an environment variable such as `VITE_API_URL`) before building. The current code does not read environment variables.

## Project structure

```text
.
├── index.html
├── package.json
├── package-lock.json
└── src/
    ├── App.jsx                   # Route definitions
    ├── main.jsx                  # Application entry point
    ├── index.css                 # Global styles
    ├── assets/
    │   └── logo.png              # Login logo
    ├── components/
    │   ├── AdminLayout.jsx       # Admin navigation shell
    │   ├── ChefFiliereLayout.jsx # Head of program navigation shell
    │   ├── TeacherLayout.jsx     # Teacher navigation shell
    │   └── PrivateRoute.jsx      # Authentication and role guard
    ├── context/
    │   └── AuthContext.jsx       # Login, logout, and session state
    ├── layouts/
    │   ├── AdminLayout.jsx
    │   └── StudentLayout.jsx     # Student navigation shell
    ├── pages/
    │   ├── Login.jsx             # Public login page
    │   ├── Admin*.jsx            # Admin profile, dashboard, and student management
    │   ├── Filiere.jsx           # Program management
    │   ├── Departement.jsx       # Department management
    │   ├── Enseignant.jsx        # Teacher management
    │   ├── Module.jsx            # Module management
    │   ├── Inscription.jsx       # Enrollment management
    │   ├── AnneeUniversitaire.jsx# Academic year management
    │   ├── Notes.jsx             # Admin notes page
    │   ├── Teacher*.jsx          # Teacher profile, modules, and notes
    │   ├── ChefFiliere*.jsx      # Head of program pages
    │   └── Student*.jsx          # Student profile, notes, and documents
    └── services/
        ├── studentService.js     # Student-related API calls
        └── teacherService.js     # Teacher-related API calls
```

## Building for production

```bash
npm run build
```

The output is written to `dist/`. Serve it with any static web server. Because the app uses client-side routing (`BrowserRouter`), configure the server to fall back to `index.html` for unknown paths.

To verify the production build locally:

```bash
npm run preview
```

## Troubleshooting

- **Blank page or network errors after login** — make sure the backend is running at `http://localhost:8080` and that CORS allows requests from the frontend origin.
- **Immediate redirect back to `/login`** — the access token may be missing or expired. Log in again; tokens are stored in `localStorage`.
- **Redirected to `/` after login** — the stored `userRole` did not match the route's allowed roles.
- **PDF download fails** — the certificate and bulletin endpoints must return a PDF blob and accept the JWT in the `Authorization` header.
- **`npm run lint` fails** — the repository does not currently include an ESLint configuration file.
