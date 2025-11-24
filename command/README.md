# Command
- CLI (Command Line Interface) tools that allow user to interact with the Docker engine to manage containers, images, networks, and volumes. 
- These commands provide the primary interface for:
    - **Container Lifecycle Management** - Creating, starting, stopping, and removing containers
    - **Image Management** - Pulling, building, listing, and removing Docker images  
    - **Container Interaction** - Executing commands inside running containers
    - **System Administration** - Monitoring resources and maintaining Docker environments

## List of Commands

- [build](build/README.md): Build Docker images from a Dockerfile and build context
- [exec](exec/README.md): Execute commands inside running Docker containers
- [history](history/README.md): Show the history of an image with layer information
- [images](images/README.md): List Docker images stored locally on the system
- [inspect](inspect/README.md): Display detailed information about Docker objects in JSON format
- [login](login/README.md): Authenticate Docker client with a registry server
- [logs](logs/README.md): Display and follow container logs for debugging and monitoring
- [manifest](manifest/README.md): Create and manage Docker image manifests and manifest lists
- [ps](ps/README.md): List Docker containers with their status and configuration
- [pull](pull/README.md): Download Docker images from a registry to local system
- [push](push/README.md): Upload Docker images from local system to a remote registry
- [rm](rm/README.md): Remove one or more containers from the system
- [rmi](rmi/README.md): Remove one or more Docker images from the local system
- [run](run/README.md): Create and start a new container from a Docker image
- [stop](stop/README.md): Stop one or more running containers gracefully
- [system](system/README.md): Manage Docker system-wide information and resources
- [tag](tag/README.md): Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE