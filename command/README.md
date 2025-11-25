# Command
- CLI (Command Line Interface) tools that allow user to interact with the Docker engine to manage containers, images, networks, and volumes. 
- These commands provide the primary interface for:
    - **Container Lifecycle Management** - Creating, starting, stopping, and removing containers
    - **Image Management** - Pulling, building, listing, and removing Docker images  
    - **Container Interaction** - Executing commands inside running containers
    - **System Administration** - Monitoring resources and maintaining Docker environments

## Basic syntax

```bash
docker [OPTIONS] COMMAND [ARG...]
```

**Components:**
- `docker` - The Docker CLI client
- `[OPTIONS]` - Global options that apply to the Docker command
- `COMMAND` - The specific Docker command to execute (e.g., run, build, ps)
- `[ARG...]` - Arguments specific to the command

## Environment Variables
- Dynamic values stored in the operating system
- Can affect how Docker CLI behaves
- Provide a way to configure Docker without modifying configuration files or passing command-line options repeatedly.
`os.environ.get('ENV_NAME')` : Python method used to retrieve environment variable values safely.

## List of Commands

- [build](build/README.md): Build Docker images from a Dockerfile and build context
- [exec](exec/README.md): Execute commands inside running Docker containers
- [history](history/README.md): Show the history of an image with layer information
- [images](images/README.md): List Docker images stored locally on the system
- [inspect](inspect/README.md): Display detailed information about Docker objects in JSON format
- [login](login/README.md): Authenticate Docker client with a registry server
- [logs](logs/README.md): Display and follow container logs for debugging and monitoring
- [manifest](manifest/README.md): Create and manage Docker image manifests and manifest lists
- [network](network/README.md): Manage Docker networks for container communication
- [ps](ps/README.md): List Docker containers with their status and configuration
- [pull](pull/README.md): Download Docker images from a registry to local system
- [push](push/README.md): Upload Docker images from local system to a remote registry
- [rm](rm/README.md): Remove one or more containers from the system
- [rmi](rmi/README.md): Remove one or more Docker images from the local system
- [run](run/README.md): Create and start a new container from a Docker image
- [stop](stop/README.md): Stop one or more running containers gracefully
- [system](system/README.md): Manage Docker system-wide information and resources
- [tag](tag/README.md): Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE