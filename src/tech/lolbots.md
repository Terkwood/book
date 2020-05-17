# lolbots 🙄 🤣 🤖 💩

Why not just make a bot which responds ironically to our discord server?  It should train itself based on our local subgroup of freaks and weirdos.  _Obviously_, we would make the source open for the public to scrutinize, because if we don't keep tinkering frenetically with two-star projects on Github, _then we won't achieve immortality_.

We could try using the [DeepMoji](https://github.com/Terkwood/DeepMoji) project to somehow mix sarcasm into the intellectual brew, but we still need to investigate, and make sure that this isn't going to eat up weeks of time.  I mean, we sort of coasted through that data mining course in grad school without considering the 72 Trillion Dollar AI Hype Machine that would barrel into the world a few short years after we graduated.  Dear Reader, we're not machine learning experts over here.  Just a curious application developer.

![but really](https://user-images.githubusercontent.com/38859656/82142447-62a07800-980a-11ea-8a6f-94ff5c843602.jpg)

![so much love](https://user-images.githubusercontent.com/38859656/82142476-a2fff600-980a-11ea-962e-3c461ca975ae.jpg)


[You wanna try?](https://deepmoji.mit.edu/)

The test cases are what led us this far down the rabbit hole.  Look at this test case.  [JUST LOOK AT IT!](https://github.com/bfelbo/DeepMoji/blob/33a6640bff1aa700eb376a9932c5f35cca986807/tests/test_finetuning.py#L166) 💖 🐇

![deepmoji test cases](https://user-images.githubusercontent.com/38859656/82142168-6e8b3a80-9808-11ea-8f6d-a6b0ea15e743.jpg)

OK, you get the idea.  Go ahead and read more about [the DeepMoji project from MIT](https://www.media.mit.edu/projects/deepmoji/overview/) if you want to.

## So What?

We already have an NVIDIA Jetson Nano sitting on our desk computing the moves for our [Goban](https://github.com/Terkwood/BUGOUT) using [KataGo - a slick, community-sponsored neural net](https://github.com/lightvector/KataGo). Please don't judge us by the level of dust collecting on this lil' guy -- we're working in a _very dynamic environment_. 🤧

![lilbrain in situ](https://user-images.githubusercontent.com/38859656/82141784-2834dc00-9806-11ea-8591-0c9cbdb53074.jpeg)


We even created 1/3 of the [necessary architecture](https://github.com/Terkwood/BUGOUT/tree/unstable/botlink) in our recent project: we could just string up a websocket from the local NVIDIA GPU, to a cloud provider, and avoid exposing our blessed domicile's network surface to the world of trolls, bots, and whatever else.  Rather than creating some sort of PULL-based situation, we would prefer to have some cloud-hosted Discord gateway push data down to the NVIDIA Nano GPU, which would then consider our chats in a Machine Learny Way, and then respond with a snarky reaction.

🙄 🤖 We're gunning for Artificial Irony. 🤖 🙄

## Install tensorflow on the NVIDIA Jetson Nano

Our Nano is still pretty lightly loaded.  We tried to install several packages, hoping to follow something like the [NVIDIA instructions](https://docs.nvidia.com/deeplearning/frameworks/install-tf-jetson-platform/index.html) to easily run Tensorflow.  Yeah.  Right.

### The Woe

After about 38 minutes we've given up on the project entirely, realizing that the model we want to use only supports python 2, while the NVIDIA jetson nano (surprise!) wants us to install TF using python 3.

🔥 🚒  🔥

```text
Successfully installed absl-py-0.9.0 astor-0.8.1 cachetools-4.1.0 google-auth-1.14.3 google-auth-oauthlib-0.4.1 google-pasta-0.2.0 grpcio-1.29.0 keras-preprocessing-1.1.2 markdown-3.2.2 oauthlib-3.1.0 opt-einsum-3.2.1 pyasn1-0.4.8 pyasn1-modules-0.2.8 requests-oauthlib-1.3.0 rsa-4.0 scipy-1.4.1 tensorboard-2.1.1 tensorflow-2.1.0+nv20.4 tensorflow-estimator-2.1.0 termcolor-1.1.0 werkzeug-1.0.1 wrapt-1.12.1


$ python3
Python 3.6.9 (default, Apr 18 2020, 01:56:04)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow
2020-05-17 00:03:42.129454: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcudart.so.10.2'; dlerror: libcudart.so.10.2: cannot open shared object file: No such file or directory
2020-05-17 00:03:42.129529: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
2020-05-17 00:03:45.107357: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libnvinfer.so.7'; dlerror: libnvinfer.so.7: cannot open shared object file: No such file or directory
2020-05-17 00:03:45.107613: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libnvinfer_plugin.so.7'; dlerror: libnvinfer_plugin.so.7: cannot open shared object file: No such file or directory
2020-05-17 00:03:45.107673: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:30] Cannot dlopen some TensorRT libraries. If you would like to use Nvidia GPU with TensorRT, please make sure the missing libraries mentioned above are installed properly.
>>>
```


BUT WAIT THEY'RE STILL WORKING ON IT! [Here is a forked repo with active contribution](https://github.com/nklapste/DeepMoji/tree/feature/SimplePython3Migration), working toward supporting python 3 and TF 2.0.  Note that it's TF 2.0.  Not TF 2.1.

Here is the [related PR](https://github.com/bfelbo/DeepMoji/pull/53) on the main DeepMoji repo.

### Not So Good

Sort of fail. [Try TF 2.0.0](https://docs.nvidia.com/deeplearning/frameworks/install-tf-jetson-platform/index.html#install_multiple_versions_tensorflow).


[Others are having this same issue](https://github.com/tensorflow/tensorflow/issues/34759#issuecomment-629179592)


THIS DOES NOT REALLY WORK, VERSIONS INCOMPATIBLE: We probably need to [line up versions just so](
https://docs.nvidia.com/deeplearning/frameworks/install-tf-jetson-platform-release-notes/tf-jetson-rel.html#tf-jetson-rel)...

THIS DOES NOT REALLY WORK, CRASHES: Or we could potentially [run in a container](https://docs.nvidia.com/deeplearning/frameworks/tensorflow-release-notes/running.html#running)

Looks like one sure-fire fix is to [build from source](https://github.com/tensorflow/tensorflow/issues/34759#issuecomment-570102033).  In which case we would need golang so that we can install bazel

And yes, we must build bazel from source, because we're on an ARM64 platform. 🤓

https://github.com/bazelbuild/bazel/issues/8833#issuecomment-629039759

### FINALLY: PLEASING SCROLL

We really needed some of this scroll.  After a bit of a struggle, we're compiling Tensorflow 2.1.0 on our NVIDIA Jetson Nano and may yet be able to Hello.

```text
Configuration finished
userhost:~/git/tensorflow$ bazel build //tensorflow/tools/pip_package:build_pip_package
Extracting Bazel installation...
Starting local Bazel server and connecting to it...
```

![yo theres even color](https://user-images.githubusercontent.com/38859656/82141600-f707dc00-9804-11ea-9fc5-139b7aa374b1.png)

### Come Back 65,536 Hours Later...

...and the Nano is still compiling Tensorflow 2.1.  That's right, 2.1.  We completely forgot that there's likely no support by DeepMoji for this version - we should have compiled 2.0.  

That's fine.  It doesn't matter.  Nothing matters. (OK, PLENTY OF THINGS MATTER, calm down.)  But yeah, we're sticking with the latest version of Tensorflow and we're just going to see how things play out.

Here's what's up, we've already downloaded half the internet during this compile.

And finally IT FAILS!

## You Need a Rest Break

💤
💤💤
```text
pdate(highwayhash::HH_U64, const char*, highwayhash::HH_U64, State*)':
external/highwayhash/highwayhash/compiler_specific.h:52:46: warning: requested alignment 32 is larger than 16 [-Wattributes]
 #define HH_ALIGNAS(multiple) alignas(multiple)  // C++11
                                              ^
external/highwayhash/highwayhash/state_helpers.h:49:41: note: in expansion of macro 'HH_ALIGNAS'
   char final_packet[State::kPacketSize] HH_ALIGNAS(32) = {0};
                                         ^~~~~~~~~~
ERROR: /home/user/.cache/bazel/_bazel_user/9fef75a91d3167b0d0af1e5d464b12c3/external/aws-c-common/BUILD.bazel:12:1: C++ compilation of rule '@aws-c-common//:aws-c-common' failed (Exit 1)
external/aws-c-common/source/arch/cpuid.c:23:10: fatal error: immintrin.h: No such file or directory
 #include <immintrin.h>
          ^~~~~~~~~~~~~
compilation terminated.
Target //tensorflow/tools/pip_package:build_pip_package failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 2988.839s, Critical Path: 67.64s
INFO: 1146 processes: 1146 local.
FAILED: Build did NOT complete successfully
```
💤
💤💤


## Cheer Up

Look, this isn't our first rodeo.

[Let's just try compiling TF for the next 40 hours using this guide](https://jkjung-avt.github.io/build-tensorflow-2.0.0/).  This individual has even figured out TF 2.0!

We don't have to sit and watch every step of the process.

We don't have to even shut down KataGo.  That would be weird.

If it works, it works.  If it doesn't, _FINE!_
