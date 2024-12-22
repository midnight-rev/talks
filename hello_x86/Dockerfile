FROM alpine:latest

WORKDIR /slides

COPY . .

RUN apk update
RUN apk add git py3-pip python3 strace binutils figlet slides nasm perl-app-cpanminus musl-dev gcc bash go file shadow nano vim man-db man-pages

# Syscall Searcher
RUN git clone https://github.com/midnight-rev/syscall_searcher
RUN python3 -m pip install requests --break-system-packages
RUN chmod a+x syscall_searcher/main.py
RUN mv syscall_searcher/main.py syscall_searcher/syscall_searcher.py
RUN mv syscall_searcher /root

RUN cpanm Graph::Easy --force

RUN echo "export PATH=\"/root/go/bin:/root/syscall_searcher:\$PATH\"" >> $HOME/.bashrc
RUN chsh -s bash

RUN go install github.com/bnagy/cgasm@latest

RUN chmod +x hello_x86.md

CMD ["slides", "hello_x86.md"]
