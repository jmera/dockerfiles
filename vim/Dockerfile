FROM alpine:3.11.5
MAINTAINER "Julich Mera <julich.mera@gmail.com>"
ENV TERM=xterm-256color

RUN apk add --update \
  bash \
  g++ \
  git \
  libxt-dev \
  make \
  ncurses-dev \
  ruby \
  ruby-dev \
  the_silver_searcher \
  xclip
RUN gem install rake

# Compile and install VIM from source
ARG VERSION=v8.2.0587
RUN git clone --depth 1 --branch $VERSION https://github.com/vim/vim.git /tmp/vim/
RUN cd /tmp/vim/src && \
  ./configure --enable-rubyinterp --with-x && \
  make && \
  make install
RUN rm -rf /tmp/vim/

RUN adduser dev -D --shell /bin/bash
RUN cd /home/dev/ && \
  git clone --depth 1 https://github.com/jmera/dotfiles.git .dotfiles/ && \
  cd .dotfiles/ && \
  ./install

RUN mkdir /home/dev/workdir/
RUN chown -R dev:dev /home/dev/

USER dev
WORKDIR /home/dev/workdir/

CMD ["/usr/local/bin/vim"]
