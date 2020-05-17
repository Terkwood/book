# Trying Google Cloud Run

This is just a quick try-out of Google Cloud Run.

We will build minimal containers that serve Hello World text over HTTP using both Rust 🦀 and Deno 🦕.

Google provides [very nice documentation](https://cloud.google.com/run/docs/quickstarts/build-and-deploy?_ga=2.252230089.-199990648.1584658988#containerizing) to help get started.

## Using Rust on Google Cloud Run

We expect to be able to run Hello World in rust using a tiny docker build image.  [The rust musl builder](https://github.com/emk/rust-musl-builder/blob/master/examples/using-diesel/Dockerfile) from emk & community is simply wonderful in this regard. You can create a tiny little rust program in a tiny little docker container.  When you're counting your cloud provider's storage quota in terms pennies and cents, that's a very good thing.

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

```ignore
#[async_std::main]
async fn main() -> Result<(), std::io::Error> {
    let mut app = tide::new();
    app.at("/").get(|_| async { Ok("Hello, world!") });
    app.listen("127.0.0.1:8080").await?;
    Ok(())
}
```

## Actually Have Google Do The Cloud Things Now

This part is refreshingly straightforward.  First, [we installed the gcloud command line utility](https://cloud.google.com/sdk/docs/quickstarts).  Dead simple.

### Building a Docker Image in GCR

The remotely-created docker build will just work on the first try, because Google loves you. (Does that sound creepy? 🤔)

We built it remotely using `gcloud`, after which it is automatically stored in the Google container registry:

```sh
gcloud builds submit --tag gcr.io/PROJECT_NAME/rust-gcr-demo
```

We received a very comforting stream of text indicating forward progress while engaging with google's cloud in this way:

```text
Creating temporary tarball archive of 8 file(s) totalling 2.0 KiB before compression.
Uploading tarball of [.] to [gs://PROJECT_NAME_cloudbuild/source/9999999999.34-aaaaaad7496d917b02d0cdd1b30a.tgz]
Created \[https://cloudbuild.googleapis.com/v1/projects/PROJECT_NAME/builds/e0e0e0e0-aaaa-ffff-928b-9a9a9a9a9a9a\].
Logs are available at \[https://console.cloud.google.com/cloud-build/builds/e0e0e0e0-aaaa-ffff-928b-9a9a9a9a9a9a?project=000000000000\].
--------------------------------- REMOTE BUILD OUTPUT ---------------------------------
starting build "e0e0e0e0-aaaa-ffff-928b-9a9a9a9a9a9a"

FETCHSOURCE
Fetching storage object: gs://PROJECT_NAME_cloudbuild/source/9999999999.34-9999999999999999999999.tgz#8888888888888888
Copying gs://PROJECT_NAME_cloudbuild/source/999999999934-9999999999999999999999.tgz#8888888888888888...
/ [1 files][  1.5 KiB/  1.5 KiB]
Operation completed over 1 objects/1.5 KiB.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  9.728kB
Step 1/6 : FROM hayd/alpine-deno
f5a1a52ea0cc: Pull complete
91f70b368d00: Pull complete
Digest: sha256:68c60a649aadbb7139bd4fd248eb7426d3f829f4574ce78ebc97fc3e22059498
Status: Downloaded newer image for hayd/alpine-deno:latest
 ---> 4b8d66a41759
Step 2/6 : WORKDIR /var/hello
 ---> Running in 78f3fc613ff3

...SNIP...

Step 6/6 : CMD ["run","--allow-env","--allow-net","index.ts"]
 ---> Running in 45e58559d684
Removing intermediate container 45e58559d684
 ---> 3ecf00d3c661
Successfully built 3ecf00d3c661
Successfully tagged gcr.io/PROJECT_NAME/rust-gcr-demo:latest
PUSH
Pushing gcr.io/PROJECT_NAME/rust-gcr-demo
The push refers to repository [gcr.io/PROJECT_NAME/rust-gcr-demo]
5ca1b5bf7698: Preparing

...SNIP...

f8d7c190aaa1: Pushed
3b7d1dad260a: Pushed
5216338b40a7: Layer already exists
517a217ae4f9: Pushed
70c4b84bb2a4: Pushed
latest: digest: sha256:e1563eb2fdcba0a36b23c34050a2145b47833ce607adc912ee750348bc512e9$
 size: 1573
DONE
--------------------------------------------------------------------------------------$

ID                                    CREATE_TIME                DURATION  SOURCE
                                                                          IMAGES
                         STATUS
e0e0e0e0-aaaa-ffff-928b-9e9a9a9a9a9a  2020-05-15T23:18:50+00:00  20S       gs://PROJECT_NAME_cloudbuild/source/6666666666.34-aaaaaaaaaaaaaaaaaaaaaa.tgz  gcr.io/PROJECT_NAME/rust-gcr-demo (+1 more)  SUCCESS
```

The build output is tiny.  When we finally added `tide` to the hello world example  pictured below, the artifact was closer to 4.4MB... but you get the idea.

![here is a pic of a tiny build, having been uploaded](https://user-images.githubusercontent.com/38859656/82101143-acd60c00-96d9-11ea-9009-06ffaa96d6c1.png)


### Deploying to Google Cloud Run

Deployment is also painless.  Make sure you bind to the host `0.0.0.0` in your application, and obey the `PORT` variable expected by GCR.  We are lazy and just hardcoded port `8080` in our code, ignoring best practices for the sake of a quick, throwaway learning experience.

`gcloud` CLI deployment remains simple:

```sh
gcloud run deploy --image gcr.io/PROJECT_NAME/rust-gcr-demo --platform managed
```

And there is another pleasing interaction waiting for you in your terminal:

```text
Service name (rust-gcr-demo):

Allow unauthenticated invocations to [rust-gcr-demo] (y/N)?

Deploying container to Cloud Run service [rust-gcr-demo] in project [PROJECT_NAME] region [us-east1]
✓ Deploying new service... Done.
  ✓ Creating Revision...
  ✓ Routing traffic...
Done.
Service [rust-gcr-demo] revision [rust-gcr-demo-00001-qow] has been deployed and is serving 100 percent of traffic at https://rust-gcr-demo-somenonsense-xx.z.run.app
```

## Comparison to a Deno Hello World

We did the same thing over again
