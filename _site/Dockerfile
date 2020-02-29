FROM ruby:2.4.0-alpine

RUN apk update && apk add nodejs build-base libxml2-dev libxslt-dev 
RUN mkdir /app
WORKDIR /app

COPY Gemfile ./Gemfile
COPY plainwhite.gemspec ./plainwhite.gemspec

RUN gem update --system
RUN gem update
RUN gem uninstall bundler
RUN gem install bundler
RUN bundle