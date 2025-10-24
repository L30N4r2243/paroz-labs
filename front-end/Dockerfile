# Etapa de construcción
FROM node:18-alpine AS builder

# Instalar pnpm globalmente
RUN npm install -g pnpm

WORKDIR /app

# Copiamos package files
COPY package.json pnpm-lock.yaml* ./

# Instalamos todas las dependencias (incluyendo devDependencies)
RUN pnpm install --frozen-lockfile

# Copiamos el código fuente
COPY . .

# Construimos la aplicación
RUN pnpm run build

# Etapa de producción
FROM node:18-alpine AS runner

# Instalar pnpm globalmente
RUN npm install -g pnpm

WORKDIR /app

# Copiamos package.json y pnpm-lock.yaml
COPY --from=builder /app/package.json ./
COPY --from=builder /app/pnpm-lock.yaml* ./

# Instalamos solo dependencias de producción
RUN pnpm install --prod --frozen-lockfile

# Copiamos el build desde la etapa anterior
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

# Crear usuario no-root para seguridad
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Cambiar ownership de los archivos
RUN chown -R nextjs:nodejs /app
USER nextjs

# Exponemos el puerto 3000
EXPOSE 3000

# Levantar Next.js en modo producción
CMD ["pnpm", "run", "start"]
