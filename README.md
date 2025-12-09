# NS8 Stirling-PDF Module

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![NS8 Module](https://img.shields.io/badge/NS8-Module-green.svg)](https://www.nethserver.org/)

A comprehensive NS8 module for deploying [Stirling-PDF](https://github.com/Stirling-Tools/Stirling-PDF), a powerful, locally hosted PDF manipulation tool. This module provides a one-stop-shop for all your PDF needs with features like merging, splitting, converting, and securing PDF documents.

## Features

- **PDF Manipulation**: Merge, split, rotate, and reorder PDF pages
- **Format Conversion**: Convert between PDF and various image formats
- **Security**: Add passwords, watermarks, and digital signatures
- **OCR**: Extract text from scanned PDFs
- **Compression**: Reduce PDF file sizes while maintaining quality
- **Web Interface**: Modern, responsive web UI built with Vue.js
- **NS8 Integration**: Seamless integration with NS8 ecosystem including:
  - Traefik reverse proxy with automatic SSL
  - Centralized authentication and user management
  - Backup and restore capabilities
  - Multi-instance support

## Prerequisites

- NS8 server with admin access
- Valid domain name (for Let's Encrypt certificates)
- At least 1GB RAM and 2GB disk space per instance

## Installation

### Quick Install

Deploy the module with a single command:

```bash
add-module ghcr.io/geniusdynamics/stirlingpdf:latest 1
```

The command returns the instance details:

```json
{
  "module_id": "stirlingpdf1",
  "image_name": "stirlingpdf",
  "image_url": "ghcr.io/geniusdynamics/stirlingpdf:latest"
}
```

### Multiple Instances

Deploy multiple instances for different departments or use cases:

```bash
add-module ghcr.io/geniusdynamics/stirlingpdf:latest 3
```

## Configuration

### Basic Configuration

Configure your Stirling-PDF instance with essential settings:

```bash
api-cli run configure-module --agent module/stirlingpdf1 --data - <<EOF
{
  "host": "pdf.yourdomain.com",
  "http2https": true,
  "lets_encrypt": true
}
EOF
```

### Advanced Configuration Options

| Parameter                | Type    | Required | Default | Description                                              |
| ------------------------ | ------- | -------- | ------- | -------------------------------------------------------- |
| `host`                   | string  | Yes      | -       | Fully qualified domain name (e.g., `pdf.yourdomain.com`) |
| `http2https`             | boolean | Yes      | true    | Redirect HTTP to HTTPS                                   |
| `lets_encrypt`           | boolean | Yes      | true    | Request Let's Encrypt SSL certificate                    |
| `docker_enable_security` | boolean | No       | false   | Enable password protection                               |
| `security_enablelogin`   | boolean | No       | false   | Require login authentication                             |

### Security Configuration

When enabling login protection, you can set custom credentials:

```bash
api-cli run configure-module --agent module/stirlingpdf1 --data - <<EOF
{
  "host": "pdf.yourdomain.com",
  "http2https": true,
  "lets_encrypt": true,
  "docker_enable_security": true,
  "security_enablelogin": true,
  "security_initiallogin_username": "admin",
  "security_initiallogin_password": "your-secure-password"
}
EOF
```

**Default credentials** (when login is enabled without custom values):

- Username: `admin`
- Password: `stirling`

⚠️ **Security Note**: Change default credentials immediately after first login.

## Management

### View Configuration

Retrieve current configuration:

```bash
api-cli run get-configuration --agent module/stirlingpdf1
```

### Update Module

Update to the latest version:

```bash
api-cli run update-module --data '{
  "module_url": "ghcr.io/geniusdynamics/stirlingpdf:latest",
  "instances": ["stirlingpdf1"],
  "force": true
}'
```

### Backup and Restore

The module supports NS8's built-in backup system. All PDF files and configurations are automatically included in system backups.

### Uninstall

Remove the instance (data will be preserved unless `--no-preserve` is used):

```bash
# Remove with data preservation
remove-module stirlingpdf1

# Remove completely (data lost)
remove-module --no-preserve stirlingpdf1
```

## Architecture

### Container Structure

The module deploys the following containers:

- **stirlingpdf-app**: Main Stirling-PDF application (Nginx + PHP)
- **stirlingpdf**: Application backend services
- **Traefik**: Reverse proxy with SSL termination

### File Structure

```
/home/stirlingpdf1/
├── .config/
│   ├── state/          # Environment variables and configuration
│   └── bin/            # Helper scripts
├── data/               # Persistent data storage
├── logs/               # Application logs
└── tmp/                # Temporary files
```

### Service Management

Services are managed by systemd:

```bash
# View service status
systemctl --user status stirlingpdf-app.service

# Restart services
systemctl --user restart stirlingpdf-app.service
```

## Troubleshooting

### Debug Mode

Access the module environment for debugging:

```bash
# Enter module environment
runagent -m stirlingpdf1

# View environment variables
runagent -m stirlingpdf1 env

# Check running containers
runagent -m stirlingpdf1 podman ps
```

### Container Debugging

Inspect and debug individual containers:

```bash
# View container logs
runagent -m stirlingpdf1 podman logs stirlingpdf-app

# Access container shell
runagent -m stirlingpdf1 podman exec -it stirlingpdf-app sh

# Check container environment
runagent -m stirlingpdf1 podman exec stirlingpdf-app env
```

### Common Issues

| Issue                  | Solution                                                            |
| ---------------------- | ------------------------------------------------------------------- |
| SSL certificate errors | Ensure domain DNS is correct and Let's Encrypt can reach the server |
| High memory usage      | Increase available RAM or limit concurrent PDF operations           |
| Slow performance       | Check disk space and consider SSD storage for better I/O            |
| Login failures         | Verify security settings and reset credentials if needed            |

### Log Locations

- Application logs: `/home/stirlingpdf1/logs/`
- Systemd logs: `journalctl --user -u stirlingpdf-app.service`
- Traefik logs: Available through NS8 logging system

## Development

### Testing

Run the test suite using Robot Framework:

```bash
./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/stirlingpdf:latest
```

### UI Development

The web interface is built with Vue.js 2 and Carbon Design System:

```bash
cd ui/
npm install
npm run serve    # Development server
npm run build    # Production build
npm run lint     # Code linting
```

### Module Structure

- `imageroot/`: Container images and configuration
- `ui/`: Vue.js web interface
- `tests/`: Robot Framework test cases
- `actions/`: NS8 module lifecycle scripts

## Integration

### Smarthost Configuration

The module automatically integrates with NS8's centralized smarthost configuration for email notifications and external services.

### API Access

Stirling-PDF provides REST API endpoints for automation. Access the API documentation at `https://your-domain.com/api/docs` after installation.

### Multi-Tenant Support

Deploy multiple instances for different departments or clients, each with isolated data and configuration.

## Security

### Best Practices

1. **Always use HTTPS** with valid SSL certificates
2. **Change default credentials** immediately
3. **Regular updates** to patch security vulnerabilities
4. **Network isolation** through NS8 firewall rules
5. **Access logging** for audit trails

### Data Protection

- All PDF files are stored locally on the NS8 server
- No data is transmitted to external services
- Encrypted storage options available through NS8

## Support

### Documentation

- [Stirling-PDF Official Docs](https://stirlingtools.com/docs/Overview/What%20is%20Stirling-PDF)
- [NS8 Core Documentation](https://www.nethserver.org/docs/)

### Community

- **Issues**: [GitHub Issues](https://github.com/geniusdynamics/dev/issues)
- **Discussions**: [GitHub Discussions](https://github.com/geniusdynamics/ns8-Stirling-PDF/discussions)
- **NS8 Community**: [NethServer Forum](https://community.nethserver.org/)

### Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## Translation

The UI is translated using [Weblate](https://hosted.weblate.org/projects/ns8/).

### Adding Translations

1. Add the [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
2. Request addition to the NS8 Weblate project
3. Contribute translations through the Weblate interface

### Supported Languages

- English (en)
- German (de)
- Spanish (es)
- Basque (eu)
- Italian (it)
- Portuguese (pt, pt_BR)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Changelog

### Version History

- **v1.0.0**: Initial release with basic PDF manipulation features
- **v1.1.0**: Added security and authentication options
- **v1.2.0**: Enhanced UI with Vue.js and Carbon Design System
- **Current**: Latest stable version with full NS8 integration

---

**Note**: This module is part of the NS8 ecosystem. For general NS8 questions, visit [nethserver.org](https://www.nethserver.org/).
