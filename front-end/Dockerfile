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
# Configuramos pnpm para usar menos memoria y espacio
RUN pnpm config set store-dir /app/.pnpm-store && \
    pnpm install --frozen-lockfile --prod=false

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

# 2. Copiamos los archivos necesarios para pnpm
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/pnpm-lock.yaml ./pnpm-lock.yaml

# 3. Instalamos solo las dependencias de producción con pnpm
# Esto es más eficiente en memoria y espacio
RUN pnpm config set store-dir /app/.pnpm-store && \
    pnpm install --frozen-lockfile --prod

# 4. Copiamos el build y archivos públicos
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

# 5. Crear usuario no-root para seguridad
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# 6. Cambiar ownership de los archivos para el usuario no-root
RUN chown -R nextjs:nodejs /app
USER nextjs

# 7. Exponemos el puerto 3000
EXPOSE 3000

# 8. Levantar Next.js en modo producción
# Utilizamos 'start' para correr la aplicación construida
CMD ["pnpm", "run", "start"]
