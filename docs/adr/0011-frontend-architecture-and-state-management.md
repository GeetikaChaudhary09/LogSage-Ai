
# ADR-0011: Frontend Architecture and State Management

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI requires a modern, responsive and maintainable frontend for developers and organization members.

The frontend will provide dashboards, log exploration, anomaly analysis, incident management, alert configuration, project management and organization settings.

The application must communicate with the Java Spring Boot backend through documented APIs.

The frontend architecture must support reusable components, predictable state management, form validation, protected navigation and consistent loading, error and empty states.

## Decision

LogSage AI will use React with TypeScript and Vite.

The frontend will follow a feature-based architecture with reusable shared components and a centralized API communication layer.

The initial technology choices will include:

- React for the component-based UI.
- TypeScript for static typing.
- Vite for development and production builds.
- React Router for navigation.
- TanStack Query for server-state management.
- React Hook Form for form management.
- Zod for schema validation.
- Recharts for data visualization.
- Lucide React for icons.
- Tailwind CSS for styling.

## Application Architecture

The frontend will be organized into distinct responsibilities:

### Application Layer

Responsible for:

- Application initialization.
- Global providers.
- Routing.
- Authentication context.
- Global error handling.
- Theme configuration.

### Feature Layer

Each major product capability will have its own feature module.

Initial features include:

- Authentication.
- Onboarding.
- Dashboard.
- Log Explorer.
- Anomaly Detection.
- Incident Management.
- Alert Management.
- Services.
- Projects.
- API Keys.
- Organization Settings.

Feature modules should contain their own components, hooks, API functions, types and validation schemas where appropriate.

### Shared Layer

Contains reusable application-wide functionality:

- UI components.
- Layout components.
- Navigation.
- Common hooks.
- API client.
- Utility functions.
- Shared types.
- Theme tokens.
- Error and loading components.

## Feature-Based Folder Structure

The frontend should follow a structure similar to:

```text
frontend/
├── public/
├── src/
│   ├── app/
│   │   ├── providers/
│   │   ├── router/
│   │   └── App.tsx
│   │
│   ├── assets/
│   │
│   ├── components/
│   │   ├── ui/
│   │   ├── layout/
│   │   └── common/
│   │
│   ├── features/
│   │   ├── auth/
│   │   ├── onboarding/
│   │   ├── dashboard/
│   │   ├── logs/
│   │   ├── anomalies/
│   │   ├── incidents/
│   │   ├── alerts/
│   │   ├── services/
│   │   ├── projects/
│   │   ├── api-keys/
│   │   └── settings/
│   │
│   ├── hooks/
│   ├── lib/
│   ├── services/
│   ├── types/
│   ├── utils/
│   ├── styles/
│   └── main.tsx
│
├── .env.example
├── package.json
├── tsconfig.json
└── vite.config.ts
```

The structure may evolve as implementation requirements become clearer.

## Routing

React Router will manage frontend navigation.

The application will support:

- Public routes.
- Authentication routes.
- Protected application routes.
- Organization-scoped routes.
- Project-scoped routes.
- Not-found routes.

Protected routes must check authentication state before rendering protected application pages.

The frontend must not treat route protection as a replacement for backend authorization.

Unauthorized API responses must be handled consistently.

## State Management

The frontend will distinguish between server state and client-side UI state.

### Server State

TanStack Query will manage data retrieved from backend APIs.

Examples:

- Dashboard metrics.
- Log search results.
- Incident records.
- Alert rules.
- Service information.
- Project information.
- Organization members.

It will support:

- Data fetching.
- Caching.
- Background refetching.
- Pagination.
- Query invalidation.
- Mutation handling.
- Loading and error states.

### Client UI State

Local React state and Context will manage shared UI concerns where necessary.

Examples:

- Sidebar visibility.
- Theme preference.
- Active organization selection.
- Modal visibility.
- Temporary filters.
- User interface preferences.

A global state library will not be introduced unless actual application complexity justifies it.

Server data must not be unnecessarily duplicated in global client state.

## API Communication Layer

The frontend will use a centralized API client based on Axios or typed Fetch.

The API layer will handle:

- Backend base URL configuration.
- Request and response typing.
- Authentication headers.
- Standardized error handling.
- Request cancellation where appropriate.
- Timeout configuration.
- Response parsing.

Feature-specific API modules will call the centralized client rather than duplicating request configuration.

The backend remains the source of truth for application data.

## Authentication Integration

The frontend will integrate with the backend authentication system defined in ADR-0007.

It must support:

- Registration.
- Login.
- Logout.
- Access token handling.
- Refresh token workflow.
- Expired session handling.
- Protected navigation.

Refresh-token transport and storage must follow the backend security design.

Sensitive credentials must not be placed in URLs or exposed in browser logs.

## Forms and Validation

React Hook Form will manage form state and submission.

Zod will define reusable validation schemas.

Forms must support:

- Required field validation.
- Type validation.
- Appropriate length limits.
- Clear validation messages.
- Submission loading states.
- Backend validation error handling.
- Prevention of accidental duplicate submissions.

Frontend validation improves usability but does not replace backend validation.

## UI Component System

The application will use a consistent reusable component system built with Tailwind CSS and shadcn/ui-style patterns.

Reusable components may include:

- Buttons.
- Inputs.
- Selects.
- Dialogs.
- Dropdown menus.
- Tables.
- Cards.
- Tabs.
- Badges.
- Tooltips.
- Toast notifications.
- Pagination.
- Skeleton loaders.

Components must have consistent visual behaviour, accessibility and responsive layouts.

## Design System

LogSage AI will use a dark-first interface with support for a light theme.

The design system will define:

- Color tokens.
- Typography.
- Spacing.
- Border radius.
- Shadows.
- Component variants.
- Responsive breakpoints.
- Focus states.
- Semantic severity colors.

Severity indicators must remain consistent across logs, incidents and alerts.

The interface should prioritize readability and efficient developer workflows.

## Data Visualization

Recharts will be used for dashboard charts and operational visualizations.

Possible visualizations include:

- Log volume over time.
- Error and warning trends.
- Severity distribution.
- Service-level activity.
- Anomaly trends.
- Incident trends.

Charts must use actual backend data.

The frontend must not invent metrics or display fabricated production activity.

When data is unavailable, the UI must clearly distinguish unavailable data from a genuine zero value.

## Loading, Error and Empty States

Every data-driven page must handle:

### Loading State

Display appropriate loading indicators or skeletons while requests are pending.

### Empty State

Show a meaningful empty state when no matching records exist.

### Error State

Display understandable errors and provide retry actions where appropriate.

### Success State

Show confirmation for successful user operations such as creating a project or updating an alert rule.

The interface must never silently replace failed requests with fabricated values.

## Search, Filtering and Pagination

The frontend will support server-driven search and filtering for large datasets.

Examples:

- Log severity.
- Service.
- Environment.
- Time range.
- Full-text search.
- Trace ID.
- Request ID.
- Incident status.
- Alert severity.

Pagination and sorting must be integrated with backend API capabilities.

Large datasets must not be unnecessarily loaded entirely into browser memory.

## Accessibility and Responsive Design

The frontend must support:

- Semantic HTML.
- Keyboard navigation.
- Visible focus indicators.
- Accessible form labels.
- Appropriate color contrast.
- Screen-reader-friendly controls.
- Responsive desktop, tablet and mobile layouts.

Interactive components must remain usable across supported screen sizes.

## Performance

The frontend should use:

- Route-level code splitting where beneficial.
- Lazy loading for large feature modules.
- Query caching.
- Efficient component rendering.
- Pagination for large datasets.
- Optimized production builds.
- Appropriate asset optimization.

Performance improvements must not compromise correctness or accessibility.

## Configuration

Frontend configuration will use environment variables.

The initial configuration must include:

- Backend API base URL.
- Application environment.
- Other non-secret public configuration where required.

Frontend environment variables must never contain private backend secrets, database credentials, JWT signing keys or raw API key administration secrets.

An `.env.example` file must document required frontend configuration.

## Testing Requirements

Planned frontend tests:

- Application rendering.
- Public and protected routing.
- Authentication forms.
- Form validation.
- API client behaviour.
- Query loading and error states.
- Dashboard rendering with real API responses.
- Log filtering and pagination.
- Incident status interactions.
- Alert rule forms.
- Organization and project isolation in UI navigation.
- Responsive component behaviour.
- Accessibility checks for critical workflows.

Integration and end-to-end tests must cover important user journeys.

## Alternatives Considered

### Plain JavaScript

Rejected in favour of TypeScript because static typing improves maintainability and helps catch integration errors.

### One Large Component Structure

Rejected because it would make a growing application difficult to maintain.

### Global State for All Data

Rejected because server data is better managed through a dedicated server-state library.

### Separate UI Framework for Every Feature

Rejected because a consistent shared component system improves maintainability and user experience.

## Consequences

### Positive

- Clear feature boundaries.
- Reusable UI components.
- Better type safety.
- Predictable server-state management.
- Consistent API communication.
- Easier testing.
- Improved responsiveness and accessibility.
- Better long-term maintainability.

### Negative

- Initial project structure requires additional setup.
- Multiple frontend libraries require dependency management.
- Query caching and invalidation require careful design.
- Authentication state must remain synchronized with backend sessions.

## Implementation Notes

- Use React, TypeScript and Vite.
- Follow feature-based folder organization.
- Use React Router for navigation.
- Use TanStack Query for server state.
- Use React Hook Form and Zod for forms.
- Centralize API communication.
- Build reusable accessible components.
- Use actual backend data for all operational metrics.
- Test loading, empty, error and success states.
- Keep frontend authorization separate from backend security enforcement.

## Review Conditions

Revisit this decision if:

- Frontend complexity requires a different state-management strategy.
- Performance testing identifies architectural bottlenecks.
- Accessibility requirements change.
- A major frontend framework migration is considered.
- The API integration model changes.
