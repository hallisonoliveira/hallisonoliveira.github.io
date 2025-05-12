FROM jekyll/jekyll:4.2.0

WORKDIR /srv/jekyll

COPY . /srv/jekyll

RUN bundle install

EXPOSE 4000

CMD ["jekyll", "serve", "--watch", "--force_polling", "--host", "0.0.0.0"]
