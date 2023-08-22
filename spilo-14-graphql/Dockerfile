FROM registry.opensource.zalan.do/acid/spilo-14:2.1-p3 as builder

RUN apt-get update -qq
RUN apt-get install -yqq pkg-config libssl-dev libclang-dev postgresql-server-dev-14 zlib1g-dev libreadline6-dev curl build-essential 

# install rust
RUN useradd -ms /bin/bash graphql
USER graphql
RUN curl https://sh.rustup.rs -sSf | bash -s -- -y
ENV PATH="/home/graphql/.cargo/bin:${PATH}"

# install pgrx
RUN cargo install cargo-pgrx --version 0.9.8 --locked
RUN cargo pgrx init --pg14=/usr/lib/postgresql/14/bin/pg_config

WORKDIR /home/graphql
RUN curl -L https://github.com/supabase/pg_graphql/archive/refs/tags/v1.3.0.tar.gz -o pg_graphql.tar.gz && \
  tar xzf pg_graphql.tar.gz && \
  cd pg_graphql-1.3.0

WORKDIR /home/graphql/pg_graphql-1.3.0
RUN cargo pgrx package

USER root
RUN cp /home/graphql/pg_graphql-1.3.0/target/release/pg_graphql-pg14/usr/share/postgresql/14/extension/pg_graphql.control /usr/share/postgresql/14/extension/pg_graphql.control && \
    cp /home/graphql/pg_graphql-1.3.0/target/release/pg_graphql-pg14/usr/share/postgresql/14/extension/pg_graphql--1.3.0.sql /usr/share/postgresql/14/extension/pg_graphql--1.3.0.sql && \
    cp /home/graphql/pg_graphql-1.3.0/target/release/pg_graphql-pg14/usr/lib/postgresql/14/lib/pg_graphql.so /usr/lib/postgresql/14/lib/pg_graphql.so


# target
FROM registry.opensource.zalan.do/acid/spilo-14:2.1-p3

COPY --from=builder /home/graphql/pg_graphql-1.3.0/target/release/pg_graphql-pg14/usr/share/postgresql/14/extension/pg_graphql.control /usr/share/postgresql/14/extension/pg_graphql.control
COPY --from=builder /home/graphql/pg_graphql-1.3.0/target/release/pg_graphql-pg14/usr/share/postgresql/14/extension/pg_graphql--1.3.0.sql /usr/share/postgresql/14/extension/pg_graphql--1.3.0.sql
COPY --from=builder /home/graphql/pg_graphql-1.3.0/target/release/pg_graphql-pg14/usr/lib/postgresql/14/lib/pg_graphql.so /usr/lib/postgresql/14/lib/pg_graphql.so
