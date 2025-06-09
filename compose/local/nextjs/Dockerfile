# syntax=docker.io/docker/dockerfile:1

FROM node:lts-bookworm-slim AS base

# Base system setup
RUN apt-get update && apt-get install --no-install-recommends -y \
    libc6 \
    sudo \
    bash \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Add wait-for-it script for Postgres connection handling
ADD https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh /wait-for-it
RUN chmod +x /wait-for-it

# Create dev user at the base level so it's available in all stages
# Use -o flag for useradd to allow non-unique UID and handle existing UID gracefully
RUN groupadd -f --gid 1000 dev-user \
    && if ! id -u dev-user &>/dev/null; then useradd -o --uid 1000 --gid dev-user --shell /bin/bash --create-home dev-user || true; fi \
    && echo "dev-user ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/dev-user \
    && chmod 0440 /etc/sudoers.d/dev-user

# Install dependencies only when needed
FROM base AS deps

WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* .npmrc* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# For development, we don't need to build the app
# Just prepare the development environment
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Disable telemetry during build
ENV NEXT_TELEMETRY_DISABLED=1

# No build step for development - we'll use the dev server instead

# Development image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=development

# Install additional development tools
RUN apt-get update && apt-get install --no-install-recommends -y \
  git bash-completion nano ssh \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# We already installed the necessary dependencies in the base stage

# Setup startup scripts
COPY ./compose/production/nextjs/entrypoint /entrypoint
COPY ./compose/local/nextjs/start /start
RUN chmod +x /entrypoint \
    && chmod +x /start \
    && chown dev-user:dev-user /entrypoint /start

# Copy only the essential files for development
COPY --from=builder --chown=dev-user:dev-user /app/. ./

USER dev-user

EXPOSE 3000

ENV PORT=3000

# Basic environment setup
ENV HOSTNAME="0.0.0.0"

# Use the entrypoint script to handle database connection before starting
ENTRYPOINT ["/entrypoint"]
CMD ["/start"]