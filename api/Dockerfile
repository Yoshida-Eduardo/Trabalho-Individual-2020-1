FROM ruby:2.5.7-alpine as builder

RUN apk add --update --no-cache \
  binutils-gold \
  build-base \
  curl \
  file \
  g++ \
  gcc \
  git \
  less \
  libstdc++ \
  libffi-dev \
  libc-dev \
  linux-headers \
  libxml2-dev \
  libxslt-dev \
  libgcrypt-dev \
  make \
  netcat-openbsd \
  openssl \
  pkgconfig \
  postgresql-dev \
  python \
  tzdata

COPY Gemfile Gemfile.lock ./
RUN gem install bundler -v 2.1.4 &&\
    bundle install

FROM ruby:2.5.7-alpine

RUN apk add --update --no-cache \
  tzdata \
  postgresql-client \
  nodejs \
  bash \
  xz-libs

COPY --from=builder /usr/local/bundle/ /usr/local/bundle/

RUN mkdir /myapp
WORKDIR /myapp

COPY . .

# Removes rails server pid just in case
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Start the main process.
CMD ["rails", "server", "-b", "0.0.0.0"]