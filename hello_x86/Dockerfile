FROM alpine:latest

WORKDIR /slides

COPY . .

RUN apk update
RUN apk add strace binutils figlet slides nasm perl-app-cpanminus musl-dev gcc bash go file shadow nano vim man-db man-pages
RUN cpanm Graph::Easy --force

RUN echo "export PATH=\"/root/go/bin:\$PATH\"" >> $HOME/.bashrc
RUN chsh -s bash

RUN go install github.com/bnagy/cgasm@latest

RUN chmod +x hello_x86.md

CMD ["slides", "hello_x86.md"]
