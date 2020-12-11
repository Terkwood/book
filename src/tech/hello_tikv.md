# Hello TiKV

Dec 11, 2020. I am tinkering with TiKV and trying to use the rust client.

After spending some time in the rust client README but not having much success, I found [this document](https://tikv.org/docs/4.0/tasks/try/docker-stack/) explaining how to operate TiKV alone, using docker stack.

I published a [working example of using TiKV rust client with TiKV in docker stack on Github](https://github.com/Terkwood/hello-tikv-rust).

## Things that didn't work

I had to make a few changes to the rust client dockerfile.

### non-interactive install for tzdata

I used `DEBIAN_FRONTEND="noninteractive"`, so that `tzdata` would not prompt during installation. I also had to use `apt-get` instead of `apt`!

```text
RUN DEBIAN_FRONTEND="noninteractive" apt-get install --yes build-essential protobuf-compiler curl cmake golang
```

### rust stable has async await

The doc website incorrectly requires a nightly rust toolchain. This is no longer necessary since `async`/`await` functionality has landed in rust stable. I removed all references to the version-hacking file `rust-toolchain`.

```text
# ...snip...
# Install Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="/builder/.cargo/bin:${PATH}"

# Fetch, then prebuild all deps
COPY Cargo.toml /builder/build/
```

### Missing libssl-dev

I also had to install [libssl-dev](https://github.com/sfackler/rust-openssl/issues/763#issuecomment-339269157).

## Report these issues

The above issues were [reported to the TiKV docs website](https://github.com/tikv/website/issues/214).
