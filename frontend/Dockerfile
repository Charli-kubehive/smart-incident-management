# ---- build stage ----
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi
COPY . .
# API address is baked into the bundle at build time (only used if the code reads VITE_API_URL)
ARG VITE_API_URL=http://localhost:8088
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build

# ---- runtime stage ----
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
