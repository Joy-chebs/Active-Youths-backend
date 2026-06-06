# ── Stage 1: Build ────────────────────────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files first
COPY package*.json ./
COPY tsconfig.json ./
COPY prisma.config.ts ./

# Copy prisma schema BEFORE installing dependencies
COPY prisma ./prisma

# Install ALL dependencies (including devDependencies for build)
RUN npm ci

# Generate Prisma client inside the container (critical!)
RUN npx prisma generate

# Now copy source code
COPY src ./src

# Build TypeScript
RUN npm run build

# ── Stage 2: Production ───────────────────────────────────────────────────────
FROM node:20-alpine AS production

WORKDIR /app

ENV NODE_ENV=production

COPY package*.json ./
COPY prisma ./prisma

# Install production dependencies only
RUN npm ci --omit=dev

# Generate Prisma client again in production stage
RUN npx prisma generate

# Copy built output from builder
COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/src/server.js"]