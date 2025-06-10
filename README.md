# C++ Code Quality Platform

A web platform for analyzing and improving the quality of C++ projects. It is built with Next.js, Tailwind CSS and TypeScript.

## Features

- Next.js 15 with the App Router
- TypeScript and ESLint for a robust developer experience
- Tailwind CSS for styling
- Clerk authentication
- DrizzleORM for database access
- Vitest and Playwright for testing
- Internationalization with next-intl
- Sentry integration for error monitoring

## Requirements

- Node.js 20+
- npm

## Getting Started

Clone the repository and install dependencies:

```bash
git clone https://github.com/thanhtrungnguyen/cpp-code-quality-platform.git
cd cpp-code-quality-platform
npm install
```

Environment variables are stored in the `./.envs` directory. Copy the files under `./.envs/.local` and provide your own values.

Run the development server:

```bash
npm run dev
```

You can also use Docker Compose:

```bash
docker compose -f docker-compose.local.yml up
```

Build the application for production:

```bash
npm run build
npm start
```

## Project Structure

```text
.
├── src       # Application source
├── public    # Static assets
├── tests     # Test suites
├── compose   # Docker Compose files
└── ...
```

## License

[MIT](LICENSE)
