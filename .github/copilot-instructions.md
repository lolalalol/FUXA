# Copilot Instructions for FUXA

## Project Overview

FUXA is a web-based SCADA/HMI platform for industrial automation, IoT, and real-time process visualization. It provides:

- **Backend**: Node.js/Express server with TypeScript supporting multiple industrial protocols (Modbus, OPC-UA, MQTT, Siemens S7, BACnet IP, Ethernet/IP, ODBC, etc.)
- **Frontend**: Angular 18 with Material Design components for dashboards and visual editor
- **Data Storage**: Built-in historian (DAQ) with SQLite, InfluxDB, and time-series database support
- **Real-time Communication**: Socket.io for live updates
- **Cross-platform**: Linux, Windows, macOS, Docker, Raspberry Pi

## Repository Structure

```
server/          # Node.js backend (Express, TypeScript)
├── api/         # REST API modules (projects, auth, users, alarms, plugins, etc.)
├── runtime/     # Core runtime engine
├── integrations/# Industrial protocol drivers
└── test/        # Mocha tests

client/          # Angular 18 frontend
├── src/
│   ├── app/     # Angular components and services
│   └── assets/  # Static files and themes
└── dist/        # Production build output (DO NOT COMMIT)

app/            # Electron desktop application
├── electron/   # Electron main process
└── headless/   # Headless server binaries

docs/           # MkDocs documentation (Material theme)
```

## Build, Test, and Lint Commands

### Backend (Server)

```bash
cd server

# Install dependencies
npm install

# Run development server (HTTP on port 1881 by default)
npm start

# Compile TypeScript
npm run build

# Run Mocha tests
npm test

# Run a specific test file
npm test -- test/path/to/test.js

# Run tests with grep filter
npm test -- --grep "pattern"
```

### Frontend (Client)

```bash
cd client

# Install dependencies
npm install

# Development server (ng serve on port 4200)
npm start

# Development server on all network interfaces
npm start:lan

# Demo configuration
npm run demo

# Production build
npm run build

# ESLint checks
npm run lint

# End-to-end tests
npm run e2e
```

### Documentation

```bash
# Install MkDocs dependencies (Python)
pip install "mkdocs<2.0"
pip install mkdocs-material

# Serve docs locally
mkdocs serve

# Access at: http://127.0.0.1:8000/FUXA/
```

### Docker

```bash
# Build and run with Docker Compose (persistent storage)
docker compose up -d

# Access at: http://localhost:1881
```

## High-Level Architecture

### Backend Architecture

1. **Express Server** (`server/main.js`)
   - Entry point: Initializes HTTP/HTTPS server with Socket.io
   - Configurable port (default 1881) via CLI arguments
   - JWT-based authentication with optional refresh tokens

2. **API Layer** (`server/api/`)
   - Modular route handlers for each domain
   - Rate limiting: 100 requests per 5 minutes (except `/api/version`)
   - Authentication middleware checks API keys or JWT tokens
   - All endpoints require authentication unless configured as guest

3. **Runtime Engine** (`server/runtime/`)
   - Core FUXA runtime managing projects and devices
   - Event system for real-time communication
   - Logger (Winston) for application logging
   - Utilities for common operations

4. **Device Integration** (`server/integrations/`)
   - Industrial protocol drivers (Modbus, OPC-UA, MQTT, S7, etc.)
   - Node-RED integration
   - ODBC for external database connectivity

5. **Data Historian (DAQ)**
   - Time-series data storage
   - Support for SQLite (default), InfluxDB, QuestDB, TDengine
   - Configured via `daqstore` settings

### Frontend Architecture

1. **Angular 18 Application** (`client/src/app/`)
   - Component-based architecture with Material Design (v18)
   - Service layer for API communication and state management
   - Real-time updates via Socket.io
   - **Current Issue**: All modules bundled into single main.js (~8MB), inefficient for simple viewers

2. **Viewer vs Editor Modules**
   - **Essential for Viewer** (must be eager-loaded):
     - `home`, `view`, `fuxa-view` - Dashboard viewing
     - `gauges` - Widget rendering system
     - `header`, `sidenav` - Navigation UI
     - `_services` (HmiService, ProjectService, AppService) - Core data
     - `device-adapter` - Real-time tag updates via Socket.io
   
   - **Non-essential for Viewer** (lazy-loadable):
     - `editor` (large: editor configs, SVG selector, property dialogs)
     - `device` (device/tag management - admin feature)
     - `scripts`, `scheduler` - Script execution and automation
     - `alarms`, `notifications` - Alarm/event management
     - `reports` - Report generation
     - `users`, `apikeys` - User/security management
     - `language` - I18n configuration
     - `lab`, `tester`, `plugins` - Development/debug tools
     - `integrations/node-red` - Node-RED flows
     - `logs-view` - System logs
     - `maps` - Map-based locations
     - `resources` - Asset management

3. **Bundle Optimization Strategy**
   - **main.js** (eager): ~2MB with core viewer + gauges + navigation
   - **editor.js** (lazy): ~3MB for editor module + configs
   - **admin.js** (lazy): ~1.5MB for device/users/scripts management
   - **tools.js** (lazy): ~0.5MB for reports, logs, plugins
   - **integrations.js** (lazy): ~1MB for Node-RED and external connectors

4. **Communication**
   - Socket.io client for real-time data push
   - HTTP REST API for configuration and data retrieval

## Key Conventions

### Coding Standards

1. **Indentation**: 4 spaces (no tabs) throughout codebase
2. **Braces**: Opening brace on same line, closing brace on its own line
   ```javascript
   if (condition) {
       // code
   }
   ```

### TypeScript Configuration

- **Server**: Strict mode enabled, target ES2020, CommonJS modules
- **Client**: Strict mode enabled, target ES2020, ESM modules
- Both use `tsconfig.json` with `strict: true` for full type checking

### Frontend (Angular/ESLint)

- ESLint extends `@angular-eslint/recommended`
- Component/directive selector rules are disabled (flexible naming)
- Unused imports flagged as errors
- Curly braces required for multi-line statements

### API Design

1. **Authentication**:
   - JWT tokens stored in `Authorization: Bearer <token>` header
   - API keys supported via `X-API-Key` header
   - Guest access enabled when `secureEnabled: false`

2. **Request/Response**:
   - Max payload size configurable (default 100MB)
   - JSON request/response format
   - HTTP status codes: 400 (bad request), 401 (unauthorized), 403 (forbidden), 404 (not found), 413 (payload too large), 500 (server error)

3. **Settings**:
   - Configuration stored in `settings.default.js` and user settings file
   - Sensitive data (JWT secret, passwords) never exposed in API responses
   - Settings changes trigger runtime restart

### Project Configuration

- Projects stored in `server/_appdata/`
- DAQ (historian) data in `server/_db/`
- Logs in `server/_logs/`
- User images in `server/_images/`

### Git Conventions

**DO NOT COMMIT:**
- `client/dist/` - Frontend production build
- `server/dist/` - Backend compiled output
- `node_modules/` in any directory
- Generated files or build artifacts

**Only commit:**
- Source code (TypeScript, JavaScript, SCSS, HTML)
- Configuration files (package.json, tsconfig.json, etc.)
- Documentation

Pull requests must contain only source code changes. Build artifacts should be excluded via `.gitignore`.

## Development Workflow

### Client Bundle Optimization (Lazy-Loading Strategy)

The current client bundle (main.js ~8MB) includes all modules (editor, admin, tools) even for simple viewers. Implement lazy-loading to reduce initial bundle size:

**Step 1: Create Feature Modules** (each with own routing)
```
editor/
├── editor.module.ts (NgModule)
├── editor-routing.module.ts (with lazy routes)
└── components/

device/
├── device.module.ts
├── device-routing.module.ts
└── components/

admin/  (new)
├── admin.module.ts
├── admin-routing.module.ts
└── components/ (users, apikeys, scripts, scheduler, alarms)

tools/  (new)
├── tools.module.ts
├── tools-routing.module.ts
└── components/ (reports, logs, language, plugins, tester)

integrations/
├── integrations.module.ts
├── integrations-routing.module.ts
└── components/ (node-red, lab)
```

**Step 2: Update app-routing.ts** with lazy-loaded routes
```typescript
const appRoutes: Routes = [
    // Eager routes (always loaded)
    { path: '', component: HomeComponent },
    { path: 'home/:viewName', component: HomeComponent },
    { path: 'view', component: ViewComponent },
    
    // Lazy routes (loaded on-demand)
    { path: 'editor', loadChildren: () => import('./editor/editor.module').then(m => m.EditorModule) },
    { path: 'device', loadChildren: () => import('./device/device.module').then(m => m.DeviceModule) },
    { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) },
    { path: 'tools', loadChildren: () => import('./tools/tools.module').then(m => m.ToolsModule) },
    { path: 'flows', loadChildren: () => import('./integrations/integrations.module').then(m => m.IntegrationsModule) },
    
    { path: '**', redirectTo: '' }
];
```

**Step 3: Refactor AppModule** to declare only eager components
- Remove editor, device, admin, tools component imports
- Keep only: HomeComponent, ViewComponent, HeaderComponent, SidenavComponent
- Keep shared services in `_services` but move module-specific services into their feature modules

**Step 4: Update angular.json** for differential loading (optional optimization)
```json
{
  "build": {
    "options": {
      "optimization": true,
      "outputHashing": "all",
      "sourceMap": false,
      "namedChunks": false,
      "aot": true,
      "buildOptimizer": true,
      "vendorChunk": false
    }
  }
}
```

**Expected Results After Optimization:**
- main.js: ~2MB (core viewer + gauges)
- editor.js: ~3MB (lazy-loaded on demand)
- admin.js: ~1.5MB (lazy-loaded)
- tools.js: ~0.5MB (lazy-loaded)
- integrations.js: ~1MB (lazy-loaded)
- **Initial load**: ~40-50% smaller for view-only use cases

**Testing Lazy-Loading:**
1. Build production: `npm run build`
2. Check dist/: Look for chunk files (e.g., `editor-xxxxx.js`)
3. Open DevTools Network tab → Route to `/editor` → Confirm chunk loads
4. Verify view-only route (`/view`, `/home`) doesn't fetch editor chunk

**Key Files to Modify:**
- `client/src/app/app.routing.ts` - Add loadChildren
- `client/src/app/app.module.ts` - Remove feature module imports
- Create `*/xyz-routing.module.ts` in each feature module
- Update `angular.json` - Ensure optimization flags enabled

**Best Practices:**
- Keep `_services`, `_models`, `_helpers` in root (shared across all modules)
- Each feature module should have private services in its folder
- Use `unsubscribe()` pattern or `takeUntil` to prevent memory leaks in lazy modules
- Pre-load critical modules (e.g., editor) on user interaction: `Router.ngPreloadAllModules()`

### Adding a New API Endpoint

1. Create a new module in `server/api/` (e.g., `server/api/myfeature/index.js`)
2. Export `init(runtime, authMiddleware, verifyGroups)` and `app()` functions
3. Register in `server/api/index.js`: call `myApi.init(...)` and `apiApp.use(myApi.app())`
4. All endpoints inherit rate limiting and authentication

### Adding a New Frontend Component

1. Generate with Angular CLI: `ng generate component path/to/component`
2. Use Material components from `@angular/material`
3. Communicate with backend via services (HTTP + Socket.io)
4. Follow Angular 18 style guide (OnPush change detection recommended)

### Running Full-Stack Debug

```bash
# Terminal 1: Backend
cd server && npm install && npm start

# Terminal 2: Frontend
cd client && npm install && npm start

# Terminal 3: VS Code debugger
# Use Debug 'Server & Client' configuration (see .vscode/launch.json)
```

## Important Notes

- **Node.js 18 LTS** is recommended (required: ≥14.x)
- On Linux systems with Node 18, native module builds may require additional build tools
- If you don't need Siemens S7 support, remove `node-snap7` from `server/package.json` to simplify installation
- If you don't need ODBC support, remove `odbc` from `server/package.json`
- The frontend proxy configuration is in `client/proxy.conf.json` for development
- Socket.io version must match between server (`socket.io 4.8.1`) and client (`socket.io-client 4.8.1`)
- Electron builds available in GitHub Actions artifacts (Electron workflow)
- Headless portable binaries for embedded devices available in GitHub Actions artifacts (Headless workflow)

### Client Bundle Size Optimization

The main.js file currently reaches ~8MB because all modules (editor, admin, tools) are bundled together. To enable lightweight viewer-only deployments, implement lazy-loading for non-essential modules (see **Client Bundle Optimization** section in Development Workflow above).

**Why This Matters:**
- Viewer deployments: Only need visualization → 40-50% smaller bundle
- Embedded/IoT devices with limited bandwidth benefit from smaller bundles
- Faster initial page load and reduced memory footprint
- Editor and admin tools loaded only when needed

**Implementation Priority:**
1. Move editor module to lazy-loading (saves ~3MB)
2. Move admin features (users, scripts, alarms) to lazy-loading (saves ~1.5MB)
3. Move tools (reports, logs, plugins) to lazy-loading (saves ~0.5MB)
4. Keep integrations lazy (Node-RED already semi-isolated)

**Result:** View-only deployments could reduce to 2-2.5MB initial bundle, ~70% savings
