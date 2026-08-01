# =============================================================================
# Stage 1: Dependencies — instala dependências
# =============================================================================

FROM node:20-alpine AS deps

RUN apk add --no-cache libc6-compat

WORKDIR /app

# Copia arquivos de dependência
COPY package.json package-lock.json ./

# Instala dependências usando lockfile
RUN npm ci


# =============================================================================
# Stage 2: Builder — compila aplicação Next.js
# =============================================================================

FROM node:20-alpine AS builder

WORKDIR /app

# Copia node_modules do estágio anterior
COPY --from=deps /app/node_modules ./node_modules

# Copia código fonte
COPY . .

# Desabilita telemetria Next.js
ENV NEXT_TELEMETRY_DISABLED=1

# Gera build de produção
RUN npm run build


# =============================================================================
# Stage 3: Runner — imagem final de produção
# =============================================================================

FROM node:20-alpine AS runner

WORKDIR /app


# Variáveis de ambiente
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=3000
ENV HOSTNAME=0.0.0.0


# Instala curl para o HEALTHCHECK
RUN apk add --no-cache curl


# =============================================================================
# Usuário não-root
# =============================================================================

RUN addgroup --system --gid 1001 nodejs

RUN adduser --system \
    --uid 1001 \
    nextjs


# =============================================================================
# Copia arquivos gerados pelo Next.js standalone
# =============================================================================

COPY --from=builder --chown=nextjs:nodejs \
    /app/.next/standalone \
    ./

COPY --from=builder --chown=nextjs:nodejs \
    /app/.next/static \
    ./.next/static


# =============================================================================
# Executa como usuário não-root
# =============================================================================

USER nextjs


# =============================================================================
# Configuração do serviço
# =============================================================================

EXPOSE 3000


HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000 || exit 1


# =============================================================================
# Inicialização Next.js
# =============================================================================

CMD ["node", "server.js"]