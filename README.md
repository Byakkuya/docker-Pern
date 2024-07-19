# Docker boilerplate for PERN Stack Project

This project is a Docker boilerplate for a simple CRUD application using the PERN stack. The PERN stack consists of PostgreSQL, Express, React (Next.js), and Node.js.

## Tech Stack

- **Next.js**: React framework for building web applications.
- **Axios**: Promise-based HTTP client for the browser and Node.js.
- **Express**: Web framework for Node.js.
- **Prisma**: Next-generation ORM for Node.js and TypeScript.
- **PostgreSQL**: Open-source relational database.

## Docker Configuration
### Compose file
```dockerfile 
version: '3.9'

services:
  frontend:
    container_name: frontend
    image: frontend
    build:
      context: ./frontend
      dockerfile: frontend.dockerfile
    ports:
      - 3000:3000
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:4000
    restart: always
    depends_on:
      - backend
  backend:
    container_name: backend
    image: backend
    build:
      context: ./backend
      dockerfile: backend.dockerfile
    ports:
      - 4000:4000
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/postgres?schema=public
  db:
    container_name: db
    image: postgres:12
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - 5432:5432
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata: {}
```

### Backend Dockerfile

```dockerfile
# backend.dockerfile
FROM node:20

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY prisma ./prisma

RUN npx prisma generate

COPY . .

EXPOSE 4000

CMD ["node", "server.js"]
```
## frontend.dockerfile
```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build && ls -l /app/.next


# If using npm comment out above and use below instead
# RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
# Uncomment the following line in case you want to disable telemetry during runtime.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
# set hostname to localhost
ENV HOSTNAME "0.0.0.0"

# server.js is created by next build from the standalone output
# https://nextjs.org/docs/pages/api-reference/next-config-js/output
CMD ["node", "server.js"]
```
# How to Run This Locally
<div style="display: flex;">

<div style="flex: 1; padding-right: 10px;">
  
```bash
#Clone the repository:
git clone https://github.com/your-username/pern-docker-template.git
cd pern-docker-template

#Start Docker containers:
docker-compose up --build
```
</div>
<div style="flex: 1; padding-left: 10px;">
  
Access the application:
- **Frontend**: http://localhost:3000
- **Backend**:  http://localhost:4000
- **DB**: localhost:5432

</div>
</div>




