# Nile

A full-stack social media web application — users can register, sign in, write and edit posts, comment, like, follow other users, bookmark posts, and receive notifications. Built with React on the frontend and Node.js/Express + SQLite on the backend.

## Tech Stack

**Frontend** (`nile-frontend/`)
- React 19 with React Router v7 (`BrowserRouter`, lazy-loaded routes via `React.Suspense`)
- React Bootstrap + Bootstrap 5 for UI
- Formik + Yup for form handling and validation
- Axios for HTTP requests

**Backend** (`nile-backend/`)
- Express 4 REST API
- SQLite via `better-sqlite3` (WAL mode, foreign keys enabled)
- JSON Web Tokens (`jsonwebtoken`) for authentication, `bcryptjs` for password hashing
- Jest + Supertest for API tests, `nodemon` for local dev reloads

**End-to-end tests**
- Playwright (`nile-e2e.js` at the project root)

## Project Structure

```
nile/
├── nile-backend/         Express API server, SQLite database, Jest tests
│   ├── routes/           Route definitions (auth, posts, users, bookmarks, notifications)
│   ├── controllers/      Request handlers / business logic
│   ├── middleware/       Auth middleware (JWT verification)
│   ├── db.js             Database connection + schema setup
│   └── __tests__/        Jest + Supertest API test suites
├── nile-frontend/        React single-page application
│   └── src/components/   Pages and UI components (Feed, Post, Profile, Navbar, ...)
├── nile-e2e.js           Playwright end-to-end browser test scenarios
└── docs/                 Project documentation (graduation project book, etc.)
```

## Getting Started

### Prerequisites
- Node.js and npm installed

### Backend setup

```bash
cd nile-backend
npm install
cp .env.example .env   # then edit .env and set your own JWT_SECRET
node server.js
```
The API runs on http://localhost:5000. For development with auto-reload, use `npm run dev` instead.

### Frontend setup

```bash
cd nile-frontend
npm install
npm start
```
The app runs on http://localhost:3000 and expects the backend to be running on port 5000.

## API Reference

All endpoints below are mounted under `/api` and (unless noted) require a valid JWT in the `Authorization` header, set via the auth middleware (`requireAuth`).

**Auth** — `/api/auth`
- `POST /register` — create a new account
- `POST /login` — authenticate and receive a JWT

**Posts** — `/api/posts` *(requires auth)*
- `GET /` — list posts (paginated)
- `POST /` — create a post
- `PUT /:id` — edit a post
- `DELETE /:id` — delete a post
- `POST /:id/comments` — add a comment to a post
- `DELETE /:id/comments/:cid` — delete a comment
- `POST /:id/like` — like / unlike a post
- `POST /:id/bookmark` — bookmark / unbookmark a post

**Users** — `/api/users` *(requires auth)*
- `GET /:id` — view a user's public profile
- `PUT /:id` — update profile (bio, name, password fields)
- `POST /:id/follow` — follow / unfollow a user
- `PUT /:id/password` — change password

**Bookmarks** — `/api/bookmarks` *(requires auth)*
- `GET /` — list the current user's bookmarked posts

**Notifications** — `/api/notifications` *(requires auth)*
- `GET /` — list notifications for the current user
- `PUT /read` — mark notifications as read

## Database Schema

SQLite database stored at `nile-backend/db/nile.db` (WAL mode, foreign keys enforced):

- **users** — id, username, email, password (hashed), first_name, last_name, bio, created_at
- **posts** — id, user_id, title, content, image_url, tags, created_at, edited_at
- **comments** — id, post_id, user_id, content, created_at
- **likes** — id, post_id, user_id (unique per post/user pair)
- **follows** — id, follower_id, following_id (unique per follower/following pair)
- **bookmarks** — id, user_id, post_id (unique per user/post pair)
- **notifications** — id, user_id, actor_id, type, post_id, is_read, created_at

## Frontend Pages

Routed in `App.js`, with a persistent `Navbar`/`Footer` and route protection via `ProtectedRoute`:

| Route | Component | Description |
|---|---|---|
| `/` | `Home` | Landing page |
| `/sign-in` | `SignIn` | Login form |
| `/register` | `Register` | Account creation form |
| `/feed` | `Feed` | Main feed — search, tag filter, pagination *(protected)* |
| `/profile` | `Profile` | Own profile — edit bio and change password *(protected)* |
| `/users/:id` | `PublicProfile` | View another user's profile and follow/unfollow *(protected)* |
| `/bookmarks` | `Bookmarks` | List of bookmarked posts *(protected)* |

Posts support images, tags, likes, comments, bookmarking, and inline editing (see `Post.js` / `PostForm.js`).

## Running Tests

**Backend API tests** (Jest + Supertest, runs against an in-memory SQLite database):
```bash
cd nile-backend
npm test
```

**End-to-end browser tests** (Playwright — requires both the backend and frontend running locally):
```bash
node nile-e2e.js
```

## Documentation

Project write-ups, including the full graduation project book, live in [`docs/`](docs/).
