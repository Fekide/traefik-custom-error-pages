# jekyll
FROM ruby:3.3 AS jekyll
WORKDIR /tmp

COPY Gemfile /tmp/Gemfile
COPY Gemfile.lock /tmp/Gemfile.lock

RUN bundle install

ENV JEKYLL_ENV=production

COPY _config.yml /tmp/_config.yml
COPY src /tmp/src

RUN bundle exec jekyll build
# nginx
FROM nginx:alpine AS nginx

COPY .cloud/nginx/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=jekyll /tmp/_site/ /var/www/public/
