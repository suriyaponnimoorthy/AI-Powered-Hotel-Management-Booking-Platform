> **Payments and private KYC:** Razorpay checkout/refunds and encrypted identity documents are now implemented. Read [SECURITY_SETUP.md](SECURITY_SETUP.md) before enabling them. Configuration and activation requirements are documented there.

> **Deployment:** Start with [DEPLOYMENT.md](DEPLOYMENT.md) for the tested Render setup, account creation, and feature configuration.

# ABC Hotels — AI-Powered Hotel Management & Booking Platform

A full-stack, dynamic hotel management and booking web application built with
**Python, Django, Django REST Framework**, and a real **AI hotel concierge**.
Not a static site or a frontend mock — every room, price, booking, and review
you see comes from the database, and every price shown to a guest is
recalculated and validated on the server before a booking is created.

---

## 1. Project Overview

ABC Hotels lets guests search and filter rooms by dates/guests/price/amenities,
book rooms with a real (overlap-based) availability engine, pay through a
Razorpay hosted checkout (or an explicit local demo gateway), manage their bookings/reviews/notifications
from a dashboard, and chat with an **AI Concierge** that recommends rooms
using live hotel data (never hallucinated prices or availability).

Staff/admins get a separate analytics dashboard (Chart.js) plus room, booking,
customer, and offer management, in addition to the fully customized Django
Admin.

## 2. Features

- Dynamic room browsing, filtering, and **real date-overlap availability search**
- Full booking workflow: search → select → dates → services → price → pay → confirm
- Server-side price calculation & validation (frontend numbers are never trusted)
- Promo codes / offers applied and validated server-side
- Razorpay hosted checkout, verified capture webhooks, full refunds and reconciliation
- Encrypted private identity uploads, staff review permissions, audit logs and retention cleanup
- Customer dashboard: bookings, wishlist (DB-backed, per-user), notifications,
  profile, **digital check-in**
- Reviews system (only after a completed stay)
- **AI Hotel Concierge** — natural-language chat + structured recommendation
  endpoint, grounded in real DB data (rooms, policies, services, attractions),
  with **persisted, resumable conversation history** for logged-in guests
- **✨ Plan My Stay** — AI builds a day-by-day itinerary + estimated cost from
  real rooms/services/attractions only
- **"Why this room?"** transparency panel on room detail pages
- **Room comparison** (`/compare/`) — side-by-side feature table for up to 3 rooms
- **Hotel photo gallery** (`/hotel-gallery/`) with lightbox, plus a full
  room-image gallery system (primary image, ordering, alt text)
- Personalized homepage for returning users (upcoming booking, welcome-back banner)
- Forgot/reset password flow
- Custom admin analytics dashboard with live charts (revenue, bookings,
  occupancy, room-type popularity, ratings) + Django Admin for full CRUD
- **Django Groups & Permissions** — a "Hotel Staff" group with scoped
  view/change permissions vs. a "Hotel Admin" group with full access
- REST API with JWT auth, OpenAPI docs (`/api/docs/`)
- Seed data management command for realistic demo data
- **Classic Teal design system** — a restrained, human-designed visual
  language: a four-color palette (`#FBFCFC` / `#0F3D3E` / `#14746F` /
  `#8EB69B`), Manrope for UI text with Cormorant Garamond reserved for
  hospitality headings, a fixed spacing scale (4–80px) and radius scale
  (4–12px), one subtle shadow level, and compact dashboards with no
  oversized KPI tiles

## 3. Technology Stack

- Python 3.12, Django 6.1.3, Django REST Framework
- SQLite by default (zero config) — PostgreSQL supported via `.env`
- django-filter, djangorestframework-simplejwt, django-cors-headers, drf-spectacular
- HTML5, CSS3 (custom, no Bootstrap), vanilla JavaScript (Fetch API/AJAX)
- Chart.js for admin analytics
- Anthropic API for the AI concierge (provider-swappable; offline mock mode
  works out of the box with no API key)

## 4. Project Structure

```
abc_hotels/
├── config/                # settings, urls, wsgi/asgi
├── accounts/               # custom User model, auth (web + JWT API), password reset
├── hotels/                 # RoomType, Room, RoomImage, HotelImage, Amenity, Wishlist,
│                           # SpecialRequest, DigitalCheckIn, policies, attractions, seed_data
├── bookings/                # Booking, BookingRoom, BookingService, availability + pricing logic,
│                           # digital check-in & special-request API endpoints
├── payments/                # Payment model + pluggable gateway (mock/Razorpay/Stripe)
├── reviews/                  # Review model + API
├── offers/                   # Offer/promo code model + API
├── notifications/            # Notification model + API
├── ai_assistant/              # AI Concierge: prompts.py, services.py, views.py,
│                             # AIConversation/AIMessage history, Plan My Stay
├── dashboard/                  # Customer + Admin analytics dashboards
├── templates/                    # Django templates (home, rooms, gallery, compare,
│                                 # plan-my-stay, bookings, dashboard, auth...)
├── static/                        # css/js
├── requirements.txt
├── .env.example
└── manage.py
```

## 5. Database Architecture (high level)

`User` (role: CUSTOMER/STAFF/ADMIN) →
`Booking` → `BookingRoom` (→ `Room` → `RoomType`, `Amenity`) and
`BookingService` (→ `HotelService`) →
`Payment`; plus `Review`, `Offer`, `Notification`, `HotelPolicy`,
`NearbyAttraction`, `ContactMessage`.

Availability is **never** a simple boolean — `bookings/services.py` computes
it from date-range overlaps against bookings in an active status
(PENDING/CONFIRMED/CHECKED_IN), combined with the room's own status.

## 6. Installation

```bash
git clone <repository-url>
cd abc_hotels

python -m venv venv
venv\Scripts\activate            # Windows
source venv/bin/activate          # Linux/macOS

pip install -r requirements.txt

cp .env.example .env               # then edit values as needed

python manage.py migrate
python manage.py seed_data --allow-demo-data
python manage.py createsuperuser

python manage.py runserver
```

Visit `http://127.0.0.1:8000/`. The seeder is restricted to local debug databases and creates demo accounts with disabled passwords. Register a separate test customer or create a superuser for administration.

### Enabling PostgreSQL

Set `USE_POSTGRES=True` in `.env` and fill in `DATABASE_NAME/USER/PASSWORD/HOST/PORT`,
then re-run `migrate`.

### Enabling the live AI Concierge

By default (no `AI_API_KEY`), the concierge runs in an offline demo mode that
still parses guest intent and returns real matching rooms — it just skips the
LLM call. Set `AI_API_KEY` and `AI_PROVIDER=anthropic` in `.env` to enable
live natural-language responses grounded in your hotel's data.

## 7. Running Tests

```bash
python manage.py test
```

52 tests cover registration/login, availability logic, server-side price
calculation, double-booking prevention, promo codes, booking cancellation/
payment, review eligibility, offer validity, notification scoping, AI
concierge intent parsing, wishlist scoping, digital check-in, special
requests, Plan My Stay, AI conversation history, and the gallery/compare pages.

## 8. API Documentation

- Schema: `/api/schema/`
- Swagger UI: `/api/docs/`

Key endpoints: `/api/auth/register|login/`, `/api/rooms/`, `/api/rooms/search/`,
`/api/bookings/` (+ `cancel/`, `pay/`, `check_in/`, `check_out/` actions),
`/api/reviews/`, `/api/offers/`, `/api/notifications/`, `/api/wishlist/`,
`/api/check-in/`, `/api/special-requests/`, `/api/ai/chat/`,
`/api/ai/recommend/`, `/api/ai/plan-stay/`, `/api/ai/conversations/`.

## 9. Security Notes

CSRF protection, session + JWT auth, password hashing via Django's built-in
hashers, environment-variable secrets (`SECRET_KEY`, `AI_API_KEY`,
`DATABASE_PASSWORD`, `PAYMENT_API_KEY` are never committed), and all pricing/
availability decisions are re-validated server-side regardless of what the
client sends.

## 10. What's Deliberately Simplified for This Build

This is a complete, working full-stack implementation, but a few areas are
intentionally left as clearly-marked extension points rather than production
integrations:
- **Payments**: the development `MockGateway` always succeeds; `payments/services.py`
  is restricted to explicit demo use. Real Razorpay checkout, verified webhooks and full refunds are implemented in `payments/razorpay_services.py`; Stripe is not implemented.
- **PDF invoices**: the confirmation page is print/PDF-friendly via the
  browser's print dialog; wiring up a server-generated PDF (e.g. WeasyPrint)
  is a drop-in addition to `bookings/views.py`.
- **Hotel/room photos**: `HotelImage` and `RoomImage` are fully modeled
  (primary image, ordering, alt text, 5MB validation) and manageable from
  Django Admin, but no sample image *files* ship with the seed data — the
  gallery pages show a graceful empty state until an admin uploads real
  photos.
- **Identity verification**: private documents can be uploaded and reviewed by authorized hotel staff. Review is manual; automated government/identity-provider validation is not included. The existing digital check-in flow is separate.

## Reviewed AI update (31 October 2026)

See `PROJECT_REVIEW.md` for the prioritized drawbacks, remaining production blockers, setup, limitations and AI roadmap. Open `/recommendations/` for explainable room matching. The concierge now supports hotel-record offline answers, date-aware suggestions, bounded follow-ups, owner-scoped saved history and provider-error fallback. The content-based ranking does not require an API key; live language-model chat requires Anthropic configuration. Budget is per room per night, before taxes and extras. 68 automated tests pass with the pinned dependencies.


## Booking safety update (31 October 2026)

Booking creation now uses shared atomic rules for HTML and REST, with ordered inventory locks and a second availability/price check at save time. Date/capacity validation and repeat demo-payment protection are included. General booking edits/deletion and Django admin booking mutations are disabled. Pending holds expire after 15 minutes by default; run `python manage.py expire_booking_holds` periodically to update their displayed status and send an in-app notification. Availability queries release expired holds even without cleanup. See PROJECT_REVIEW.md for rollout implications for existing pending records, configuration, remaining limitations and PostgreSQL test instructions.

Current validation: all 146 tests passed on local PostgreSQL 16.2, including both concurrency tests. New payment/KYC migrations are included. Provider credentials, private storage, scanning and live service verification are still required. See SECURITY_SETUP.md.

## Premium interface update

The interface now uses a warm, minimal editorial design with full-width photography, serif headings, image-led room cards and a simplified navigation menu. See UI_DESIGN_NOTES.md for design files, image sources, customization and verification. Existing booking, AI recommendations, concierge, account and dashboard routes are retained.
