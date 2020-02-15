---
title: 'Squashing Docker Images'
date: '2020-02-15'
categories: [ops]
tags: ['docker']
---

I was wondering the effect of merging layers (squashing) on the size of an image. Now Docker provides an experimental `--squash` option for the build.

> Squash newly built layers into a single new layer

In order to illustrate the impact I have chosen a simple example that has room to be improved by squashing layers.
The example installs `pandas` in a `miniconda` image for Python 3.

```dockerfile
FROM continuumio/miniconda3:latest

RUN conda install --quiet --yes 'pandas'
RUN  conda clean --all -f -y

CMD ["python", "-c", "import pandas; print(pandas.__version__);"]
```

Let's build and run it.

```bash
$ docker build --force-rm -t pandas .

$ docker run --rm pandas

# 1.0.1
```

So far so good. Checking the size of the image and the corresponding layers.

```bash
$  docker images --filter=reference='pandas'                    

# REPOSITORY  TAG     IMAGE ID      CREATED        SIZE
# pandas      latest  6a097f0c0eae  3 minutes ago  1.48GB

# 5 Layers
$ docker inspect --format '{{range .RootFS.Layers}}{{printf "%s\n" .}}{{end}}' pandas
 
# sha256:2db44bce66cde56fca25aeeb7d09dc924b748e3adfe58c9cc3eb2bd2f68a1b68
# sha256:3ee7190fd43a351744a978485681a07109d66997c522ad3583860965439e1828
# sha256:5215cb249b792178bbfd0562910c157435697f159508635da513e6a0709869b6
# sha256:4af92f6fd12d224d7df98a054c53168f2d3594d8cd70c8ba0c7e28df31ac7858
# sha256:f22c6f2f28a966a5b2d62448f7dda9364595fa6daf9769486654ee108366ce80

# The layers at the bottom are the layers of Miniconda base image
$ docker history pandas

# IMAGE         CREATED         CREATED BY                                      SIZE                COMMENT
# 6a097f0c0eae  14 minutes ago  /bin/sh -c #(nop)  CMD ["python" "-c" "impor…   0B                  
# e62ed6b5d6d8  16 minutes ago  /bin/sh -c conda clean --all -f -y              0B                  
# 2fe14c3fd8e7  16 minutes ago  /bin/sh -c conda install --quiet --yes 'pand…   1.05GB              
# 406f2b43ea59  4 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
# <missing>     4 months ago    /bin/sh -c wget --quiet https://repo.anacond…   151MB               
# <missing>     4 months ago    /bin/sh -c apt-get update --fix-missing &&  …   210MB               
# <missing>     4 months ago    /bin/sh -c #(nop)  ENV PATH=/opt/conda/bin:/…   0B                  
# <missing>     4 months ago    /bin/sh -c #(nop)  ENV LANG=C.UTF-8 LC_ALL=C…   0B                  
# <missing>     5 months ago    /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
# <missing>     5 months ago    /bin/sh -c #(nop) ADD file:1901172d265456090…   69.2MB     
```

Summary

- Size: **1.48GB** -- ouch
- Nb layers: **5**

# Squash

Let's try the new `--squash` option. To be able to use this option you have to **turn on experimental feature on the Docker daemon**.

```bash
# The build with the squash option
$ docker build --force-rm --squash -t pandas .

$  docker images --filter=reference='pandas'  

# REPOSITORY  TAG     IMAGE ID      CREATED        SIZE
# pandas      latest  82f65007af87  3 minutes ago  1.29GB

# 4 layers
$ docker inspect --format '{{range .RootFS.Layers}}{{printf "%s\n" .}}{{end}}' pandas 

# sha256:2db44bce66cde56fca25aeeb7d09dc924b748e3adfe58c9cc3eb2bd2f68a1b68
# sha256:3ee7190fd43a351744a978485681a07109d66997c522ad3583860965439e1828
# sha256:5215cb249b792178bbfd0562910c157435697f159508635da513e6a0709869b6
# sha256:322504279862275d0a2e00aaec5f41eb7009765baff817e6347bd3992082f17e

$ docker history pandas 

# IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
# 82f65007af87        7 minutes ago                                                       858MB               merge # sha256:5a32e51cdbb504aa518d92847a98b00f6cd11fb5dcd33a3903daae6197c5283a to sha256:406f2b43ea59a121345b188cc94595c539014c5b644bf95c61458a9b5b2905ba
# <missing>           11 minutes ago      /bin/sh -c #(nop)  CMD ["python" "-c" "impor…   0B                  
# <missing>           11 minutes ago      /bin/sh -c conda clean --all -f -y              0B                  
# <missing>           11 minutes ago      /bin/sh -c conda install --quiet --yes 'pand…   0B                  
# <missing>           4 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
# <missing>           4 months ago        /bin/sh -c wget --quiet https://repo.anacond…   151MB               
# <missing>           4 months ago        /bin/sh -c apt-get update --fix-missing &&  …   210MB               
# <missing>           4 months ago        /bin/sh -c #(nop)  ENV PATH=/opt/conda/bin:/…   0B                  
# <missing>           4 months ago        /bin/sh -c #(nop)  ENV LANG=C.UTF-8 LC_ALL=C…   0B                  
# <missing>           5 months ago        /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
# <missing>           5 months ago        /bin/sh -c #(nop) ADD file:1901172d265456090…   69.2MB   
```

Obviously layers at the bottom coming from the base image are not squashed. In the history we can see an additional step (at the top) with a comment `merge sha256xx to sha256:yyy`. This is where the squash has occurred.

Summary

- Size: **1.29GB** -> **~ -190MB (-13%)** not so bad
- Nb layers: **4**

# Explanation

The size reduction is not magical. Squashing layers avoid to store one layer before the clean step and one layer after. In consequence only the cleaned layer is kept and so the saving is directly related to the cleaning. We can check if this hypothesis is valid by performing the `clean` manually in the image.

```bash
$ docker run --rm -it pandas bash

# Checking the size before cleaning
$ du -s -B MB /opt/conda

# 1221MB  /opt/conda

# Cleaning
$ conda clean --all -q -f -y

# Checking the size before cleaning
$ du -s -B MB /opt/conda

# 1026MB  /opt/conda
```

It's consistent since we can see that the ~190 MB are saved by the `clean` step.

# Alternatives

There is some alternatives to the `--squash` options however it's always a tradeoff with something else.

## Crafting `Dockerfiles`

Best practices on building `dockerfiles` have led to avoid this inconvenient by using big one-liner commands.
The drawback is that `dockerfiles` loose in readability and some of them end with endless one-liner spanning on multiple lines--it's a paradox.

```dockerfile
FROM continuumio/miniconda3:latest

# Only one layer
RUN conda install --quiet --yes 'pandas' && \
    conda clean --all -f -y

CMD ["python", "-c", "import pandas; print(pandas.__version__);"]
```

Let's see the result with this well crafted `Dockerfile`.

```bash
# No size overhead
$ docker images --filter=reference='pandas'          

# REPOSITORY  TAG     IMAGE ID      CREATED        SIZE
# pandas      latest  6db3ad2d6438  5 seconds ago  1.29GB

# 4 layers
$ squash docker inspect --format '{{range .RootFS.Layers}}{{printf "%s\n" .}}{{end}}' pandas 

# sha256:2db44bce66cde56fca25aeeb7d09dc924b748e3adfe58c9cc3eb2bd2f68a1b68
# sha256:3ee7190fd43a351744a978485681a07109d66997c522ad3583860965439e1828
# sha256:5215cb249b792178bbfd0562910c157435697f159508635da513e6a0709869b6
# sha256:6b9cf98f6831e55f95fe7f5876ccc9befe648504921e2032ec45f80ae4995ec9

$ squash docker history pandas 

# IMAGE         CREATED         CREATED BY                                      SIZE   COMMENT
# 6db3ad2d6438  32 seconds ago  /bin/sh -c #(nop)  CMD ["python" "-c" "impor…   0B     
# 2f4c8e61f521  33 seconds ago  /bin/sh -c conda install --quiet --yes 'pand…   858MB  
# 406f2b43ea59  4 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B     
# <missing>     4 months ago    /bin/sh -c wget --quiet https://repo.anacond…   151MB  
# <missing>     4 months ago    /bin/sh -c apt-get update --fix-missing &&  …   210MB  
# <missing>     4 months ago    /bin/sh -c #(nop)  ENV PATH=/opt/conda/bin:/…   0B     
# <missing>     4 months ago    /bin/sh -c #(nop)  ENV LANG=C.UTF-8 LC_ALL=C…   0B     
# <missing>     5 months ago    /bin/sh -c #(nop)  CMD ["bash"]                 0B     
# <missing>     5 months ago    /bin/sh -c #(nop) ADD file:1901172d265456090…   69.2MB 
```

As expected the result is the same with a crafted `Dockerfile`.

## Multi-stage builds

The idea is to use a multi-stage build to copy all the layers into a single layer. I think this is the worst thing to do however I mention it since this solution is discussed see [here](https://github.com/moby/moby/issues/34565) for example. For this reason I will not develop this solution. If you are interested in finding alternatives, see also [this question](https://stackoverflow.com/questions/55220569/alternative-to-using-squash-when-building-docker-images-on-windows-using-local) on SO.

# Wrap up

Squashing image at the build has several benefits

- **Lighter images**: The squashing will optimize the image size. However if `Dockerfiles` are written with layer optimization in mind the size reduction will be negligible. 
- **Cleaner Dockerfiles**: This is obviously the main advantage since you can will write Dockerfile with readability in mind rather than focusing on layer optimization. Computers are better than human on these low level tasks.
- **Avoid accidental leaking**: Sometimes it is not desirable to give access to intermediate layers. They may contain information you don't want to share.
  
Other potential unconfirmed--for me--benefits

- **Transfer / Storage optimization**: The reduction of the number of layers may have a positive impact on registries to host transfers (`push` / `pull`).
- **Faster builds**: Could lead to faster builds since cache has not to be cleaned at each step?  
