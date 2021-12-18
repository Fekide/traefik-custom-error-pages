# jekyll
FROM jekyll/builder:latest as jekyll
WORKDIR /tmp

COPY Gemfile /tmp/Gemfile
COPY Gemfile.lock /tmp/Gemfile.lock

RUN gem install bundler:2.2.33 &&\ 
  bundle install

ENV JEKYLL_ENV=production

COPY _config.yml /tmp/_config.yml
COPY src /tmp/src

RUN chown -R jekyll:jekyll /tmp &&\
  jekyll build
# nginx
FROM nginx:alpine as nginx

COPY .cloud/nginx/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=jekyll /tmp/_site/ /var/www/public/
