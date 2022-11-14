FROM rust:1.65.0 as build_wasm

WORKDIR /out

COPY package.json pnpm-workspace.yaml ./
COPY ./wasm/ ./wasm/

RUN cargo install -f wasm-bindgen-cli && rustup target add wasm32-unknown-unknown
RUN curl -fsSL https://get.pnpm.io/install.sh | bash - && . ~/.bashrc && pnpm run build:wasm

FROM node:18.12 as build_web

WORKDIR /out

# Install pnpm
RUN corepack enable && corepack prepare pnpm@7.16.0 --activate

# Copy wasm files
COPY ./wasm/ ./wasm/
COPY --from=build_wasm /out/wasm/simple/dist/ ./wasm/simple/dist/
COPY --from=build_wasm /out/wasm/game-of-life/dist/ ./wasm/game-of-life/dist/

# Install dependencies
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml .npmrc ./
RUN pnpm install --frozen-lockfile

# Copy the rest of the files
COPY . ./

# Build the project
RUN pnpm run build

# Final image
FROM jensdll/image-data-hiding:nginx-base.1.23.2-alpine

COPY --from=build_web --chown=nginx:nginx /out/dist/ /data/www/

EXPOSE 80
