FROM bluedoc/base:2.7.2-alpine

ENV RAILS_ENV "production"
ENV DATABASE_URL "postgres://postgres@localhost:5432/bluedoc"
ENV REDIS_URL "redis://localhost:6379/1"
ENV APP_HOST "http://localhost"

EXPOSE 443 80

WORKDIR /home/app/bluedoc

RUN mkdir -p /home/app && \
  find / -type f -iname '*.apk-new' -delete && \
  rm -rf '/var/cache/apk/*' '/tmp/*'

COPY Gemfile Gemfile.lock package.json yarn.lock /home/app/bluedoc/

COPY bin/docker-entrypoint /home/app/bluedoc/bin/docker-entrypoint
RUN chmod +x /home/app/bluedoc/bin/docker-entrypoint


# `apk` を使って nodejs, yarn をインストール
RUN apk add --no-cache nodejs npm yarn

# ミラー設定を削除
RUN bundle config unset mirror.https://rubygems.org

RUN bundle config set --local path 'vendor/bundle'

# bundle install と yarn install のエラー回避
RUN bundle install --jobs 20 --retry 5 && \
    yarn install --frozen-lockfile && \
    if [ -d "/home/app/bluedoc/vendor/bundle" ]; then \
      find /home/app/bluedoc/vendor/bundle -name tmp -type d -exec rm -rf {} + && \
      rm -Rf /home/app/bluedoc/vendor/bundle/ruby/*/cache; \
    fi
ADD . /home/app/bluedoc

ENV RUBYOPT "--jit"
ARG COMMIT_VERSION
ENV BLUEDOC_BUILD_VERSION=${COMMIT_VERSION}

VOLUME /home/app/bluedoc/storage \
  /home/app/bluedoc/log \
  /home/app/bluedoc/tmp \
  /home/app/bluedoc/data \
  /tmp \
  /var/lib/postgresql \
  /var/lib/redis \
  /usr/share/elasticsearch/data

RUN SECRET_KEY_BASE=$(rails secret) bundle exec rails assets:precompile RAILS_ENV=production

RUN rm -Rf /home/app/bluedoc/.git && \
  rm -Rf /home/app/bluedoc/app/javascript && \
  rm -Rf /home/app/bluedoc/docs && \
  rm -Rf /home/app/bluedoc/node_modules && \
  rm -Rf /home/app/bluedoc/package.json && \
  rm -Rf /home/app/bluedoc/yarn.lock && \
  rm -Rf /home/app/bluedoc/.babelrc && \
  rm -Rf /home/app/bluedoc/.circleci && \
  rm -Rf /home/app/bluedoc/.rubocop.yml && \
  rm -Rf /home/app/bluedoc/.dockerignore && \
  rm -Rf /home/app/bluedoc/.ruby-version && \
  rm -Rf /home/app/bluedoc/.byebug_history && \
  rm -Rf /home/app/bluedoc/test && \
  rm -Rf /usr/local/share/.cache && \
  rm -Rf /home/app/bluedoc/vendor/cache

# Add config
COPY ./config/elasticsearch /usr/share/elasticsearch/config
ADD ./config/etc/redis.conf /etc/redis.conf
ADD ./config/nginx/ /etc/nginx
ADD ./config/etc/Caddyfile /etc/Caddyfile
COPY bin/docker-entrypoint /home/app/bluedoc/bin/docker-entrypoint
RUN chmod +x /home/app/bluedoc/bin/docker-entrypoint

RUN apk del .builddeps

ENTRYPOINT ["/home/app/bluedoc/bin/docker-entrypoint"]
CMD ["/home/app/bluedoc/bin/docker-start"]
