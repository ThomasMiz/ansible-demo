FROM rust:latest

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get clean \
    && apt-get install -y git

RUN git clone https://github.com/ThomasMiz/reddy.git && mkdir -p /reddy/executable
WORKDIR /reddy
RUN cargo build --release && mv /reddy/target/release/reddy /reddy/executable/