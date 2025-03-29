# Frontend Documentation

This document provides a comprehensive overview of the frontend project, built with modern web technologies to offer a dynamic and user-friendly interface for the FastAPI backend.

## Introduction

The frontend is developed using a modern stack comprising [Vite](https://vitejs.dev/), [React](https://reactjs.org/), [TypeScript](https://www.typescriptlang.org/), [TanStack Query](https://tanstack.com/query), [TanStack Router](https://tanstack.com/router), and [Chakra UI](https://chakra-ui.com/). This combination is chosen to ensure a fast, efficient, and maintainable frontend application. It focuses on providing a smooth user experience while adhering to best practices in frontend development.

## Architecture Choices

The frontend architecture is designed with the following principles in mind:

*   **Component-Based UI**: React is at the heart of the UI, enabling the creation of reusable and composable components. This approach promotes modularity and maintainability.
*   **Type Safety**: TypeScript is used to ensure type safety throughout the codebase, reducing errors and improving code quality.
*   **Efficient Bundling and Development**: Vite is used as the build tool and development server. Vite's speed and efficiency significantly improve the development experience with fast hot module replacement and optimized builds.
*   **Data Fetching and State Management**: [TanStack Query](https://tanstack.com/query) (formerly React Query) is used for efficient data fetching, caching, and state management of server data. It simplifies data synchronization and reduces boilerplate.
*   **Declarative Routing**: [TanStack Router](https://tanstack.com/router) (formerly React Router v6) is used for declarative and dynamic routing. It allows for type-safe route definitions and simplifies navigation within the application.
*   **UI Library and Styling**: [Chakra UI](https://chakra-ui.com/) is used as the UI component library. It provides a set of accessible, reusable, and composable UI components, and facilitates consistent styling through theme customization.

## Dependencies

Here's an overview of the key dependencies and their roles in the project, as listed in `frontend/package.json`:

### Core Libraries

*   **react**, **react-dom**:  Fundamental libraries for building user interfaces with React.
*   **typescript**:  Adds static typing to JavaScript, improving code maintainability and reducing errors.
*   **vite**:  A fast build tool and development server that enhances development speed and optimizes production builds.

### UI and Styling

*   **@chakra-ui/react**: A component library providing accessible and reusable UI components.
*   **@emotion/react**:  CSS-in-JS library used by Chakra UI for styling React components.
*   **next-themes**:  For theming support, allowing users to switch between different themes (e.g., light and dark mode).
*   **react-icons**:  Provides a wide range of icons to use in the application.

### Data Fetching and State Management

*   **@tanstack/react-query**:  For managing, caching, and synchronizing asynchronous data in React applications.
*   **axios**:  A promise-based HTTP client for making requests to the backend API.
*   **form-data**:  A library to construct FormData objects for making multipart/form-data requests, useful for file uploads or form submissions.

### Routing

*   **@tanstack/react-router**:  For declarative routing in React applications.
*   **@tanstack/router-devtools**:  Devtools for TanStack Router, aiding in debugging and understanding routing behavior.
*   **@tanstack/router-vite-plugin**:  Vite plugin for TanStack Router, enabling seamless integration.

### Development and Testing

*   **@biomejs/biome**:  A fast and comprehensive code formatter and linter, ensuring code consistency and quality.
*   **@hey-api/openapi-ts**:  Tool to generate TypeScript client SDK from OpenAPI specifications, used to create `src/client`.
*   **@playwright/test**:  A framework for end-to-end testing, ensuring application stability and functionality.
*   **@types/\***:  Various type definition packages for TypeScript, providing type checking for JavaScript libraries.
*   **dotenv**:  For loading environment variables from `.env` files, useful for configuration management.

## Project Structure

The project structure, as seen in the attached file list, is organized to separate concerns and improve maintainability. Key directories and files include:

*   **frontend/**: Root directory of the frontend project.
    *   **documentation/**: Contains documentation files, including this `documentation.md`.
    *   **.dockerignore**, **Dockerfile**, **Dockerfile.playwright**: Docker configuration files for containerizing the frontend application.
    *   **.env**:  Environment variables configuration file.
    *   **.gitignore**:  Specifies intentionally untracked files that Git should ignore.
    *   **index.html**:  The main HTML entry point for the application.
    *   **nginx.conf**, **nginx-backend-not-found.conf**: Nginx configuration files for serving the frontend in production.
    *   **openapi-ts.config.ts**: Configuration for generating the API client from OpenAPI specification.
    *   **package.json**, **package-lock.json**:  NPM package manifest and lock file, managing project dependencies and scripts.
    *   **playwright.config.ts**: Configuration file for Playwright end-to-end tests.
    *   **public/**:  Directory for public assets, such as images and static files.
    *   **src/**:  Source code directory.
        *   **assets/**: Static assets used in the application.
        *   **client/**:  Generated API client using `@hey-api/openapi-ts`. Contains core API request logic and generated SDK (`sdk.gen.ts`, `types.gen.ts`, `schemas.gen.ts`).
        *   **components/**:  Reusable React components.
        *   **hooks/**:  Custom React hooks for reusable logic.
        *   **routes/**:  Route definitions and page components for the application, likely using TanStack Router.
        *   **main.tsx**:  Entry point for the React application.
        *   **theme.tsx**:  Custom Chakra UI theme configuration.
        *   **vite-env.d.ts**:  TypeScript declaration file for Vite environment variables.
    *   **test-results/**, **tests/**, **blob-report/**, **playwright-report/**: Directories related to testing and test reports.
    *   **tsconfig.json**, **tsconfig.node.json**, **tsconfig.build.json**: TypeScript configuration files.
    *   **vite.config.ts**:  Vite configuration file.

## How it Works

The frontend application operates as follows:

1.  **Initialization**: When the application starts, `main.tsx` is the entry point. It sets up the React application, likely configures the TanStack Router, Chakra UI `ThemeProvider`, and potentially React Query providers.
2.  **Routing**: TanStack Router handles navigation. Route definitions in the `routes` directory define the different pages and components rendered for each route. Route loaders can be used to fetch data before a route is activated.
3.  **Component Rendering**: React components in the `components` and `routes` directories are responsible for rendering the UI. Chakra UI components are used for styling and structure.
4.  **Data Fetching**: When components need to display data from the backend, they use React Query hooks (e.g., `useQuery`, `useMutation`). These hooks interact with the generated API client in `src/client` to make HTTP requests to the FastAPI backend.
5.  **API Client**: The `src/client` directory contains auto-generated code from the backend's OpenAPI specification. This client provides type-safe functions to interact with the backend API endpoints.  For example, `ItemsService.readItems()` in `sdk.gen.ts` would be used to fetch items from the `/api/v1/items/` endpoint.
6.  **State Management**: React Query manages the state of fetched data, including caching, background updates, and re-fetching. For UI-specific state, React's built-in state management (`useState`, `useReducer`) or context API might be used within components.
7.  **Themeing**: Chakra UI's theming system, configured in `theme.tsx`, ensures a consistent look and feel across the application. The `useTheme` hook from `next-themes` allows components to access and modify the current theme (e.g., switching between light and dark mode).
8.  **Building and Deployment**: Vite is used to build the application for production. The build process optimizes and bundles the code into static assets, which are then served by Nginx in a Docker container, as defined by the `Dockerfile` and `nginx.conf`.

## Development

To work on the frontend, follow these steps:

### 1. Prerequisites

*   **Node.js and npm**: Ensure you have Node.js (version specified in `.nvmrc`, currently v20) and npm installed. It's recommended to use [nvm](https://github.com/nvm-sh/nvm) or [fnm](https://github.com/Schniz/fnm) for managing Node.js versions.
*   **Backend API Running**: For full functionality, the FastAPI backend should be running, as the frontend will interact with it. Refer to the backend documentation to set it up.

### 2. Setup Development Environment

1.  **Navigate to the frontend directory**:

    ```bash
    cd frontend
    ```

2.  **Install Node.js version (if using nvm/fnm)**:

    ```bash
    # If using fnm
    fnm use

    # If using nvm
    nvm use
    ```

3.  **Install dependencies**:

    ```bash
    npm install
    ```

### 3. Run Development Server

Start the Vite development server with hot module replacement:

```bash
npm run dev
```

This will start the frontend development server, usually at `http://localhost:5173/`. Open this URL in your browser to view the application. Any changes you make to the code in `src/` will be automatically reflected in the browser.

### 4. Code Formatting and Linting

The project uses [Biome](https://biomejs.dev/) for code formatting and linting. Before committing code, run:

```bash
npm run lint
```

This command will automatically format and lint your code according to the project's Biome configuration (`biome.json`).

### 5. Building for Production

To create a production build, run:

```bash
npm run build
```

This command compiles the TypeScript code, bundles the application using Vite, and outputs the optimized static assets to the `dist` directory. These assets are ready to be served by a web server like Nginx.

### 6. Running End-to-End Tests

End-to-end tests are written using Playwright. To run the tests:

1.  **Ensure Docker Compose stack is running**: The tests might depend on a running backend and database. Start the stack using `docker compose up -d --wait backend` from the project root.
2.  **Run Playwright tests**:

    ```bash
    npx playwright test
    ```

    Or to run tests in UI mode:

    ```bash
    npx playwright test --ui
    ```

### 7. Extending and Modifying the Frontend

*   **Adding New Pages/Routes**: Define new routes in the `src/routes` directory using TanStack Router. Create new React components for these pages, placing them either in `src/routes` or `src/components`.
*   **Creating Components**: Develop reusable UI components in the `src/components` directory. Utilize Chakra UI components for styling and accessibility. Follow Chakra UI best practices for customization and theming.
*   **Data Fetching**: Use React Query hooks in your components to fetch data from the backend API. Import and use the generated API client functions from `src/client/sdk.gen.ts`. Implement proper error handling and loading states. Refer to React Query best practices for caching and invalidation.
*   **Customizing Theme**: Modify the Chakra UI theme in `src/theme.tsx` to change the application's visual appearance. You can customize colors, typography, breakpoints, and component styles. Follow Next Themes best practices if implementing theme switching.
*   **Replacing Parts**: If you need to replace libraries or major parts of the frontend, consider the impact on the architecture and dependencies. For example, replacing Chakra UI would require rewriting component styling and potentially accessibility features. Replacing React Query would necessitate a new data fetching and state management strategy. Ensure thorough testing after making significant changes.

### 8. Generating API Client

Whenever the backend API schema changes (e.g., after modifying the FastAPI backend), you need to regenerate the frontend API client:

1.  **Start the Docker Compose stack** to ensure the backend is running and serving the OpenAPI schema.
2.  **Run the client generation script**:

    ```bash
    npm run generate-client
    ```

    This script uses `openapi-ts` to fetch the OpenAPI schema from the backend and regenerate the client code in `src/client`.
3.  **Commit the changes** to include the updated client in your version control.

By following these guidelines, developers can effectively work on, extend, and maintain the frontend application, ensuring a robust and user-friendly interface for the FastAPI backend.
