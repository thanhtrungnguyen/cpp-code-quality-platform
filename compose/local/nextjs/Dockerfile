FROM node:lts-bookworm-slim AS base

# 1. Install dependencies only when needed
FROM base AS deps

ARG BUILD_ENVIRONMENT=local
ARG APP_HOME=/app

WORKDIR ${APP_HOME}
# Install apt packages
RUN apt-get update && apt-get install --no-install-recommends -y

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* .npmrc* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm i; \
  else echo "Lockfile not found." && exit 1; \
  fi

# 2. Rebuild the source code only when needed
FROM base AS builder
WORKDIR ${APP_HOME}
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# Build the Next.js app to generate .next/standalone and .next/static
# RUN if [ -f yarn.lock ]; then yarn build; \
#     elif [ -f package-lock.json ]; then npm run build; \
#     elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm run build; \
#     else echo "Lockfile not found." && exit 1; \
#     fi

# 3. Production image, copy all the files and run next
FROM base AS runner
WORKDIR ${APP_HOME}

# Ensure sudoers.d directory exists before writing to it
RUN mkdir -p /etc/sudoers.d \
  && groupadd --gid 1001 dev-user \
  && useradd --uid 1001 --gid dev-user --shell /bin/bash --create-home dev-user \
  && echo dev-user ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/dev-user \
  && chmod 0440 /etc/sudoers.d/dev-user

# Install required system dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
  postgresql-client \
  wait-for-it \
  # Translations dependencies
  gettext \
  # cleaning up unused files
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/*

# Copy build output for production runner
# COPY --from=builder /app/public ./public
# COPY --from=builder --chown=dev-user:dev-user /app/.next/standalone ./
# COPY --from=builder --chown=dev-user:dev-user /app/.next/static ./.next/static

COPY ./compose/production/nextjs/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint

COPY ./compose/local/nextjs/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start

USER dev-user

EXPOSE 3000

ENV PORT=3000

ENTRYPOINT ["/entrypoint"]