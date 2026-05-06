# KOTS Project Walkthrough For Better Application

Use this as your 8-10 minute Loom script. Keep the project open in VS Code and move through the files in this order. The goal is to show that you can ship, explain decisions, care about code quality, and use AI with judgment.

Job signal to emphasize:
- Builder mindset: this is a working rental/property workflow, not just a UI demo.
- Python strength: modular Flask backend, service layer, models, auth, migrations, deployment.
- Frontend awareness: the frontend is Angular, but the same API/state/component thinking transfers to React.
- Judgment with AI: AI helped speed up review and edge-case thinking, but final choices were based on code, database behavior, and product requirements.

## 0:00-0:45 - Quick Intro

Show:
- `KOTS/flask/README.md`
- Optional live app/API links from the README

Say:

Hi, I am Jay. This project is KOTS, a rental and property booking platform. I chose this project because it shows how I structure a real full-stack product: users can discover buildings, towers, flats, amenities, and book flats; admins manage inventory and approve bookings; master users can create and supervise admins.

For this walkthrough I will focus mainly on the Python backend because it shows the strongest architecture decisions. The frontend here is Angular, but the patterns around API separation, typed data, state, and route-based pages are transferable to React.

Recruiter signal:
- You understand the product, not only the code.
- You immediately connect the project to the job requirement.

## 0:45-2:00 - Backend Architecture

Show:
- `KOTS/flask/app.py`
- `KOTS/flask/common/`
- `KOTS/flask/users/`
- `KOTS/flask/admins/`
- `KOTS/flask/master/`

Say:

The backend is a modular Flask REST API. I split it by role and domain: `users`, `admins`, `master`, and `common`.

In `app.py`, the app factory wires together configuration, CORS, SQLAlchemy, migrations, JWT, Cloudinary, blueprints, error handlers, and response caching. My main decision was to keep routes thin and move business logic into services.

So the structure is:
- routes handle HTTP input and call a service
- services contain business logic and database operations
- schemas validate payloads and serialize responses
- models define the database shape
- common contains shared behavior like permissions, response format, caching, and error handling

This makes the project easier to extend because adding a new role or feature does not require mixing all logic into one large file.

Recruiter signal:
- You know backend architecture.
- You can explain why the folders exist.
- You care about maintainability.

## 2:00-3:15 - Database Model And Relationships

Show:
- `KOTS/flask/admins/models_admins.py`

Point at:
- `Building`
- `Tower`
- `Flat`
- `FlatPicture`
- `Amenity`
- `Booking`
- `flat_amenities`

Say:

This is the main data model. A building has towers, a tower has flats, flats can have amenities, flat pictures, and bookings. I used SQLAlchemy relationships and cascades so related data is managed consistently.

There are also database-level constraints. For example, `Flat` has a unique constraint on `tower_id` and `flat_number`, so the same flat number cannot accidentally be created twice in one tower. `FlatPicture` also has a unique constraint on `flat_id` and `room_name`, so there cannot be duplicate room images for the same flat.

I like putting important business rules close to the database when possible because it protects the system even if a future API path forgets a validation check.

Recruiter signal:
- You think about data integrity.
- You understand ORM relationships and constraints.
- You can reason beyond simple CRUD.

## 3:15-4:15 - Auth, Roles, And Protected Routes

Show:
- `KOTS/flask/common/permissions.py`
- `KOTS/flask/users/models_users.py`
- `KOTS/flask/app.py`, JWT blocklist section

Say:

KOTS uses JWT authentication with three roles: user, admin, and master.

The `role_required` decorator centralizes authorization. Instead of copying permission logic into every admin route, routes can simply require `admin` or `master`. The decorator resolves the user from the JWT identity, checks whether the account is master, admin, or user, and returns a proper 401 or 403 response when access is invalid.

I also added token revocation for logout. That means logout is not just clearing frontend state. The token JTI is stored and checked by the JWT blocklist loader in `app.py`.

Recruiter signal:
- You know authentication and authorization are different.
- You avoid repeated security logic.
- You consider logout/session behavior.

## 4:15-5:45 - Business Rule Example: Booking Approval

Show:
- `KOTS/flask/admins/services_admins.py`
- Function: `update_admin_booking_status_service`

Search in file:
- `def update_admin_booking_status_service`
- `status == "APPROVED"`
- `flat.is_available = False`

Say:

This is one of my favorite examples because it is where the project becomes more than CRUD.

When an admin approves a booking, the system also marks the flat as unavailable. It blocks approving a booking that is already approved, and it checks whether the flat already has another approved booking.

Without this, two users could potentially get approved for the same flat. The UI might still look fine, but the business workflow would be wrong. So this is the kind of edge case I tried to protect in the service layer.

I handled this in the backend because availability should not depend only on frontend state. The source of truth should be the database and backend business logic.

Recruiter signal:
- You understand product correctness.
- You can explain edge cases.
- You protect critical workflows server-side.

## 5:45-6:45 - Search And Performance

Show:
- `KOTS/flask/users/services_users.py`
- `KOTS/flask/migrations/versions/c1d4f5a9b2e1_enable_pg_trgm_and_search_indexes.py`
- `KOTS/flask/config.py`

Search in file:
- `similarity`
- `word_similarity`
- `status_counts`
- `ADDRESS_TRIGRAM_MIN_SIMILARITY`

Say:

Search is important for a rental platform because users often search by address, city, building name, rent range, and flat type.

I improved search using PostgreSQL trigram search with `pg_trgm`. This gives typo-tolerant matching and ranked results instead of only basic string filters.

The tuning values are environment-driven in `config.py`, and the database migration enables `pg_trgm` and adds indexes. I also return pagination metadata and status counts so the frontend can show available and unavailable results consistently.

Recruiter signal:
- You think about user experience and performance.
- You know when database features are better than ad hoc Python filtering.
- You made the behavior configurable.

## 6:45-7:45 - Images, Caching, And Production Thinking

Show:
- `KOTS/flask/common/image_compression.py`
- `KOTS/flask/common/cache.py`
- `KOTS/flask/Dockerfile`
- `KOTS/flask/Procfile`

Say:

Because this is a property platform, images matter. I added image compression before Cloudinary upload to reduce upload size and improve load time.

In `common/cache.py`, GET responses that include image URLs get cache headers and ETags. Public discovery APIs can be cached publicly for a short time, while private responses vary by Authorization.

This is also deployed with production files like Dockerfile, Procfile, Gunicorn config, and Cloud Run/Render deployment notes. So I was not only building locally; I was thinking about how it runs after deployment.

Recruiter signal:
- You care about production behavior.
- You know images can affect performance.
- You have deployment experience.

## 7:45-8:45 - Frontend Integration

Show:
- `KOTS/angular/kots_frontend/src/app/app.routes.ts`
- `KOTS/angular/kots_frontend/src/app/users_users/api_users_auth.ts`
- `KOTS/angular/kots_frontend/src/app/admins_admins/api_admins.ts`
- `KOTS/angular/kots_frontend/src/app/shared/api_error_message.ts`
- `KOTS/angular/kots_frontend/src/app/route_prefetch_resolvers.ts`

Say:

The frontend is Angular, but I organized it in a way that maps well to React. Pages are route-based, API calls are separated into API files, response types live separately, and shared utilities handle things like error messages and loading behavior.

For example, `api_users_auth.ts` and `api_admins.ts` keep HTTP details out of page components. `api_error_message.ts` converts backend failures into user-friendly messages. Route prefetch resolvers start loading key data before the page fully renders.

If I were building this in React, I would use similar boundaries with React Router, typed API clients, hooks, and shared state.

Recruiter signal:
- You can discuss frontend structure even though this project is Angular.
- You understand separation between UI and API logic.
- You can connect your experience to their Python + React stack.

## 8:45-9:30 - Commits And Iteration

Show terminal in backend repo:

```bash
cd KOTS/flask
git log --oneline -n 10
```

Show terminal in frontend repo:

```bash
cd KOTS/angular/kots_frontend
git log --oneline -n 10
```

Good backend commits to mention:
- `826d253` - added flat image model and APIs
- `86b97f3` - increased flat image limit from 6 to 18
- `3c10441` - hardened booking approval and conflict handling
- `d3f0123` / `a8c3c9e` - Dockerized backend

Good frontend commits to mention:
- `3ce2e4b` - flat detail page and UI fixes
- `9984de6` - CSS budget fixes
- `f3af520` - optimized Dockerfile for deployment
- `8320ca5` - global header update

Say:

I used commits to move in stages instead of treating the project as one big dump. You can see the project evolving from first implementation, to Dockerization, to image APIs, booking conflict fixes, UI fixes, deployment, and documentation updates.

Recruiter signal:
- You can talk through iteration.
- You understand commits as a history of decisions.
- You are not only showing finished code.

## 9:30-10:00 - AI Usage And Closing

Show:
- Any service file where you made a careful decision, such as `admins/services_admins.py`
- Optional: README update history

Say:

I use AI as a coding partner, mainly to explore alternatives, catch edge cases, and speed up repetitive implementation. But I do not blindly accept generated code. For example, in the booking approval and search work, I checked the actual models, database behavior, API responses, and migrations before finalizing.

What I am proud of in KOTS is that it is not just a screen demo. It has real backend architecture, role-based access, database migrations, image handling, search, deployment files, and business-rule protection.

That matches what I liked in Better's job post: builders who ship, care about craft, and use AI well while still relying on judgment.

Recruiter signal:
- You directly answer their AI requirement.
- You sound thoughtful, not tool-dependent.
- You end with confidence and relevance.

## Short 5-Minute Version

If you need to cut this down to the exact Loom requirement, use this version:

1. 0:00-0:30 - Product intro and why this project.
2. 0:30-1:20 - Flask architecture in `app.py` and module folders.
3. 1:20-2:10 - Database model in `admins/models_admins.py`.
4. 2:10-3:00 - Auth and role guard in `common/permissions.py`.
5. 3:00-3:50 - Booking approval business rule in `admins/services_admins.py`.
6. 3:50-4:30 - Search, images, caching, and deployment.
7. 4:30-5:00 - Commits, AI usage, and closing.

## Recording Tips

- Keep VS Code zoomed in enough that code is readable.
- Start with the product and workflow before diving into files.
- Do not read every line. Point to the important block and explain the decision.
- Say "why" more than "what".
- Avoid spending too much time on Angular because the job asks for React. Use it to show frontend thinking, then return to Python strength.
- Mention exact business risks: duplicate bookings, unauthorized access, slow image-heavy pages, weak search relevance.
- End by connecting back to Better's wording: builder, craft, AI with judgment.

