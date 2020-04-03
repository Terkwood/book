# Colab Notebooks

Colab can be useful.  Sometimes.

## Deepdream

[This is the Deepdream Colab Notebook](https://research.google.com/seedbank/seed/deepdream), it its original location on Seedbank.

[The newer version](https://aihub.cloud.google.com/p/products%2Ff9e8fc11-ad0f-410a-bebe-2482066ce570) is hosted on AIHub.

It can transform strange things into even stranger artifacts.

It's based on TF 1.x, so you need to make sure you use the old version of the lib before starting your notebook.

```
%tensorflow_version 1.x
```

### Experimenting

You can then do some transformations.


#### Waking Image

![This input](https://user-images.githubusercontent.com/38859656/78388122-78d1dc00-75ae-11ea-819b-626d608970ee.jpg)


#### Dreamy Image

This output had all the default settings for step #4, but we boosted strength to ~500.

![An output example](https://user-images.githubusercontent.com/38859656/78389969-bf750580-75b1-11ea-8ef4-9b295657588a.jpeg)

#### Dreamier Image

This output had further-jiggled settings:

- Layer: `mixed4d_3x3_bottleneck_pre_relu`
- Strength: `500`
- iter_n: `25`

![Output](https://user-images.githubusercontent.com/38859656/78391770-04e70200-75b5-11ea-8451-06236a9036c2.jpeg)
