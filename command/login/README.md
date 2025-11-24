# login

## Overview
- Authenticates Docker client with a registry server
- Required before pushing images to or pulling private images from registries
- Stores credentials securely in Docker configuration file
- Supports Docker Hub, private registries, and cloud container registries
- Credentials persist until logout or manual removal
- Essential for accessing private repositories and pushing images

## Basic Syntax
```bash
docker login [OPTIONS] [SERVER]
```

## Common Options

### Basic Options
- `-u, --username` - Username for registry authentication
- `-p, --password` - Password for registry authentication (not recommended for security)
- `--password-stdin` - Read password from stdin (recommended)

### Registry Options
- If no server specified, defaults to Docker Hub
- Can specify custom registry URL (e.g., registry.example.com)
- Supports multiple registry authentications

## How It Works

### Authentication Process
1. **Credential Input** - Prompts for username and password
2. **Registry Connection** - Connects to specified registry server
3. **Authentication** - Validates credentials with registry
4. **Credential Storage** - Saves encrypted credentials locally
5. **Session Establishment** - Creates authenticated session for future operations

### Process Flow
```
docker login → Prompt Credentials → Validate with Registry
                    ↓                       ↓
            Input Username/Password    Authentication Check
                    ↓                       ↓
            Connect to Server       Success → Store Credentials
                                       ↓
                                   Session Active
```

## Best Practices

- **Use stdin for passwords** - Never pass passwords via command line arguments
- **Secure credentials** - Keep Docker config file permissions restricted
- **Logout when done** - Use `docker logout` on shared systems
- **Use access tokens** - Prefer personal access tokens over passwords
- **Verify registry URL** - Ensure connecting to correct registry
- **Enable 2FA** - Use two-factor authentication when available
- **Rotate credentials** - Regularly update passwords and tokens
- **Use credential helpers** - Configure secure credential storage systems
