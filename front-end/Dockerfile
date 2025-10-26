# Etapa de construcción (Builder Stage)
# Usamos node:20-alpine por ser una versión LTS más reciente y segura
FROM node:20-alpine AS builder

# 1. Configuración inicial
WORKDIR /app
# Instalar pnpm globalmente. (Necesario para usar el comando pnpm)
RUN npm install -g pnpm

# 2. Copiamos package files Y EL ARCHIVO DE BLOQUEO GENERADO LOCALMENTE
# La dependencia de la red se minimiza ya que el lockfile existe.
COPY package.json pnpm-lock.yaml ./

# 3. Instalamos todas las dependencias (con --frozen-lockfile para determinismo)
# ESTE PASO AÚN NECESITA CONEXIÓN A INTERNET, PERO ES VITAL para generar .pnpm-store.
RUN pnpm install --frozen-lockfile

# 4. Copiamos el código fuente e ignoramos archivos innecesarios
COPY . .

# 5. Construimos la aplicación
# Si pnpm install falló por timeout en el paso 3, este paso nunca se ejecuta.
RUN pnpm run build

# Etapa de producción (Runner Stage)
# Usamos node:20-alpine de nuevo
FROM node:20-alpine AS runner

# 1. Configuración inicial
WORKDIR /app
# Instalar pnpm globalmente (Necesario para el CMD)
RUN npm install -g pnpm

# 2. Copiamos ÚNICAMENTE los archivos esenciales del build anterior
# Ya no necesitamos copiar package.json y pnpm-lock.yaml ni hacer pnpm install - de nuevo
# Copiamos la carpeta node_modules completa desde la etapa builder
COPY --from=builder /app/node_modules ./node_modules
# Copiamos el build
COPY --from=builder /app/.next ./.next
# Copiamos los archivos públicos
COPY --from=builder /app/public ./public

# 3. Copiamos archivos de configuración esenciales para producción (si los tienes)
# COPY --from=builder /app/next.config.js ./
# COPY --from=builder /app/package.json ./ # Solo necesario si el script start lo requiere

# 4. Crear usuario no-root para seguridad
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# 5. Cambiar ownership de los archivos para el usuario no-root
RUN chown -R nextjs:nodejs /app
USER nextjs

# 6. Exponemos el puerto 3000
EXPOSE 3000

# 7. Levantar Next.js en modo producción
# Utilizamos 'start' para correr la aplicación construida
CMD ["pnpm", "run", "start"]
