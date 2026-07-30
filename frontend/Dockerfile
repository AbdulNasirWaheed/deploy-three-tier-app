# frontend/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Accept build argument for API URL
ARG REACT_APP_API_URL
ENV REACT_APP_API_URL=$REACT_APP_API_URL

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# ---- production stage ----
FROM node:18-alpine

RUN npm install -g serve
COPY --from=builder /app/build /build
RUN adduser -S appuser
USER appuser
EXPOSE 3000
CMD ["serve", "-s", "/build"]
