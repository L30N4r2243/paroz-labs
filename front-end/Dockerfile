# Etapa de construcción
FROM node:18-alpine AS builder

WORKDIR /app

# Copiamos package files
COPY package*.json ./

# Instalamos todas las dependencias (incluyendo devDependencies)
RUN npm ci

# Copiamos el código fuente
COPY . .

# Construimos la aplicación
RUN npm run build

# Etapa de producción
FROM node:18-alpine AS runner

WORKDIR /app

# Copiamos package.json primero
COPY --from=builder /app/package*.json ./

# Instalamos solo dependencias de producción
RUN npm install --omit=dev

# Copiamos el build desde la etapa anterior
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

# Exponemos el puerto 3000
EXPOSE 3000

# Levantar Next.js en modo producción
CMD ["npm", "run", "start"]
