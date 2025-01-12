---
layout: post
title:  "Running Jekyll in a Rootless Podman Container: Part 3 - Reproducible Development and Testing"
date: 2023-04-15 11:58:48
description: "The final part of our series on using Podman rootless containers with Jekyll. This post focuses on testing, running a reproducible server with `podman-compose`."
tags: Jekyll, Podman, Containers, Rootless, Development, Testing
categories: fries-al-folio
---

---

Welcome to the third and final part of our series on running Jekyll in a rootless Podman container. In this post, we'll focus on using `podman-compose` to run and test your Jekyll site in a more reproducible way. This builds upon the concepts and techniques discussed in [Part 1](link-to-part-1) and [Part 2](link-to-part-2) of the series. The relevant files are in this folder:

```text
~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
```
and specifically the file `docker-compose-project-podman.yml`.

## Running Jekyll with `podman-compose`

In the previous part, we explored how to run the container with manual commands. However, for reproducible testing and development, it is better to use `podman-compose`. Let's explore the `docker-compose-project-podman.yml` file:

```yaml
version: "3"

services:
  jekyll:
    image: pypak-jekyll-srv
    container_name: al-folio-website
    command: bash -c "
      rm -f Gemfile.lock
      && bundler exec jekyll serve --watch --port=8080 --host=0.0.0.0 --livereload --verbose --trace"
    ports:
      - 8080:8080
    volumes:
      - ../../..:/home/app/project:Z  # Note: PyCharm maps '.' to /opt/project/ with environment variable IDE_PROJECT_ROOTS
      - .:/home/app/srv/jekyll:Z
#    environment:
#      - JEKYLL_ROOTLESS=1  # https://github.com/envygeeks/jekyll-docker/blob/master/README.md
```
Here is a breakdown of the configuration:
*   **`version: "3"`**: Specifies the version of the Docker Compose file format.
*   **`services`**: Defines the different services (containers) that will be part of the application. In this case, we only have one service, called `jekyll`.
*   **`jekyll`**: The definition of the Jekyll service which has the following properties:
    *   `image: pypak-jekyll-srv`: The Docker image to be used for this service.
    *   `container_name: al-folio-website`: Sets the name for the container.
    *   `command: bash -c "..."`: The command to run within the container. This command:
        *   Removes `Gemfile.lock` to ensure that dependencies are updated.
        *   Starts the Jekyll server with `bundler exec jekyll serve ...`.
        *   `--watch`: Enables automatic rebuilding when there are changes to the source files.
        *  `--port=8080`: The port where the jekyll server will run.
        *   `--host=0.0.0.0`: The host where the jekyll server will be served.
        *   `--livereload`: Enables livereload for a better development experience.
        *   `--verbose` and `--trace` are used to provide additional information.
    *   `ports: - 8080:8080`: Maps port 8080 from the host to port 8080 in the container.
    *   `volumes`: Maps directories from your host to the container:
        *  `../../..:/home/app/project:Z`: Maps the directory which contains the jekyll code to `/home/app/project` inside the container.
        *  `.:/home/app/srv/jekyll:Z`: Maps the current directory to `/home/app/srv/jekyll`.

## Running the Jekyll Site

Let's test our Jekyll site by running the service with `podman-compose`:

```bash
podman-compose -f docker-compose-project-podman.yml up
```

This command will start the Jekyll container as defined in the docker-compose file.

You can view the website by going to your browser and navigating to the address `localhost:8080`.

You can shut down the service with the `down` command:
```bash
podman-compose -f docker-compose-project-podman.yml down
```

## Debugging and Testing

To debug the container you can use `podman exec` to execute commands directly in your container:

```bash
podman exec -it al-folio-website /bin/bash
```

This command opens a new bash terminal in your running container, which can be useful to debug your setup.

If for some reason you need to create a container manually, this is how you do it:

```bash
podman container run -it --rm pypak-jekyll-srv /bin/bash
```

This will create an instance of a container, which you can use to verify that the container works as expected.
You can check if the containers are running with the following command:

```bash
podman ps
```
Finally, you can remove the container by using the container name:
```bash
podman container rm al-folio-website
```

## Conclusion

In this third part, we’ve seen how `podman-compose` can make your development workflow reproducible and easier to manage. We've also shown how to inspect and test the container by executing commands or running a new instance. This approach is very useful when you need to debug or verify the behaviour of your container.

This concludes our series on running Jekyll in a rootless Podman container. We've explored key concepts such as rootless containers, file ownership, user mapping and running containers using `podman-compose`. Hopefully, you can use these principles for your own projects.
