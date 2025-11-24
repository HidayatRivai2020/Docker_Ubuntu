# How to Optimize Image Size
- Use Smaller Base Images
    - Alpine
    - Slim Variants
    - Distroless Images
    - Scratch Images
- Multi-Stage Builds
- Minimize Layers
    - Combine RUN Commands
    - Clean Up in Same Layer
- Optimize Package Installation
    - Remove Package Manager Cache
- Install Only Required Packages
- Use .dockerignore
- Copy Files Efficiently
    - Optimize COPY Order
    - Copy Only What's Needed
- Remove Build Dependencies
    - Install and Remove Build Tools
    - Multi-Stage for Build Tools
- Optimize Application Code
    - Remove Development Dependencies
    - Minimize Application Files
- Compress and Optimize
    - Use Compression Tools
    - Optimize Static Files
- Leverage BuildKit
    - Enable BuildKit Features
    - Build with BuildKit

## Optimization Checklist

### Before Building
- [ ] Choose smallest suitable base image
- [ ] Create comprehensive .dockerignore file
- [ ] Plan multi-stage build if needed
- [ ] Identify build-time-only dependencies

### During Build
- [ ] Combine RUN commands where possible
- [ ] Clean up in same layer as installation
- [ ] Use --no-cache flags for package managers
- [ ] Install only production dependencies
- [ ] Remove build tools after use

### After Build
- [ ] Measure image size
- [ ] Analyze layers with docker history
- [ ] Use dive or similar tools to inspect
- [ ] Test that optimized image works correctly
- [ ] Document optimization decisions

### Continuous Optimization
- [ ] Track image size over time
- [ ] Review base image updates
- [ ] Update dependencies regularly
- [ ] Remove unused dependencies
- [ ] Automate size checks in CI/CD

## Best Practices Summary
- Start Small
- Multi-Stage Everything
- Layer Wisely
- Clean Aggressively
- Ignore Unnecessarily
