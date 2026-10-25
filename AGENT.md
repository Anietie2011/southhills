# CLAUDE.md — Southhills Hotels & Suites

## 1. Your Role

Act as the senior full-stack engineer and technical architect responsible for helping build the Southhills Hotels & Suites website and hotel management system.

You are responsible for planning, implementing, reviewing, testing, and documenting the frontend, backend, database, and integrations.

Work as a careful engineering partner. Do not merely produce code snippets: inspect the repository, implement changes in the correct locations, test them, and explain the result.

Read `AGENTS.md` for the project's shared engineering rules. Follow both files. If an architectural or business requirement is unclear, ask a focused question rather than inventing a decision.

## 2. Business Goal

Build an original, premium, responsive digital platform for Southhills Hotels & Suites in Akwa Ibom State, Nigeria.

The platform must serve two main audiences:

### Guests

Guests should be able to:

- Explore the hotel and its facilities.
- View room categories, photographs, descriptions, and prices.
- Check room availability.
- Submit online reservations.
- Pay online when the payment integration is enabled.
- Receive booking confirmations and payment receipts.
- View and download authorized receipts.
- Find hotel contact and location information.

### Hotel staff

Authorized staff should be able to:

- Sign in to the management dashboard.
- Manage rooms, room categories, and availability.
- View and manage reservations.
- Create walk-in reservations for guests who arrive physically.
- Record cash payments and other approved payment methods.
- Check guests in and out.
- Monitor bookings that are approaching or have reached expiry.
- Receive relevant notifications.
- View payment and receipt records according to their permissions.
- Manage hotel content and media where authorized.
- Review appropriate operational and financial audit records.

The system should be designed around the hotel's actual workflow, not around unnecessary technical complexity.

## 3. Design Direction

The brand color is orange. The exact production color and logo assets should be confirmed from authorized hotel branding materials.

Design goals:

- Premium hospitality aesthetic.
- Original design rather than copying another hotel's website.
- Clear room photography and useful booking information.
- Thoughtful spacing, typography, hierarchy, and contrast.
- Smooth transitions and purposeful animations.
- Responsive mobile-first layouts.
- Accessible controls and forms.
- Professional loading, empty, success, and error states.

Use Tailwind CSS for styling, Framer Motion for suitable transitions, and GSAP for complex animations when there is a clear benefit.

Do not use animations that obstruct booking, reduce readability, or create unnecessary performance costs.

Do not invent the hotel's real room prices, amenities, logo, address, or contact details. Use clearly marked development placeholders until verified information is provided.

## 4. Technology Architecture

### Public website and management dashboard

- Next.js App Router.
- React.
- TypeScript.
- Tailwind CSS.
- Framer Motion and GSAP where appropriate.

### Backend

- NestJS.
- TypeScript.
- REST API.
- Swagger/OpenAPI.
- Socket.IO for appropriate real-time notifications.

### Database

- MongoDB Atlas.

### External services

- Cloudinary: hotel images and videos.
- Paystack: online payments.
- Resend: transactional email.
- Render: intended backend hosting.
- Vercel: intended frontend hosting.
- GitHub: source control.

Do not assume that accounts, credentials, domains, deployment settings, or live payment access are already configured merely because accounts have been created.

Use test credentials and sandbox modes during development. Production activation requires explicit authorization and the hotel's correct business details.

## 5. Architecture Principles

The NestJS backend owns business rules and is the authority for bookings, room availability, payments, permissions, and receipt access.

The Next.js frontend is responsible for user experience and displaying backend results.

MongoDB stores persistent business records. Cloudinary stores media files, while MongoDB stores relevant media metadata and references.

Paystack processes online payments. NestJS verifies payment outcomes using trusted server-side mechanisms.

PDFKit or another suitable backend PDF library generates receipts. Resend delivers receipt PDFs and other transactional emails.

Socket.IO communicates authorized updates but does not replace database persistence.

Keep service boundaries clear. Do not duplicate critical business rules in frontend components.

## 6. Suggested Application Organization

Prefer this organization for a new monorepo, subject to inspection of the existing repository:

    apps/
      web/
        src/
          app/
          components/
          features/
          hooks/
          lib/
          types/
      api/
        src/
          auth/
          users/
          guests/
          rooms/
          reservations/
          payments/
          receipts/
          notifications/
          media/
          audit-logs/
          common/
          config/
    packages/
      shared/
    docs/

Adapt the structure as implementation progresses. Do not create every module before its purpose is clear.

Keep the following responsibilities separate:

- Controllers handle HTTP requests and responses.
- Services implement business logic.
- Database repositories or data-access services handle persistence.
- DTOs and validators validate incoming data.
- Guards and authorization policies enforce permissions.
- Frontend components render user interfaces.
- Shared packages contain genuinely reusable types or validation logic.

Avoid unnecessary abstraction and avoid putting the whole application into a single service or component.

## 7. Authentication and Staff Permissions

Design role-based access control for the hotel's workflow.

Candidate roles:

- SUPER_ADMIN
- MANAGER
- RECEPTIONIST
- ACCOUNTANT
- CONTENT_MANAGER

These are proposed roles, not an instruction to grant every role all permissions. Confirm final permissions before enabling sensitive operations.

For example:

- Receptionists can create walk-in bookings, view appropriate reservation details, check guests in and out, and record cash payments if authorized.
- Accountants can review financial records and perform only explicitly authorized financial operations.
- Content managers can manage authorized hotel content and media.
- Managers can perform appropriate operational oversight.
- Super administrators manage system-level settings and staff permissions.

Enforce permissions on the backend, not only by hiding buttons in the frontend.

Record the staff identity for sensitive operations. Never let an ordinary user modify their own permissions or bypass authorization by calling an API directly.

## 8. MongoDB Data Model

Design and document the final schema before implementing complicated cross-collection workflows.

Potential collections:

### users

Staff identities, authentication information, active state, and assigned roles.

### guests

Guest contact and booking-related information. Minimize unnecessary personal data.

### roomTypes

Room category, description, approved price configuration, occupancy rules, and media references.

### rooms

Physical room identifier, room category, operational status, and maintenance information.

### reservations

Booking reference, guest reference, room category, assigned room if applicable, arrival and departure times, price snapshot, booking status, payment summary, and relevant timestamps.

### payments

Payment method, amount, currency, payment status, transaction reference, recording staff, verification metadata, and timestamps.

### receipts

Unique receipt number, associated booking and payment references, issue information, and any metadata needed for authorized retrieval.

### reservationStatusHistory

Relevant status transitions, actor, timestamp, and reason where applicable.

### notifications

Notification type, recipient or audience, related entity, creation time, and read state where useful.

### mediaAssets

Cloudinary public IDs, secure URLs or delivery references, media type, dimensions, and relevant content associations.

### maintenanceBlocks

Room or category maintenance periods and the staff member who authorized them.

### auditLogs

Sensitive operational actions, actor, affected entity, timestamp, action type, and relevant change metadata.

This is an initial proposal, not a fixed requirement to implement every collection. Keep the schema proportionate to the actual features.

Use appropriate unique indexes for booking references, receipt numbers, and payment transaction identifiers. Avoid inventing uniqueness constraints that conflict with real hotel workflows.

Store monetary amounts precisely, define the currency explicitly, and snapshot agreed booking prices. Do not recalculate historical bookings using today's prices.

## 9. Reservation Workflow

Support two booking channels:

1. Online bookings created by guests.
2. Walk-in bookings created by authorized receptionists.

For either channel, the backend must:

1. Validate the requested dates and room category.
2. Determine availability from authoritative database state.
3. Calculate prices on the server using approved pricing rules.
4. Create the reservation and relevant records.
5. Apply concurrency safeguards.
6. Return the resulting booking state.
7. Notify authorized staff when appropriate.

Do not trust a price, room availability result, payment status, or staff role supplied by the browser.

### Room assignment

The initial design may reserve a room category and assign a physical room later if that matches the hotel's process. If the hotel requires exact room selection at booking time, enforce availability for that physical room.

Confirm the chosen assignment policy before implementing the final reservation workflow.

### Date handling

Define a consistent hotel timezone and checkout time. Store timestamps consistently, preferably in UTC, and display them in the hotel's local timezone.

Use a clear date-range convention. A common convention is arrival inclusive and departure exclusive, but confirm it against the hotel's actual rules.

Do not treat a date alone as proof that a room is physically available.

## 10. Expiry and Two-Hour Grace Period

The hotel has requested the following behavior:

- A checked-in guest is expected to check out on the agreed expiry date.
- The system does not track the guest's temporary movements in and out of the property.
- The system notifies staff when a reservation or stay reaches its configured expiry time.
- A two-hour grace period follows the expiry time for the hotel's rebooking process.
- The system notifies staff when that grace period ends.
- The end of the grace period does not prove the room is vacant.

Implement these rules with server-side timestamps and reliable scheduled processing or a reconciliation mechanism appropriate to the deployment.

Notifications must be safe to retry without creating duplicates.

Do not automatically assign an occupied room to a new guest. Room release requires the appropriate checkout and physical availability confirmation.

The precise checkout hour, grace-period start, room-release rules, and any manager override must be confirmed and documented before production use.

## 11. Payment Design

Keep booking state and payment state separate.

Illustrative booking states:

- PENDING
- CONFIRMED
- CHECKED_IN
- COMPLETED
- CANCELLED
- EXPIRED

Illustrative payment states:

- PENDING
- PAID
- FAILED
- PARTIALLY_PAID
- REFUNDED
- PARTIALLY_REFUNDED

These are starting proposals. Choose the final states according to the hotel workflow, and do not combine unrelated states into a single ambiguous field.

### Online Paystack payments

1. The frontend requests a booking or payment session.
2. NestJS validates the reservation and calculates the amount.
3. NestJS initializes a Paystack transaction using server-side credentials.
4. The guest completes the payment process.
5. NestJS verifies the transaction using a trusted Paystack verification mechanism.
6. A signed webhook may also report payment events.
7. NestJS validates webhook signatures and handles duplicate events idempotently.
8. The backend updates payment records and the appropriate booking state.
9. The receipt becomes available after payment has been verified.
10. Authorized notifications and email delivery are triggered.

Do not rely on a frontend redirect alone to confirm payment. Do not mark a payment as paid merely because the browser says it succeeded.

### Receptionist cash payments

1. An authorized receptionist creates a walk-in booking.
2. The backend validates room availability.
3. The receptionist records the amount actually received and selects Cash.
4. The backend records the staff identity and timestamp.
5. The system updates payment records and reservation state according to hotel policy.
6. The system generates the appropriate receipt.
7. The receipt can be viewed, downloaded, or printed.
8. Resend can email the receipt if the guest provides an email address.

Cash payment records must be auditable. Do not silently rewrite or delete completed payment records.

Support partial payments only if the hotel explicitly approves them. If supported, calculate and display the outstanding balance accurately.

## 12. Receipt Generation and Delivery

Use a backend PDF library such as PDFKit unless a tested alternative is justified.

The backend generates a consistent, branded PDF containing appropriate details, such as:

- Hotel name and authorized branding.
- Unique receipt number.
- Booking reference.
- Guest and room information.
- Check-in and checkout details.
- Amount actually paid.
- Payment method.
- Payment reference where applicable.
- Payment status.
- Issue timestamp.

Never fabricate payment details or issue a paid receipt for an unverified online payment.

### Guest-facing flow

After successful payment verification:

1. Display a payment success page.
2. Display the booking reference and relevant booking summary.
3. Make the receipt available through a View Receipt action.
4. Provide Download PDF and Print Receipt actions.
5. Send the same receipt PDF through Resend.
6. Confirm email sending only after the email provider accepts the send request.
7. If email sending fails, preserve receipt access and offer an appropriate retry mechanism.

The guest should not lose the ability to access their receipt because email delivery failed.

For walk-in cash bookings, authorized staff can view, print, or download the receipt and email it when an address is available.

Protect receipt endpoints using authentication, authorization, or appropriately scoped short-lived access tokens. Receipt numbers must not function as unrestricted public access credentials.

Avoid generating inconsistent documents for the same payment. Reuse the original receipt record and document bytes when appropriate. Any corrected receipt must have an auditable relationship to the original.

## 13. Resend Email Integration

Use Resend for transactional email only after configuration is ready.

Expected email types include:

- Booking confirmation.
- Payment receipt.
- Booking updates.
- Relevant expiry notifications.
- Payment-related notifications.

Use appropriate verified sender identities and domain configuration for production.

Do not embed API keys in client-side code. Handle provider failures, retries, and duplicate delivery risks. Avoid emailing receipts repeatedly because of duplicate webhook events.

Store enough delivery metadata to support troubleshooting without retaining unnecessary sensitive content.

## 14. Cloudinary Integration

Use Cloudinary for hotel photographs and videos.

Requirements:

- Store media references and metadata in MongoDB.
- Use server-authorized or signed upload flows where appropriate.
- Restrict allowed formats and upload sizes.
- Validate upload authorization.
- Keep API secrets server-side.
- Avoid orphaning media when records are updated or removed.
- Use optimized delivery formats and image sizes for the frontend.

Do not assume production hotel photographs or logos are available until they have been provided or approved.

## 15. Real-Time Notifications

Use Socket.IO when real-time dashboard updates materially improve operations.

Potential events:

- New reservation.
- Reservation cancellation.
- Payment verified.
- Cash payment recorded.
- Upcoming expiry.
- Expiry reached.
- Grace period ended.
- Room maintenance or availability change.

Authorize socket connections and restrict events to the intended users or roles.

Persist important business events and records in MongoDB. Socket messages can be missed, so dashboards must also fetch authoritative state from the backend.

Design event processing to tolerate reconnects, retries, and duplicate notifications.

## 16. Dashboard Design

Build a practical dashboard rather than an oversized analytics interface.

Potential sections:

- Overview.
- Reservations.
- Rooms and room categories.
- Guests.
- Check-in and checkout.
- Payments.
- Receipts.
- Notifications.
- Media and hotel content.
- Staff and permissions.
- Audit records.
- Settings.

Receptionists need quick access to:

- Room availability.
- Create walk-in booking.
- Record cash payment.
- Search reservations.
- Check guests in and out.
- View and print receipts.
- Review arrivals, departures, and expiry alerts.

Keep sensitive financial reports and staff administration restricted to authorized roles.

Do not implement every proposed dashboard section at once. Build and test the core reservation and payment workflows first.

## 17. API and Validation

Use NestJS DTOs and appropriate validation for all externally supplied input.

Use Swagger/OpenAPI to document the API as endpoints are implemented.

Potential endpoint groups include:

- `/auth`
- `/rooms`
- `/room-types`
- `/reservations`
- `/guests`
- `/payments`
- `/receipts`
- `/notifications`
- `/media`
- `/staff`

The exact routes and request formats must be designed and documented during implementation.

Use correct HTTP methods and status codes. Apply authentication, role authorization, rate limits, pagination, and filtering where appropriate.

Do not expose database documents indiscriminately. Return explicit response shapes and avoid leaking internal fields.

## 18. Testing Strategy

Write tests for business-critical behavior before treating it as production-ready.

Priorities:

- Reservation validation.
- Room availability.
- Concurrent booking attempts.
- Price calculation.
- Payment verification.
- Duplicate webhook processing.
- Cash payment authorization.
- Receipt generation.
- Receipt access permissions.
- Email delivery failure.
- Booking expiry.
- Two-hour grace-period handling.
- Checkout and room-release workflow.
- Staff role enforcement.
- Audit log creation.

Test the frontend's loading, error, and success states.

Run type checks, linting, and relevant tests after changes. Report any untested areas honestly.

## 19. Deployment and Environment Configuration

Use environment variables for service credentials and environment-specific settings.

Potential configuration names include:

- `MONGODB_URI`
- `PAYSTACK_SECRET_KEY`
- `PAYSTACK_PUBLIC_KEY`
- `PAYSTACK_WEBHOOK_SECRET` if applicable to the configured integration
- `RESEND_API_KEY`
- `RESEND_FROM_EMAIL`
- `CLOUDINARY_CLOUD_NAME`
- `CLOUDINARY_API_KEY`
- `CLOUDINARY_API_SECRET`
- `NEXT_PUBLIC_API_URL`
- `FRONTEND_URL`
- `PORT`
- `NODE_ENV`

These are proposed names. Confirm them against the implemented configuration and the current provider requirements.

Do not use secret values as `NEXT_PUBLIC_` variables.

Create an `.env.example` with placeholder values only. Ensure secret-bearing `.env` files are ignored by Git.

Configure CORS to allow only the intended frontend origins. Use HTTPS for production and keep development, test, and production credentials separate.

Before deployment, verify current provider limits, persistent storage needs, service sleep behavior, backup options, monitoring, and production costs.

Do not enable live payments or deploy sensitive production changes without explicit authorization.

## 20. Development Workflow

For each feature:

1. Inspect the current repository and related files.
2. Identify the requirement and relevant business rules.
3. Explain the implementation plan briefly.
4. Make focused changes.
5. Add or update tests.
6. Run relevant checks.
7. Review security and data integrity implications.
8. Update documentation where needed.
9. Report changed files, test results, configuration needs, and remaining issues.

Prefer vertical slices that work end to end. For example, implement reservation creation across frontend, backend, and database before expanding unrelated dashboard functionality.

Do not fabricate completion, hide errors, or skip validation to make a feature appear finished.

## 21. Recommended Implementation Phases

### Phase 1 — Foundation

- Inspect the repository.
- Establish the frontend and backend structure.
- Configure TypeScript and linting.
- Configure MongoDB access.
- Add environment validation.
- Establish API error handling and health checks.

### Phase 2 — Authentication and permissions

- Implement staff authentication.
- Implement role-based authorization.
- Secure the management dashboard.

### Phase 3 — Hotel data and rooms

- Implement room categories and physical rooms.
- Implement hotel content and Cloudinary media references.
- Establish initial availability rules.

### Phase 4 — Reservations

- Implement reservation creation.
- Implement availability checks and concurrency controls.
- Implement online and walk-in booking workflows.
- Implement receptionist operations.

### Phase 5 — Payments

- Integrate Paystack test mode.
- Verify transactions and webhooks.
- Implement authorized cash payment records.
- Add payment history and audit records.

### Phase 6 — Receipts and email

- Generate PDF receipts.
- Add secure receipt viewing and downloading.
- Add print support.
- Integrate Resend.
- Test email failures and retries.

### Phase 7 — Operational dashboard

- Add check-in and checkout.
- Add expiry notifications and the two-hour grace period.
- Add room-release confirmation and relevant staff workflows.
- Add authorized real-time notifications.

### Phase 8 — Public website

- Build hotel pages and room details.
- Connect live room availability.
- Build the guest booking experience.
- Add the payment confirmation and receipt experience.

### Phase 9 — Quality and launch preparation

- Complete integration and security testing.
- Review access permissions.
- Test backups and recovery.
- Configure monitoring and deployment.
- Verify production service requirements and costs.
- Obtain hotel approval before enabling live operations.

The phases may be adjusted if repository conditions or dependencies require it. Do not treat the order as permission to skip prerequisite security or payment safeguards.

## 22. Final Handover Documentation

After implementation, prepare comprehensive documentation that accurately describes the system as actually built.

Include:

- Project overview and setup.
- Repository structure.
- Frontend architecture.
- Backend modules and APIs.
- MongoDB collections, indexes, and relationships.
- Authentication and staff permissions.
- Reservation and room-availability logic.
- Expiry and grace-period behavior.
- Paystack integration and verification.
- Cash payment workflow.
- PDF receipt generation and secure retrieval.
- Resend email delivery.
- Cloudinary media workflow.
- Socket.IO events.
- Environment configuration.
- Deployment and rollback procedures.
- Backups and recovery.
- Security considerations.
- Troubleshooting.
- Receptionist and management user guides.
- Known limitations and future improvements.

Do not document unimplemented features as if they already exist. Keep the documentation consistent with the real code and configuration.

## 23. Definition of Success

The project succeeds when guests can use a polished website to make valid reservations, authorized staff can manage walk-in and online bookings, payments and receipts are handled accurately, room availability is protected against conflicts, and the hotel's operational records remain secure and auditable.

Prioritize reliable hotel operations, financial correctness, data protection, accessibility, maintainability, and clear staff workflows over unnecessary features or premature complexity.
