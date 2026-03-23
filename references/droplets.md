# Droplets

Linux virtual machines with full root access. Use Droplets when you need more control than App Platform, custom networking, GPU compute, or want to run multiple services on one machine.

For quick web app deployments, prefer [App Platform](deploy-to-app-platform.md) instead.

## When to use Droplets vs App Platform

| | App Platform | Droplets |
|---|---|---|
| Setup | Minimal — push to GitHub, it deploys | You manage the server |
| Scaling | Automatic | Manual or with load balancers |
| GPU support | No | Yes (H100, A100, MI300X, etc.) |
| Customization | Limited | Full control |
| Best for | Web apps, APIs, hackathons | ML workloads, custom stacks, GPU inference |

## Create a Droplet

Provision via the [DO Console](https://cloud.digitalocean.com) (Create > Droplets) or CLI:

```bash
doctl compute droplet create my-app \
  --size s-2vcpu-4gb \
  --image ubuntu-22-04-x64 \
  --region nyc3 \
  --ssh-keys <your-ssh-key-id> \
  --wait
```

### Common flags

| Flag | Description |
|---|---|
| `--size` | vCPUs/RAM/disk (e.g., `s-1vcpu-1gb`, `s-2vcpu-4gb`) |
| `--image` | OS image (e.g., `ubuntu-22-04-x64`, `ubuntu-24-04-x64`) |
| `--region` | Datacenter (e.g., `nyc3`, `sfo3`, `ams3`) |
| `--ssh-keys` | SSH key IDs or fingerprints |
| `--user-data-file` | Cloud-init script to run on first boot |
| `--enable-monitoring` | Install monitoring agent |
| `--enable-backups` | Enable daily backups |
| `--tag-names` | Comma-separated tags |

List available sizes:

```bash
doctl compute size list --format Slug,Memory,VCPUs,Disk,PriceMonthly
```

## SSH into a Droplet

```bash
ssh root@<droplet-ip>
```

Get the IP:

```bash
doctl compute droplet list --format ID,Name,PublicIPv4
```

## Set up a Droplet for an AI app

### Cloud-init (automated setup on first boot)

Create a file `setup.sh`:

```bash
#!/bin/bash
apt-get update
apt-get install -y python3 python3-pip python3-venv nginx
python3 -m venv /app/venv
source /app/venv/bin/activate
pip install flask gunicorn openai python-dotenv
```

Pass it during creation:

```bash
doctl compute droplet create my-app \
  --size s-2vcpu-4gb \
  --image ubuntu-22-04-x64 \
  --region nyc3 \
  --ssh-keys <key-id> \
  --user-data-file setup.sh \
  --wait
```

### Manual setup (after SSH)

```bash
# Python + Flask
apt-get update && apt-get install -y python3 python3-pip python3-venv
python3 -m venv /app/venv && source /app/venv/bin/activate
pip install flask gunicorn openai python-dotenv

# Node.js + Next.js
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt-get install -y nodejs
npm install -g pm2
```

### Run with systemd

Create `/etc/systemd/system/myapp.service`:

```ini
[Unit]
Description=My Gradient App
After=network.target

[Service]
User=root
WorkingDirectory=/app
Environment=DIGITAL_OCEAN_MODEL_ACCESS_KEY=your-key
ExecStart=/app/venv/bin/gunicorn --bind 0.0.0.0:8080 --timeout 120 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable myapp && systemctl start myapp
```

## GPU Droplets

For ML training, fine-tuning, or self-hosted inference requiring GPU compute.

### Available GPUs

| GPU | Use case |
|---|---|
| NVIDIA H100 | High-performance training and inference |
| NVIDIA H200 | Large HBM for long sequences |
| NVIDIA A100 | General ML workloads |
| NVIDIA L40S | Tensor-intensive model adaptation |
| AMD MI300X | Massive HBM for fine-tuning |
| AMD MI350X | Latest CDNA 4 architecture |

Configurations from 1 to 8 GPUs. Create via Console or CLI.

Pre-configured images with drivers and ML frameworks available (AI/ML Ready images).

## Firewall

Cloud Firewalls are free. Create via Console (Networking > Firewalls) or CLI:

```bash
doctl compute firewall create \
  --name my-firewall \
  --inbound-rules "protocol:tcp,ports:22,address:0.0.0.0/0 protocol:tcp,ports:80,address:0.0.0.0/0 protocol:tcp,ports:443,address:0.0.0.0/0" \
  --outbound-rules "protocol:tcp,ports:all,address:0.0.0.0/0 protocol:udp,ports:all,address:0.0.0.0/0" \
  --droplet-ids <droplet-id>
```

## Official docs

- [Droplets overview](https://docs.digitalocean.com/products/droplets/)
- [GPU Droplets](https://docs.digitalocean.com/products/gpu-droplets/)
- [Cloud Firewalls](https://docs.digitalocean.com/products/networking/firewalls/)
- [doctl compute droplet create](https://docs.digitalocean.com/reference/doctl/reference/compute/droplet/create/)
