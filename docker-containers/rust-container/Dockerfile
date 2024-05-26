FROM rust:latest

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get clean \
    && apt-get install -y git

RUN git clone https://github.com/ThomasMiz/reddy.git && mkdir -p /reddy/executable
WORKDIR /reddy
RUN cargo build && mv /reddy/target/debug/reddy /reddy/executable/