# Build stage
FROM node:20-alpine AS build

WORKDIR /app

# Install exact dependencies reproducibly
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build

# Runtime
FROM nginx:stable-alpine

# Create a non-privileged user and ensure ownership of web content
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=build /app/dist /usr/share/nginx/html
RUN chown -R appuser:appgroup /usr/share/nginx/html

EXPOSE 80

# Note: keep the default user so nginx can bind port 80; run as unprivileged user only if configured to bind a non-privileged port
CMD ["nginx", "-g", "daemon off;"]