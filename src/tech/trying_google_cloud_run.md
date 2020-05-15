# Trying Google Cloud Run

This is just a quick try-out of Google Cloud Run.

[There is some documentation](https://cloud.google.com/run/docs/quickstarts/build-and-deploy?_ga=2.252230089.-199990648.1584658988#containerizing) to help get started.

## AMAZING! TINY rust MUSL BUILD ARTIFACT! THANKS COMMUNITY!!!! 🦀

Yes, [the rust musl builder](https://github.com/emk/rust-musl-builder/blob/master/examples/using-diesel/Dockerfile) from emk & community is simply wonderful.  I cannot gush praise quickly enough.  Please just look into the project and make use of it, because **YOU CAN CREATE A TINY LITTLE RUST PROGRAM IN A TEENY TINY DOCKER CONTAINER** and that's a very good thing.

We compared the docker image size of a trivial Hello World built using  [the primary rust docker image](https://hub.docker.com/_/rust) to the docker image size of the same program built using the [rust musl builder](https://github.com/emk/rust-musl-builder).  Here are the results:

```sh
docker images |grep traditional
docker images |grep tiny-musl-guy
```
▶️ ▶️ ▶️

```text
traditional   ...(trimmed)...     1.6 GB
tiny-musl-guy ...(trimmed)...    9.22 MB
```

## Buddy Have An HTTP

You need to serve some sort of minimal HTTP request in order for cloud run to do anything with your program.  We arbitrarily picked the Hello World example from [tide](https://github.com/http-rs/tide).

```rust
#[async_std::main]
async fn main() -> Result<(), std::io::Error> {
    let mut app = tide::new();
    app.at("/").get(|_| async { Ok("Hello, world!") });
    app.listen("127.0.0.1:8080").await?;
    Ok(())
}
```

## Actually Have Google Do The Cloud Things Now

There's really nothing to show here.  [You can install and run the
gcloud command line utility YOURSELF](https://cloud.google.com/sdk/docs/quickstarts), because the documentation is
excellent.  Dead simple.

The remotely-created docker build will just work on the first try, because Google loves you. (Does that sound creepy? 🤔)

The build output is tiny.

**TODO**
**TODO**
**TODO**
**TODO**
![here is a pic of the tiny build, having been uploaded](/path/to/pic)