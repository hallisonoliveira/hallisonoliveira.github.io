FROM ruby:3.1-slim

ENV LANG=C.UTF-8 \
    GEM_HOME=/usr/local/bundle \
    PATH=/usr/local/bundle/bin:$PATH

# Dependências do sistema
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends \
      build-essential \
      libffi-dev \
      libyaml-dev \
      libgdbm-dev \
      zlib1g-dev \
      curl \
      git \
      nodejs \
      && apt-get clean && rm -rf /var/lib/apt/lists/*

# Instala bundler
RUN gem install bundler

# Diretório da aplicação
WORKDIR /srv/jekyll

# Copia os arquivos de dependência e instala gemsd
COPY Gemfile ./
RUN bundle install

# Copia o restante do projeto
COPY . .

EXPOSE 4000

# Usa bundle exec jekyll
CMD ["bundle", "exec", "jekyll", "serve", "--watch", "--force_polling", "--host", "0.0.0.0"]
# CMD ["bundle", "install"]
