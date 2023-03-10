####################################################################################################
## Builder image
####################################################################################################
FROM python:3.11-alpine AS builder

RUN apk add --no-cache \
    ca-certificates \
    build-base \
    curl

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
RUN python -m pip install --upgrade pip setuptools
RUN curl -sSL https://install.python-poetry.org | python -
ENV PATH="${PATH}:/root/.local/bin"

RUN curl https://sh.rustup.rs -sSf | sh -s -- --profile minimal --default-toolchain stable -y
ENV PATH="${PATH}:/root/.cargo/bin"

WORKDIR /Rin

COPY /pyproject.toml /Rin/
COPY /poetry.lock /Rin/

# Create dependency wheels
RUN poetry export -f requirements.txt --output requirements.txt --without-hashes \
  && pip wheel --wheel-dir=/Rin/wheels -r requirements.txt

####################################################################################################
## Final image
####################################################################################################
FROM python:3.11-alpine

RUN apk add --no-cache \
    ca-certificates \
    tini \
    bash

WORKDIR /Rin

COPY /Bot/rinbot.py /Rin/Bot/
COPY /Bot/rincore.py /Rin/Bot/
COPY /Bot/Cogs/ /Rin/Bot/Cogs

# Copy dependency wheels from builder stage
COPY --from=builder /Rin/wheels /Rin/wheels
COPY --from=builder /Rin/requirements.txt /Rin/requirements.txt

COPY /Docker/Alpine-Docker/start.sh /Rin/start.sh

RUN pip install --upgrade pip

# Add an unprivileged user and set directory permissions
RUN adduser --disabled-password --gecos "" rin \
    && chown -R rin:rin /Rin \
    && chmod +x /Rin/start.sh

USER rin

# Install dependencies
RUN pip install --user --no-index --find-links=/Rin/wheels -r requirements.txt

ENTRYPOINT ["/sbin/tini", "--"]

CMD ["/Rin/start.sh"]

STOPSIGNAL SIGTERM

# Image metadata
LABEL org.opencontainers.image.title="Rin"
LABEL org.opencontainers.image.description="Discord bot focused on obtaining data from third-party services with lighting performance in mind"
LABEL org.opencontainers.image.url="https://rin-dbot.vercel.app"
LABEL org.opencontainers.image.licenses="Apache-2.0"
LABEL org.opencontainers.image.source="https://github.com/No767/Rin"
