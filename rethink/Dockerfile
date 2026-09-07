# BUILD_FROM must be declared before the first FROM — args declared between stages
# belong to the preceding stage and are invisible to FROM instructions.
ARG BUILD_FROM

# Build stage: compile rethink from the pinned fork revision
FROM alpine:3.21 AS build

RUN apk add --no-cache nodejs npm git

# forevaclevah2/rethink: anszom/rethink master + the 2REF11EBIR__4 handler
# (LG LF21G6200S, 65-byte truncated status body). See docs/2REF11EBIR__4.md in
# that repo for the frame capture and offset table. Bump on every add-on update.
#
# The revision below lives on branch `add-2REF11EBIR__4`, not on master. The
# clone here is deliberately NOT shallow, so all refs are fetched and the commit
# is reachable for checkout.
ARG RETHINK_REPO=https://github.com/forevaclevah2/rethink.git
ARG RETHINK_REV=c6ae9c02f8b32e6fd2b7fd30b0108047e094c9ec

RUN git clone "$RETHINK_REPO" /src \
    && git -C /src checkout "$RETHINK_REV"

WORKDIR /src
RUN npm ci --ignore-scripts \
    && npm run build \
    && npm prune --omit=dev

# Runtime stage: Home Assistant base image (s6 entrypoint disabled via init: false)
FROM $BUILD_FROM

RUN apk add --no-cache nodejs openssl jq

WORKDIR /app
COPY --from=build /src/package.json /src/package-lock.json ./
COPY --from=build /src/node_modules ./node_modules
COPY --from=build /src/dist ./dist
COPY run.sh /app/run.sh
RUN chmod +x /app/run.sh

ENTRYPOINT []
CMD ["/app/run.sh"]
