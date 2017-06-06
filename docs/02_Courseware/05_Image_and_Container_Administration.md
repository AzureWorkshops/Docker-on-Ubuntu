## Overview
You've just learned quite a bit in working with containers.  As a matter of fact, the majority of container management has already been covered.  We're now going to bring our knowledge of container and image management around full-circle.  This will complete the administration portion of the workshop.

## Tagging Images
There will be instances, like DevOps for instance, where we will need to _tag_ our images.  Tags allow us to apply labels to our images.  This is useful for tracking changes, such as in versioning, to our images.

Let's create a simple derivative of the Ubuntu image we downloaded previously.

Instantiate a new Ubuntu container by typing the following:
```bash
docker run -it ubuntu /bin/bash
```

This will place you at the command prompt _inside_ of the running container.  Now, let's interact with the OS by typing the following commands.
```bash
cd
mkdir test
cd test
echo "This is some sample text." > test.txt
exit
```
We've just created a new directory with a test text file in the user's home directory. If I view my available containers (`docker ps -a`), I'll find the id of the container I just exited from (look under the _Status_ column for the container that was just exited). In my case (see the following output), the container's id is `335abd61d52d`.
```bash
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
335abd61d52d        ubuntu              "/bin/bash"         3 minutes ago       Exited (0) About a minute ago                       blissful_wing
094012b145c8        ubuntu              "sleep inf"         About an hour ago   Exited (137) 27 minutes ago                         vigorous_yalow
a883ff18a967        docker/whalesay     "cowsay Hola!"      27 hours ago        Exited (0) 17 minutes ago                           sad_goldstine
baa0591c4392        hello-world         "/hello"            27 hours ago        Exited (0) 27 hours ago                             jovial_raman
```

Let's restart the container and check to make sure our text file is still there (just to confirm). Again, replace the id below with your container's id.
```bash
docker start -i 335
```
At the prompt type in:
```bash
ls ~/test
```
This should list a file named `test.txt`.  Now, type `exit` to exit out of the container.

We now have a customized container based on our Ubuntu image.  Let's create our own image with it's tag. We're also going to add a message and an author to the image's metadata. Once again, replace the `335` below with the id of your stopped container.
```bash
docker commit -m "added test.txt" -a "Some User" 335 mynamespace/testtext:v1
```

With this command, again, I'm added a message (`-m`) to describe the image and an author (`-a`) to inform of the author.  The `335` is the first 3 characters of my stopped, modified container.  Finally, I've supplied a namespace (`mynamespace/`), an image name (`testtext`), and a tag (`v1`).

The namespace is optional, but a good practice to differentiate between images that might have the same name.  For example, if you and another developer are working on two separate images and you have them both locally, it's easier to keep track of who's image belongs to who.

Now execute the following command:
```bash
docker images
```

You should see something similar to the following:
```bash
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
mynamespace/testtext   v1                  556c25bff4b1        5 minutes ago       118MB
ubuntu                 latest              7b9b13f7b9c0        3 days ago          118MB
a11smiles/softcover    latest              306f23683872        3 months ago        5.74GB
hello-world            latest              48b5124b2768        4 months ago        1.84kB
docker/whalesay        latest              6b362a9f73eb        2 years ago         247MB
```

Notice that you now have your custom image with its tag. Also, because our text file isn't very large, our image has, virtually, the same size as that of the `ubuntu` image (118MB).

We can then instantiate a container based on our image by running the following:
```bash
docker run -it mynamespace/testtext:v1 /bin/bash
```

(NOTE: In all the previous times we've run this command, we've never had to specify a tag because the `latest` tag is implied.  In our case, the tag we used is `v1` so we have to specify it.)

This will place us, once again, inside the container.  Run the following command in the container:
```bash
cat ~/test/test.txt
```
This will show the contents of the file we added earlier.

Now we can exit out of the container by simply typing in `exit`.

In the host machine, typing in `docker ps -a` shows us that our custom image instantiated a container which just exited successfully.
```bash
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS                           PORTS               NAMES
a215acbb7981        mynamespace/testtext:v1   "/bin/bash"         3 minutes ago       Exited (0) 13 seconds ago                            inspiring_spence
335abd61d52d        ubuntu                    "/bin/bash"         26 minutes ago      Exited (0) 16 minutes ago                            blissful_wing
094012b145c8        ubuntu                    "sleep inf"         About an hour ago   Exited (137) About an hour ago                       vigorous_yalow
a883ff18a967        docker/whalesay           "cowsay Hola!"      28 hours ago        Exited (0) 39 minutes ago                            sad_goldstine
baa0591c4392        hello-world               "/hello"            28 hours ago        Exited (0) 28 hours ago                              jovial_raman
```

Finally, if we inspect our custom image (`docker image inspect mynamespace/testtext:v1`), we will see the comment and author attributes displaying "added test.txt" and "Some User", respectively. And, the top layer of our history (`docker image history mynamespace/testtext:v1`) shows us entering into the `bash` shell.

## Deleting Containers
We can clean up disk space by removing unused containers and images.  However, we cannot remove any images that currently have dependent containers - even containers that have stopped.  Therefore, we must delete dependent containers first.

For our example, let's suppose that we no longer need the `ubuntu` container anymore because we've customized it (e.g. added our own text file).  We can delete the `ubuntu` container by typing the following:
```bash
docker rm 335
```

Let's also delete our `hello-world` container:
```bash
docker rm baa
```

Remember, for the previous two commands, substitute your respective container ids.
## Deleting Images
Deleting images are just as easy.  First, let's refresh ourselves on our locally installed images. Running `docker images` produces the following output:
```bash
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
mynamespace/testtext   v1                  556c25bff4b1        19 minutes ago      118MB
ubuntu                 latest              7b9b13f7b9c0        3 days ago          118MB
a11smiles/softcover    latest              306f23683872        3 months ago        5.74GB
hello-world            latest              48b5124b2768        4 months ago        1.84kB
docker/whalesay        latest              6b362a9f73eb        2 years ago         247MB
```

Since images, combined with their namespaces and tags, are unique on the local Docker engine, we can delete images by using the full namespace reference (including the tag) or by using the image id.  Let's practice deleting images.

First, let's delete the `hello-world` image:
```bash
docker rmi hello-world
```

As a reminder, the `latest` tag is implied.  If we were to delete our custom image, we would be required to supply the tag because it differs from `latest`.

Running the `docker rmi` command will remove any links between the image and shared layers.  If the layer is no longer required by any other image, the layer is also deleted.

Now, let's attempt to delete the `ubuntu` image:
```bash
docker rmi ubuntu
```

Running this command produces and error - namely, that the image cannot be deleted because there's still a container that depends on it. Running `docker ps -a` shows that this, indeed, is the case (the second container listed below):
```bash
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS                           PORTS               NAMES
a215acbb7981        mynamespace/testtext:v1   "/bin/bash"         20 minutes ago      Exited (0) 17 minutes ago                            inspiring_spence
094012b145c8        ubuntu                    "sleep inf"         About an hour ago   Exited (137) About an hour ago                       vigorous_yalow
a883ff18a967        docker/whalesay           "cowsay Hola!"      28 hours ago        Exited (0) About an hour ago                         sad_goldstine
```

One of the many reasons for this, is to protect against accidental deletion of our containers and images.  However, if are sure we want to delete the image and all its containers, we can _force_ a deletion:
```bash
docker rmi -f ubuntu
```

Besides forcing a delete of the image, notice how the output is different from the previous deletion of the `hello-world` image.  In this last case, only the reference, or link, was removed from the image.  The underlying layers weren't _deleted_.  Why?  Because the custom image that we created earlier still depends on the underlying Ubuntu OS layer(s).  This is one way Docker helps to conserve disk space - shared and reuse of dependencies. Deleting our custom image (and containers) would perform an actual delete of the Ubuntu OS layer(s).