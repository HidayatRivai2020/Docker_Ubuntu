# history

## Overview
- Shows the history of an image, displaying each layer and the commands that created them
- Useful for understanding how an image was built and what each layer contains
- Helps identify large layers and optimize image size
- Essential for debugging build issues and security analysis
- Works with both local images and images from registries
- Displays creation time, size, and commands for each layer

## Basic Syntax
```bash
docker history [OPTIONS] IMAGE:[TAG]
```

## Common Options

### Display Options
- `--format` - Format output using a custom template (table, json, or Go template)
- `-H, --human` - Print sizes and dates in human readable format (default: true)
- `--no-trunc` - Don't truncate output - show full commands
- `-q, --quiet` - Only show image IDs

## How It Works

### Layer Analysis Flow
1. **Image Selection** - Identifies the image to analyze
2. **Layer Extraction** - Retrieves all layers from base to top
3. **Metadata Display** - Shows creation time, size, and commands for each layer
4. **Size Calculation** - Displays individual layer sizes and cumulative impact

### Layer Structure
```
docker history → Read Image Layers → Extract Metadata → Display Info
                        ↓                   ↓               ↓
                  Base to Top          Commands        Size & Time
                                       Creators
```

### Image Layer Components
Each layer in the output shows:
- **IMAGE ID** - Layer identifier (partial SHA256)
- **CREATED** - When the layer was created
- **CREATED BY** - The Dockerfile instruction that created the layer
- **SIZE** - Size added by this layer
- **COMMENT** - Additional metadata or comments

## Understanding Output

### Sample Output
```
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
a6bd71f48f68   2 weeks ago    /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B        
<missing>      2 weeks ago    /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B        
<missing>      2 weeks ago    /bin/sh -c #(nop)  EXPOSE 80                    0B        
<missing>      2 weeks ago    /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B       
<missing>      2 weeks ago    /bin/sh -c apt-get update && apt-get install…   54.1MB    
<missing>      2 weeks ago    /bin/sh -c #(nop)  ENV NGINX_VERSION=1.21.0     0B        
<missing>      3 weeks ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      3 weeks ago    /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   72.8MB
```

### Key Points
- **`<missing>`** - Layer exists only in the final image (not stored as separate image)
- **0B layers** - Metadata instructions (CMD, ENV, EXPOSE, etc.)
- **Large layers** - Usually from RUN, COPY, or ADD instructions
- **Order** - Newest layers at top, base layers at bottom

## Best Practices

- **Use `--no-trunc` for debugging** to see complete commands and understand what each layer does
- **Identify large layers** to optimize image size by combining commands
- **Check for secrets** in layer commands before pushing to registries
- **Combine RUN commands** to reduce the number of layers and overall size
- **Use in CI/CD** to fail builds if image has too many layers or exceeds size limits
- **Analyze before and after** optimization to measure improvements
- **Document layer purposes** for team understanding and maintenance
- **Regular review** of image history to identify optimization opportunities
