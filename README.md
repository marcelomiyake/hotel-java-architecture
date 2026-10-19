# Wayfarer Stays

Wayfarer is a hotel reservation system based on the [ByteByteGo hotel reservation system case study](https://bytebytego.com/courses/system-design-interview/hotel-reservation-system). It supports city search, hotel and room details, date based availability, reservation creation, reservation history, and cancellation.

![Wayfarer Stays landing page captured by the Lighthouse mobile audit](docs/wayfarer-home-lighthouse.jpg)

*Landing page screenshot from the Lighthouse mobile audit.*

## Case study and known defects

This is the **Java architecture** implementation in [One Hotel Design, Five AI Implementations](https://marcelomiyake.com.br/posts/five-ai-hotel-implementations/), which compares five AI-built versions of the same hotel design, and in its [Part II on the cost of a new feature](https://marcelomiyake.com.br/posts/five-ai-hotel-implementations-part-2/). Part I inspected commit [`47b0c3b`](https://github.com/marcelomiyake/hotel-java-architecture/commit/47b0c3b79a96ba94c433e54639bfe9cdc5be1541).

The study found the clearest dependency direction of the five here, and its domain checks the guest count against room capacity. It also found:

- Two overlapping requests cancelling the same reservation both succeeded and both released inventory: with another booking still active, the probe ended with seven of seven rooms available instead of six. The reservation's status change is not serialized, and the test `cancellationChecksGuestIdentityAndRestoresEveryNightOnce` covers only sequential calls.
- Reservation lookup and cancellation compare a supplied email address with the stored one; that is not authentication.
- Part II: checkout offers pay at the property only, so abandonment analytics treats a browser confirmation event as completion. The event is not tied to a stored reservation: a `RESERVATION_CONFIRMED` event removed an attempt from the abandonment view without creating a reservation.

No later commit fixes these. The passing tests and SonarQube Cloud results below do not cover them. This is a local learning implementation, not a production system.

## Product scope

- Search the hotel catalog by city and inspect hotel details and room types.
- Check nightly availability for a date range, guest count, and room count.
- Create a reservation with a client supplied idempotency key.
- Find and cancel reservations using the guest email address.
- Browse a responsive editorial landing page, hotel collection, checkout, confirmation, and trip history.

Reservation commands run transactionally. PostgreSQL advisory locks serialize requests with the same idempotency key, and inventory rows are locked by stay date before availability changes. The catalog and reservation services use separate PostgreSQL schemas in one local PostgreSQL instance.

## Architecture

The backend is Java 25 with Spring Boot. The web application is React 19.3 and TypeScript. PostgreSQL migrations and the Kubernetes manifests are in the same monorepo.

| Area | Responsibilities |
| --- | --- |
| `services/catalog-service` | Hotel search and hotel detail queries; catalog domain and PostgreSQL adapter |
| `services/reservation-service` | Availability queries, reservation and cancellation commands, inventory and reservation domains, PostgreSQL adapters |
| `web/src/domain` | Browser side hotel, reservation, and stay rules and models |
| `web/src/application` | Search and reservation queries/commands expressed against ports |
| `web/src/infrastructure` | HTTP adapters for catalog and reservation APIs |
| `web/src/presentation` | React screens and components; reservation screens are lazy loaded |
| `k8s` | PostgreSQL StatefulSet and two replica Deployments for catalog, reservation, and web |

The backend packages follow domain, application, inbound adapter, and outbound adapter boundaries. Application use cases depend on ports; the web layer uses the same dependency direction with browser side domain rules, application operations, infrastructure gateways, and presentation components. Catalog reads and reservation commands/queries have separate use cases and HTTP endpoints, while sharing the local database instance.

## API routes

The web server proxies these paths to the two backend services:

| Method | Web path | Purpose |
| --- | --- | --- |
| `GET` | `/api/catalog/hotels?city=...` | Search hotels |
| `GET` | `/api/catalog/hotels/{hotelId}` | Read hotel details and room types |
| `GET` | `/api/booking/availability?checkIn=...&checkOut=...&guests=...` | Read room availability |
| `POST` | `/api/booking/reservations` | Create a reservation; requires `Idempotency-Key` |
| `GET` | `/api/booking/reservations?email=...` | List a guest’s reservations |
| `GET` | `/api/booking/reservations/{reservationId}?email=...` | Read a reservation |
| `DELETE` | `/api/booking/reservations/{reservationId}?email=...` | Cancel a reservation |

## Run locally with Kind

Prerequisites: Java 25, Node.js 24, Docker, Kind, and `kubectl`.

```bash
npm ci
./mvnw -f services/pom.xml verify
npm --workspace web run test -- --coverage
npm --workspace web run build
bash scripts/kind-up.sh
kubectl --context kind-hotel-system -n hotel-reservation get pods
kubectl --context kind-hotel-system -n hotel-reservation port-forward svc/web 4173:80
```

Open <http://127.0.0.1:4173>. `kind-up.sh` builds the three local images, loads them into Kind, applies the PostgreSQL and service manifests, and restarts each application Deployment so rebuilt local images are used. Each application Deployment has two replicas; PostgreSQL has one StatefulSet replica. To remove the application namespace, run `bash scripts/kind-down.sh`.

## Verification and quality

### SonarCloud

The monorepo is analyzed as the existing SonarCloud project [`marcelomiyake_hotel-ddd-cleanarch-cqrs`](https://sonarcloud.io/project/overview?id=marcelomiyake_hotel-ddd-cleanarch-cqrs). The latest analysis passed its quality gate and reports **0 open issues**, **0 bugs**, **0 vulnerabilities**, **0 code smells**, and **90.4% overall coverage** across **4,083 non-comment lines of code**.

Run `bash scripts/sonar-scan.sh` with `SONAR_TOKEN` available in the environment. It runs Maven verification, frontend tests with coverage, the frontend build, and the scanner. The scan configuration and project key are in `sonar-project.properties`; no token is stored in the repository.

### Tests and line coverage

The current local run passed 19 Java tests and 33 frontend tests.

| Component | Line coverage |
| --- | ---: |
| Catalog service (JaCoCo) | 98.20% |
| Reservation service (JaCoCo) | 97.92% |
| React application (Vitest/V8) | 87.82% |
| SonarCloud monorepo analysis | 90.4% |

### Lighthouse and metadata

The production frontend build, served by Vite preview with the local Kind services behind its API proxy, scored **100** in Performance, Accessibility, Best Practices, SEO, and Agentic Browsing. The report is generated at `web/lighthouse-report.json` and is ignored by Git.

The document includes a localized title and description, `lang`, viewport and theme metadata, robots directive, Open Graph and Twitter cards, favicon, hero image preload, and discovery links. `robots.txt`, `llms.txt`, and the `.well-known` AI catalog and ARD documents are served by the frontend. The Lighthouse SEO category scored 100.

The requested Chrome extension evaluation with **SEO META in 1 Click** could not be run: the connected Chrome session blocked its extension-management surface. I checked the document metadata directly and verified the Lighthouse SEO audit instead; this is not a result from the extension itself.

### OpenDesign handoff

The user supplied a self-hosted OpenDesign workspace and handoff path. The workspace endpoint responded, but the connected handoff could not return its active project or source artifact during this session. The frontend was implemented in the repository from the available design brief, but an OpenDesign source bundle was not imported or independently verified.

## Reservation abandonment analytics

The requested feature needs to distinguish a checkout that was started from one that finished, preserve the last screen visited, and make the event data available for future analysis without an administrative UI. The browser now emits events for reservation start, later screen views, and successful reservation confirmation. Events are stored in the reservation service's PostgreSQL schema, and the `reservation_funnel_abandonments` view exposes attempts that have been inactive for 30 minutes, were started, and have no confirmation. The view includes the last screen plus hotel and room identifiers.

The current checkout explicitly offers **pay at the property** and has no online payment step. For this implementation, a successful reservation confirmation is the completion event; an attempt that remains unconfirmed becomes an abandonment after 30 minutes without activity. The event model stores random session and attempt UUIDs, screen names, and hotel and room identifiers. It does not collect a guest name or email. Analytics writes are best effort and do not block reservation actions. Events share the existing PostgreSQL database and schema; no admin frontend or new user-facing booking step was added.

Tracked screens are `home`, `results`, `hotel`, `checkout`, `confirmation`, and `trips`. The screen at the start event is `checkout`; subsequent screen events update the last screen in the database view. A real online payment flow can later replace the confirmation event as the conversion signal.

## AI build record

This implementation was generated with Codex (GPT-6 Luna, max effort). The original prompts, token usage, cost estimates, and change counts for the initial build and the reservation-abandonment feature are in [docs/ai-build-record.md](docs/ai-build-record.md).
