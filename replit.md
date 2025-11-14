# Wedding Website - Fabio & Denise

## Overview

This is a full-stack wedding website built for Fabio and Denise's December 21, 2025 wedding at Tenuta Donna Fausta in Roccaromana, Italy. The application provides guests with event details, RSVP functionality, photo gallery, travel information, and gift registry access. An admin panel allows the couple to manage RSVPs, guest lists, events, gallery items, and site configuration.

The application is built as a modern single-page application with a React frontend and Express backend, using PostgreSQL for data persistence.

## Recent Changes

**November 9, 2025 - Security Fix & Admin Configuration Endpoints:**
- Fixed critical security vulnerability: SMTP credentials no longer exposed via public API
- Created dedicated admin endpoints for sensitive configuration:
  - GET/PUT `/api/admin/site-config`: Site settings (coupleInitials, siteTitle, welcomeMessage, footerMessage)
  - GET/PUT `/api/admin/rsvp-config`: RSVP and SMTP settings (rsvpEmail, rsvpDeadlineDate, smtp*)
- Public `/api/config` endpoint now filters sensitive keys (smtpPassword, smtpUser, smtpHost, smtpPort)
- SMTP password never returned from any endpoint (always empty string for security)
- Added Zod validation for all admin configuration endpoints to prevent malformed data
- Frontend updated to use separate authenticated queries for sensitive configuration
- SMTP password field is optional in admin UI (only updates if new value provided)

**November 9, 2025 - Surprise Events & Event Enhancements:**
- Implemented surprise events feature with reveal dates:
  - Added `isSurprise` and `revealDate` fields to events table
  - Events hidden from public until reveal date is reached
  - Timezone-safe date comparison logic for worldwide consistency
  - Frosted glass teaser cards integrated into timeline with "Scopri l'evento a partire da [date]" message
  - Surprise main events display with frosted styling in main section
- Enhanced main events display:
  - Dynamic icon mapping: ceremony (Church), reception (GlassWater), party (PartyPopper)
  - Map links with "Visualizza Mappa" button for main events
  - `excludeFromList` flag to hide main events from timeline
  - Photo upload support for non-main events
  - Surprise main events show frosted glass styling when not revealed
- Improved admin Events page:
  - Scrollable modal (`max-h-60vh`) for all event fields
  - Conditional form sections based on event type
  - Zod validation: requires reveal date when event is marked as surprise
  - Empty string to null transforms for clean database payloads
- Refactored public EventDetails component:
  - Single API query with client-side categorization
  - Two sections: main events grid (with icons/maps), timeline (with photos and surprise teasers)
  - Surprise events integrated inline in timeline with frosted glass styling
  - Mobile timeline fixed: dots and vertical line now visible on all screen sizes
  - Revealed surprise events automatically display normally in their sections

**November 9, 2025 - Configuration & Settings:**
- Added extensive configurability for travel information, gift registry, bank details, and contact info
- Implemented admin Settings page with tabbed interface (Travel, Gift Registry, Site Settings)
- Fixed mobile layout for "La Proposta" section (photo first, text second on mobile)
- Created 5 new database tables: travel_info, travel_accommodations, gift_registry_settings, bank_details, contact_info
- Implemented complete CRUD APIs for all configurable sections
- Updated all frontend components to use configurable data from database

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend Architecture

**Framework & Build Tools:**
- React 18 with TypeScript for type safety
- Vite as the build tool and development server
- Wouter for lightweight client-side routing
- TanStack Query (React Query) for server state management and caching

**UI Components:**
- Radix UI primitives for accessible, unstyled components
- shadcn/ui component library built on Radix UI
- Tailwind CSS for utility-first styling with custom theme
- Framer Motion for animations and scroll effects
- Custom ScrollAnimator component for intersection-observer-based animations

**State Management:**
- React Query handles server state, caching, and API synchronization
- React Context for authentication state (AuthContext)
- Session-based authentication with cookies
- Local component state with React hooks (useState, useEffect)

**Form Handling:**
- React Hook Form for form state management
- Zod for schema validation
- @hookform/resolvers for integrating Zod with React Hook Form

**Design System:**
- Custom color palette: gold (#d4b78f), cream (#f8f5f0), dark text (#333333)
- Custom fonts: Cormorant Garamond, Great Vibes, Montserrat
- Responsive design with mobile-first approach
- Theme configuration stored in theme.json

### Backend Architecture

**Server Framework:**
- Express.js as the HTTP server
- TypeScript for type safety across the stack
- Session-based authentication using express-session
- Cookie parsing with cookie-parser
- JSON body parsing for API requests

**API Design:**
- RESTful API structure with `/api` prefix for public endpoints
- `/api/admin` prefix for protected administrative endpoints
- Authentication middleware (isAuthenticated) protects admin routes
- Centralized error handling and request logging

**Database Layer:**
- Drizzle ORM for type-safe database queries
- Schema definitions in `shared/schema.ts` using drizzle-orm/pg-core
- Drizzle Kit for schema migrations in `/migrations` directory
- IStorage interface pattern for potential storage abstraction
- In-memory storage (MemStorage) as fallback implementation

**Database Schema:**
- `rsvps` table: Guest RSVP submissions with attendance, dietary needs, messages
- `users` table: Admin users with bcrypt-hashed passwords
- `site_config` table: Key-value configuration storage
- `guest_list` table: Pre-populated guest list for tracking
- `events` table: Wedding event details with isMainEvent flag (max 3 main events)
- `gallery` table: Photo gallery items with URLs and descriptions
- `story_sections` table: Configurable "Our Story" sections (meet, proposal) with images and text
- `travel_info` table: Venue details, maps link, coordinates, directions text (singleton, ID=1)
- `travel_accommodations` table: Hotel/accommodation recommendations with rates and booking links
- `gift_registry_settings` table: Gift list and honeymoon section toggles, description, contribution link (singleton, ID=1)
- `bank_details` table: Bank transfer information with enable toggle (singleton, ID=1)
- `contact_info` table: Email and two phone numbers for footer contact (singleton, ID=1)
- `session` table: PostgreSQL-backed session storage for production persistence

**Authentication Strategy:**
- Session-based authentication with secure, httpOnly cookies
- Trust proxy configured for Replit's reverse proxy (secure cookies work correctly)
- connect-pg-simple for persistent sessions in production (MemoryStore in development)
- Bcrypt password hashing (10 rounds recommended for production)
- Authentication service in `server/services/authService.ts`
- Session data includes userId and username
- 24-hour session expiration
- Cookie settings: httpOnly=true, sameSite=lax, secure=true (production only)

**Business Logic Services:**
- Email service (`server/services/emailService.ts`) for RSVP notifications
- Nodemailer integration with Gmail SMTP (configurable via environment variables)
- Authentication service for password verification

### External Dependencies

**Database:**
- PostgreSQL via @neondatabase/serverless
- Neon Serverless Postgres for cloud-hosted database
- Connection configured via DATABASE_URL environment variable
- Drizzle ORM for type-safe queries and migrations

**Email Service:**
- Nodemailer for transactional emails
- Gmail SMTP configuration (customizable)
- Environment variables: EMAIL_USER, EMAIL_PASSWORD
- RSVP notification emails sent to admin

**Third-Party UI Libraries:**
- Radix UI component primitives (@radix-ui/react-*)
- Lucide React for iconography
- cmdk for command palette functionality
- Framer Motion for animations
- React Day Picker for calendar components
- Embla Carousel for image carousels

**Development Tools:**
- Replit-specific plugins for development environment
- @replit/vite-plugin-shadcn-theme-json for theme management
- @replit/vite-plugin-runtime-error-modal for error overlay
- @replit/vite-plugin-cartographer for code mapping (dev only)

**Environment Configuration:**
- DATABASE_URL: PostgreSQL connection string (required)
- SESSION_SECRET: Session encryption key (defaults to 'matrimonio-segreto')
- EMAIL_USER: SMTP username for email sending
- EMAIL_PASSWORD: SMTP password for email sending
- NODE_ENV: Environment flag (development/production)

**Build & Deployment:**
- Development: `tsx server/index.ts` for hot-reloading
- Production build: Vite bundles frontend, esbuild bundles backend
- Static assets served from `dist/public`
- Server runs on Node.js with ES modules

**API Integration Pattern:**
- Client uses custom `apiRequest` wrapper for fetch calls
- Credentials included in all requests for session cookies
- Centralized query client configuration
- Optimistic updates and cache invalidation via React Query
- Custom query functions with 401 handling strategies