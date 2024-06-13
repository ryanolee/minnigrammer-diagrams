# use latest python 3 alpine image.
FROM python:3-alpine as base

# install system dependencies.
RUN apk update && apk add --no-cache \
  gcc libc-dev g++ graphviz git bash go imagemagick inkscape ttf-opensans curl fontconfig xdg-utils

# install go package.
RUN GOBIN=/usr/local/bin/ go install github.com/mingrammer/round@latest

# install fonts
RUN curl -O https://noto-website.storage.googleapis.com/pkgs/NotoSansCJKjp-hinted.zip \
  && mkdir -p /usr/share/fonts/NotoSansCJKjp \
  && unzip NotoSansCJKjp-hinted.zip -d /usr/share/fonts/NotoSansCJKjp/ \
  && rm NotoSansCJKjp-hinted.zip \
  && fc-cache -fv

from base as development

# add go bin to path.
ENV PATH "$PATH:/root/go/bin"

# project directory.
WORKDIR /usr/src/diagrams

# Copy the rest of your app's source code from your host to your image filesystem.
COPY . .

# install python requirements.
RUN pip install black graphviz jinja2

# Prod builder: Used for building "diagrams" for use in the prod image.
FROM base as prod-builder

WORKDIR /usr/src/diagrams

# Install poetry
RUN python -m pip install poetry

# Compile the application.
COPY poetry.lock pyproject.toml ./
RUN poetry install --only main --no-interaction
COPY . .
RUN poetry build -f wheel

FROM python:3-alpine as prod

# Create diagrams directory.
WORKDIR /diagrams

# install core system dependencies.
RUN apk add --no-cache \
    graphviz bash imagemagick inkscape ttf-opensans fontconfig xdg-utils

# Copy already pulled assets from the development image.

COPY --from=development /usr/local/bin/round /usr/local/bin/round
COPY --from=development /usr/share/fonts/NotoSansCJKjp /usr/share/fonts/NotoSansCJKjp

# Copy the compiled application.
COPY --from=prod-builder /usr/src/diagrams/dist/*.whl /tmp/


# Install the application.
RUN  apk add --no-cache g++ \
     && pip install --no-cache-dir /tmp/*.whl \
     && rm -rf /tmp/*.whl \
     && mkdir -p /diagrams \
     && fc-cache -fv \
     && apk del g++

CMD ["python", "-m", "diagrams"]
