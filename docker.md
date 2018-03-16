Images -

The file system and configuration of our application which are used to create containers.

Containers -

These are running instances of Docker images. Containers run the actual applications. A container includes an application and all of its dependencies. It shares the kernel with other containers, and runs as an isolated process in user space on the host OS.

Docker daemon -

The background service running on the host that manages building, running and distributing Docker containers.

Docker client -

The command line tool that allows the user to interact with the Docker daemon.

Docker Store -

Store is, among other things, a registry of Docker images. You can think of the registry as a directory of all available Docker images.

This flow will help you understanding it better. enter image description here

Summary:

Pull Image form docker hub or build from Dockerfile => Gives a docker Image(not editable).
Run the Image(docker run image_name:tag_name) => Gives a running Image i.e. Container(editable)
