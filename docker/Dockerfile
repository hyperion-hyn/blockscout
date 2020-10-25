# syntax = docker/dockerfile:experimental

ARG COIN

FROM bitwalker/alpine-elixir-phoenix:1.10.3

RUN apk --no-cache --update add alpine-sdk gmp-dev automake libtool inotify-tools autoconf python

EXPOSE 4000

ENV PORT=4000 \
    MIX_ENV="prod" \
    SECRET_KEY_BASE="RMgI4C1HSkxsEjdhtGMfwAHfyT6CKWXOgzCboJflfSm4jeAlic52io05KB6mqzc5"

WORKDIR /opt/app

# Cache elixir deps
ADD mix.exs mix.lock ./
ADD apps/block_scout_web/mix.exs ./apps/block_scout_web/
ADD apps/explorer/mix.exs ./apps/explorer/
ADD apps/ethereum_jsonrpc/mix.exs ./apps/ethereum_jsonrpc/
ADD apps/indexer/mix.exs ./apps/indexer/

RUN mix do deps.get, local.rebar --force, deps.compile

# Cache npm packages
ADD apps/block_scout_web/assets/package.json ./apps/block_scout_web/assets/
ADD apps/explorer/package.json ./apps/explorer/

RUN cd apps/block_scout_web/assets/ && \
    npm install && \
    cd - && \
    cd apps/explorer/ && \
    npm install && \
    cd -

ADD . .

RUN if [ "$COIN" != "" ]; then sed -i s/"POA"/"${COIN}"/g apps/block_scout_web/priv/gettext/en/LC_MESSAGES/default.po; fi

# Run forderground build and phoenix digest
RUN --mount=id=hex-cache,type=cache,sharing=locked,target=/opt/app/.cache/ \
    --mount=id=hex-cache,type=cache,sharing=locked,target=/opt/app/_build \
    mix compile

# Add blockscout npm deps
RUN --mount=id=npm-cache,type=cache,sharing=locked,target=/opt/app/.npm \
    cd apps/block_scout_web/assets/ && \
    npm run deploy && \
    cd -

RUN --mount=id=npm-cache,type=cache,sharing=locked,target=/opt/app/.npm \
    cd apps/explorer/ && \
    cd -

RUN apk update && apk del --force-broken-world alpine-sdk gmp-dev automake libtool inotify-tools autoconf python

# RUN mix do ecto.drop --force, ecto.create, ecto.migrate

RUN mix phx.digest

# USER default

# CMD ["mix", "phx.server"]
