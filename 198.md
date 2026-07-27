# Enterprise App UI Template — Spec

## Problem Statement

Enterprise teams need a reusable, production-ready Next.js template for building internal tools and admin dashboards. Starting from scratch for each new project wastes weeks on scaffolding layout, auth, theming, data tables, forms, and page structure — all of which follow predictable patterns. The template must be opinionated enough to be immediately useful, yet flexible enough to accommodate diverse enterprise use cases without customer-facing requirements.

## Solution

A full-featured Next.js 15 enterprise app template with:

- **Sidebar + Topbar layout** with collapsible sidebar navigation
- **Auth.js integration** with GitHub provider for demo/preview and Keycloak for production
- **Full page set**: Dashboard (overview), Users, Settings, Profile, Notifications, Audit Log, API Keys, Reports/Analytics, Role Management, System Health
- **Tailwind CSS + shadcn/ui** component library with dark mode toggle (system preference + manual switch)
- **Server Components + Zustand** for data fetching and client-side state
- **React Hook Form + Zod** for form validation
- **TanStack Table** for data tables with custom table fallback
- **next-intl** for internationalization
- **Vitest + Playwright** for unit/component and end-to-end testing
- **Docker** for production deployment, **Vercel** for demo/preview

## User Stories

### Layout & Navigation

1. As an enterprise user, I want a persistent sidebar on the left side of the screen, so that I can navigate between sections quickly
2. As an enterprise user, I want to collapse the sidebar to an icon-only mode, so that I can maximize my content area when needed
3. As an enterprise user, I want a top bar with search, notifications badge, and user menu, so that I have quick access to global actions
4. As an enterprise user, I want the sidebar state (collapsed/expanded) to persist across page navigations, so that my preference is remembered
5. As an enterprise user, I want the sidebar to be responsive — full sidebar on desktop, off-canvas drawer on mobile, so that the layout works on all screen sizes

### Authentication & Authorization

6. As an enterprise user, I want to sign in with GitHub during demo/preview, so that I can quickly access the app without setup
7. As an enterprise user, I want to authenticate via Keycloak in production, so that the app integrates with our corporate SSO
8. As an enterprise user, I want to see my name and avatar in the top bar user menu, so that I can confirm I'm logged in
9. As an enterprise user, I want a sign-out option in the user menu, so that I can securely end my session
10. As an enterprise user, I want to be redirected to the login page when my session expires, so that I'm not shown stale content
11. As an enterprise user, I want role-based access to sidebar items, so that I only see navigation relevant to my permissions

### Dashboard

12. As an enterprise user, I want to see an overview dashboard with key metrics cards, so that I can get a quick snapshot of system status
13. As an enterprise user, I want to see a recent activity feed on the dashboard, so that I can stay informed about what's happening
14. As an enterprise user, I want to see charts/visualizations on the dashboard, so that I can understand trends at a glance
15. As an enterprise user, I want the dashboard data to load via Server Components, so that the page loads fast

### Users Management

16. As an enterprise user, I want to see a list of all users in a searchable, sortable, filterable table, so that I can find users quickly
17. As an enterprise user, I want to view a user's details in a side panel or modal, so that I can inspect their profile without leaving the list
18. As an enterprise user, I want to create a new user via a form with validation, so that I can onboard team members
19. As an enterprise user, I want to edit an existing user's details, so that I can keep records up to date
20. As an enterprise user, I want to deactivate/delete a user, so that I can manage access
21. As an enterprise user, I want to assign roles to users, so that I can control their permissions
22. As an enterprise user, I want to bulk-select users and perform batch actions, so that I can manage many users efficiently

### Settings

23. As an enterprise user, I want to view and edit application settings (general, appearance, notifications), so that I can customize the system
24. As an enterprise user, I want settings organized in tabs or sections, so that I can find what I need quickly
25. As an enterprise user, I want form validation on all settings changes, so that I don't save invalid configuration
26. As an enterprise user, I want a confirmation dialog before saving critical settings, so that I don't make accidental changes

### Profile

27. As an enterprise user, I want to view my own profile (name, email, avatar, role), so that I can verify my information
28. As an enterprise user, I want to edit my profile details, so that I can update my information
29. As an enterprise user, I want to change my password, so that I can maintain account security

### Notifications

30. As an enterprise user, I want to see a list of all notifications (system alerts, user actions, etc.), so that I can stay informed
31. As an enterprise user, I want to mark notifications as read/unread, so that I can track what I've seen
32. As an enterprise user, I want to filter notifications by type, date, or status, so that I can find relevant ones
33. As an enterprise user, I want to clear all notifications, so that I can manage my notification inbox
34. As an enterprise user, I want notification preferences (what to be notified about), so that I can control what alerts I receive

### Audit Log

35. As an enterprise user, I want to see a chronological log of all system actions, so that I can audit who did what and when
36. As an enterprise user, I want to filter the audit log by user, action type, date range, and resource, so that I can investigate specific events
37. As an enterprise user, I want to export the audit log as CSV, so that I can share or archive it
38. As an enterprise user, I want to see the audit log in a paginated table, so that I can navigate through large volumes of data

### API Keys

39. As an enterprise user, I want to see a list of all API keys, so that I can manage programmatic access
40. As an enterprise user, I want to generate a new API key with a descriptive name and scope, so that I can create integrations
41. As an enterprise user, I want to revoke an API key, so that I can remove access when it's no longer needed
42. As an enterprise user, I want to see when an API key was last used, so that I can identify inactive keys
43. As an enterprise user, I want to see the API key value only once at creation, so that security is maintained

### Reports & Analytics

44. As an enterprise user, I want to view analytics dashboards with charts, so that I can understand system usage patterns
45. As an enterprise user, I want to filter reports by date range, so that I can analyze specific time periods
46. As an enterprise user, I want to export reports as PDF or CSV, so that I can share findings with stakeholders
47. As an enterprise user, I want to see summary statistics (totals, averages, trends), so that I can get high-level insights

### Role Management

48. As an enterprise user, I want to see a list of all roles in the system, so that I can understand the permission structure
49. As an enterprise user, I want to create a new role with specific permissions, so that I can define custom access levels
50. As an enterprise user, I want to edit an existing role's permissions, so that I can adjust access as needs change
51. As an enterprise user, I want to delete a role (with confirmation), so that I can remove unused roles
52. As an enterprise user, I want to see which users are assigned to each role, so that I can understand role distribution

### System Health

53. As an enterprise user, I want to see the current system status (services, uptime, response times), so that I can monitor infrastructure health
54. As an enterprise user, I want to see a history of incidents or alerts, so that I can review past issues
55. As an enterprise user, I want to see resource usage (CPU, memory, storage), so that I can plan capacity

### Dark Mode

56. As an enterprise user, I want to toggle between light and dark themes, so that I can work comfortably in different lighting conditions
57. As an enterprise user, I want the theme to follow my OS preference by default, so that it matches my system setting
58. As an enterprise user, I want my theme choice to persist across sessions, so that I don't have to re-select it

### Internationalization

59. As an enterprise user, I want the UI to support multiple languages, so that non-English speakers can use the app
60. As an enterprise user, I want to switch languages from the UI, so that I can change without developer intervention
61. As an enterprise user, I want date/number formatting to respect my locale, so that data is displayed correctly

### Forms & Validation

62. As an enterprise user, I want forms to validate input in real-time, so that I can fix errors before submission
63. As an enterprise user, I want clear error messages on invalid fields, so that I know exactly what to fix
64. As an enterprise user, I want forms to show a loading state during submission, so that I know the system is processing
65. As an enterprise user, I want forms to show success feedback after saving, so that I know my changes were applied

### Data Tables

66. As an enterprise user, I want tables with sortable columns, so that I can organize data by any field
67. As an enterprise user, I want tables with search/filter functionality, so that I can find specific records
68. As an enterprise user, I want paginated tables, so that I can navigate large datasets
69. As an enterprise user, I want tables to handle loading and empty states, so that I understand what's happening
70. As an enterprise user, I want to resize table columns, so that I can customize my view

### Testing

71. As a developer, I want unit tests for business logic, so that correctness is verified quickly
72. As a developer, I want component tests for UI components, so that rendering and interaction behavior is correct
73. As a developer, I want E2E tests for critical flows (login, navigation, form submission), so that full user journeys work

### Deployment

74. As a developer, I want to preview changes on Vercel before merging, so that I can verify the app in a production-like environment
75. As a developer, I want a Dockerfile for production deployment, so that I can deploy to any cloud or on-premise infrastructure
76. As a developer, I want docker-compose for local development with dependencies, so that I can run the full stack locally

## Implementation Decisions

### Project Structure (Hybrid: Features + Layers)

- `/app` — Next.js App Router routes (pages)
- `/app/(auth)` — auth-related routes (login, callback)
- `/app/(dashboard)` — main app routes with sidebar layout
- `/components/ui` — shared shadcn/ui primitives
- `/components/layout` — Sidebar, Topbar, ThemeToggle, etc.
- `/features/<name>` — domain-specific modules, each containing:
  - `components/` — feature-specific UI components
  - `hooks/` — feature-specific hooks
  - `services/` — API/data fetching logic
  - `types/` — TypeScript types
- `/lib` — utilities, Auth.js config, i18n setup, constants
- `/stores` — Zustand stores (ui store, feature stores)
- `/messages` — next-intl translation JSON files
- `/e2e` — Playwright test files

### Auth.js Configuration

- Auth.js configured with GitHub provider (demo) and Keycloak OIDC provider (production)
- Environment variable-driven provider selection: `AUTH_PROVIDER=github|keycloak`
- Session strategy: JWT (stateless, works with Docker)
- Protected routes via middleware: all `(dashboard)` routes require authentication
- Mock user data available for local development without auth providers

### Layout Architecture

- Root layout: minimal (html, body, fonts, ThemeProvider)
- `(dashboard)/layout.tsx`: Sidebar + Topbar shell with `<Outlet>`
- Sidebar: server-rendered navigation items, client-side collapse toggle
- Topbar: client-side search input, notification bell, user dropdown
- Sidebar state managed via Zustand UI store, persisted to localStorage

### State Management

- Server Components: default for all data-fetching pages (dashboard stats, user lists, audit logs)
- Zustand stores:
  - `uiStore` — sidebar collapsed, theme, locale
  - Feature stores as needed (e.g., `usersStore` for selection state)
- No React Query — Server Components handle server data, client mutations via Server Actions

### Forms

- React Hook Form for all forms (settings, user creation/editing, profile, API key generation)
- Zod schemas for validation, co-located with the form or in `features/<name>/types/`
- shadcn/ui Form components (built on RHF) for consistent styling

### Tables

- TanStack Table as default for all data tables (users, audit log, API keys, roles)
- Custom table components allowed for simple or one-off cases
- Table features: sorting, filtering, pagination, column visibility, row selection

### Dark Mode

- shadcn/ui dark mode via CSS variables (`light`/`dark` classes on `<html>`)
- ThemeProvider from next-themes wrapping the app
- Toggle button in Topbar user menu and in Settings page
- System preference as default, manual override persisted to localStorage

### i18n

- next-intl with App Router middleware-based locale detection
- Default locale: English
- Translation files in `/messages/{locale}.json`
- All user-facing strings extracted to translation keys
- Language switcher in the Topbar user menu

### Deployment

- **Vercel**: default `next build` + `next start`, used for demo/preview deployments
- **Docker**: multi-stage Dockerfile (deps → build → production), `docker-compose.yml` for local prod-like environment
- Docker build optimized for Next.js standalone output mode

## Testing Decisions

### Unit/Component Tests (Vitest)

- Vitest configured with React Testing Library
- Test files co-located: `*.test.tsx` next to components, `*.test.ts` next to utilities
- Test external behavior (rendered output, user interactions), not implementation details
- Priority: form validation logic, Zustand store actions, utility functions, component rendering

### E2E Tests (Playwright)

- Playwright configured for Chromium, Firefox, WebKit
- Test files in `/e2e/` directory
- Priority flows to test: authentication (login/logout), sidebar navigation, form submission, table filtering/sorting
- Tests should use the mock auth setup for CI (no real GitHub/Keycloak needed)

### What Makes a Good Test

- Tests verify user-visible behavior, not internal state or implementation
- Tests are independent and can run in any order
- Tests use data-testid or accessible roles for selectors, not CSS classes
- Mock external dependencies (API, auth) at the boundary, not deep in the stack

## Out of Scope

- Real backend/API implementation — this is a UI template with mock data
- Database setup — no ORM, no migrations, no real data persistence
- CI/CD pipeline configuration — consumers will set up their own
- Specific Keycloak server setup — only the Auth.js client config is included
- Mobile app (React Native) — web only
- WebSocket/real-time features — polling or static data only for the template
- Internationalization of all strings in the first pass — i18n infrastructure is in place, but not all pages will have full translations initially
- Production-grade error boundaries and monitoring — basic error handling only

## Further Notes

- The template should be immediately runnable after `pnpm install && pnpm dev` with GitHub OAuth configured
- Mock data should be realistic and varied (10+ users, 50+ audit entries, etc.) to properly demonstrate table features
- The README should include setup instructions for both Vercel demo and Docker production deployment
- Component examples should demonstrate the recommended patterns (Server Components, RHF forms, TanStack Table) so developers can copy and adapt
