# Circuit_IQ_3 API

Enterprise B2B API for text-to-PCB design automation. Convert natural language descriptions into PCB designs and extract component information from datasheets.

## Enterprise Deployment

### System Requirements

- Python 3.8 or higher
- 4GB RAM minimum (8GB recommended)
- 20GB storage space
- Linux/Unix-based OS recommended for production

### Installation

1. Clone the repository:
```bash
git clone https://github.com/YOUR_USERNAME/circuit-iq-3.git
cd circuit-iq-3
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Configure the application:
   - Copy `config.yaml.example` to your preferred location
   - Modify settings according to your environment
   - Set `CIRCUIT_IQ_CONFIG` environment variable to point to your config file

Example production setup:
```bash
# Copy configuration
sudo mkdir -p /etc/circuit-iq-3
sudo cp config.yaml.example /etc/circuit-iq-3/config.yaml
sudo vim /etc/circuit-iq-3/config.yaml

# Create required directories
sudo mkdir -p /var/circuit-iq-3/uploads
sudo mkdir -p /var/circuit-iq-3/results
sudo mkdir -p /var/log/circuit-iq-3

# Set permissions
sudo chown -R circuit-iq:circuit-iq /var/circuit-iq-3
sudo chown -R circuit-iq:circuit-iq /var/log/circuit-iq-3

# Set configuration path
export CIRCUIT_IQ_CONFIG=/etc/circuit-iq-3/config.yaml
```

### Configuration

The application can be configured through:
1. YAML configuration file (recommended for production)
2. Environment variables (override config file settings)

#### Configuration File (config.yaml)

```yaml
# Server Settings
HOST: '0.0.0.0'
PORT: 5000
DEBUG: false

# Upload Settings
MAX_UPLOAD_SIZE: 16777216  # 16MB
UPLOAD_FOLDER: '/var/circuit-iq-3/uploads'
RESULTS_FOLDER: '/var/circuit-iq-3/results'

# Security
API_KEYS_FILE: '/etc/circuit-iq-3/api_keys.json'
USAGE_LOG_FILE: '/var/log/circuit-iq-3/usage.log'
```

#### Environment Variables

All settings can be overridden using environment variables with the `CIRCUIT_IQ_` prefix:

```bash
export CIRCUIT_IQ_HOST=0.0.0.0
export CIRCUIT_IQ_PORT=5000
export CIRCUIT_IQ_MAX_UPLOAD_SIZE=16777216
```

### Running in Production

We recommend using a production-grade WSGI server like Gunicorn:

```bash
gunicorn --bind 0.0.0.0:5000 --workers 4 app:app
```

For high availability, consider using:
- Nginx as a reverse proxy
- Supervisor or systemd for process management
- Redis or Memcached for caching

Example systemd service file (`/etc/systemd/system/circuit-iq-3.service`):
```ini
[Unit]
Description=Circuit IQ 3 API Service
After=network.target

[Service]
User=circuit-iq
Group=circuit-iq
WorkingDirectory=/opt/circuit-iq-3
Environment=CIRCUIT_IQ_CONFIG=/etc/circuit-iq-3/config.yaml
ExecStart=/usr/local/bin/gunicorn --workers 4 --bind 0.0.0.0:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

## Installation

### From GitHub

1. Clone the repository:
```bash
git clone https://github.com/YOUR_USERNAME/circuit-iq-3.git
cd circuit-iq-3
```

2. Create and activate virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

### Directory Structure

```
circuit-iq-3/
├── app.py                    # Main Flask application
├── datasheet_extractor.py    # Component extraction logic
├── pcb_designer.py          # PCB design generation
├── requirements.txt         # Python dependencies
├── README.md               # This file
├── static/                 # Static web assets
├── templates/              # HTML templates
├── uploads/               # Temporary upload storage
└── results/               # Generated PCB files
```

## Quick Start for Business Integration

1. Install Python requirements:
```bash
pip install -r requirements.txt
```

2. Run the server:
```bash
python app.py
```

3. Register your company and get an API key:
```bash
curl -X POST http://localhost:5000/api/register \
  -H "Content-Type: application/json" \
  -d '{"company_name": "Your Company Name"}'
```

4. Use the API key in all subsequent requests:
```bash
curl -X POST http://localhost:5000/api/analyze_requirements \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your_api_key_here" \
  -d '{"text": "I need a simple LED circuit with current limiting resistor"}'
```

## API Integration Guide

### Authentication

All API endpoints (except `/api/register`) require authentication using an API key. Include your API key in the `X-API-Key` header:

```bash
X-API-Key: your_api_key_here
```

### Available Endpoints

- `POST /api/register`: Register your company and get an API key
- `POST /api/analyze_requirements`: Analyze PCB requirements
- `POST /api/extract_from_datasheet`: Extract component information
- `POST /api/design_pcb`: Generate PCB design
- `GET /api/component_types`: List available component types
- `GET /api/usage`: Get your API usage statistics
- `GET /health`: Check API health

### Example Integration

#### 1. Text to PCB Design Flow

```python
import requests
import json

API_KEY = 'your_api_key_here'
BASE_URL = 'http://localhost:5000'
HEADERS = {
    'Content-Type': 'application/json',
    'X-API-Key': API_KEY
}

# 1. Analyze requirements
requirements = {
    'text': 'Design a PCB for an LED circuit with current limiting resistor'
}
analysis = requests.post(f'{BASE_URL}/api/analyze_requirements', 
                        headers=HEADERS,
                        json=requirements)

# 2. Generate PCB design
design_request = {
    'project_name': 'LED Circuit',
    'requirements': requirements['text'],
    'board_params': {
        'layers': 2,
        'width': 50,
        'height': 50
    }
}
design = requests.post(f'{BASE_URL}/api/design_pcb',
                      headers=HEADERS,
                      json=design_request)
```

#### 2. Component Extraction from Datasheet

```python
# Extract from PDF
with open('component.pdf', 'rb') as f:
    files = {'file': f}
    response = requests.post(f'{BASE_URL}/api/extract_from_datasheet',
                           headers={'X-API-Key': API_KEY},
                           files=files)

# Extract from text
text_data = {
    'text': 'Component specifications...'
}
response = requests.post(f'{BASE_URL}/api/extract_from_datasheet',
                        headers=HEADERS,
                        json=text_data)
```

#### 3. Monitor API Usage

```python
usage = requests.get(f'{BASE_URL}/api/usage', headers=HEADERS)
print(json.dumps(usage.json(), indent=2))
```

#### 4. Complete Datasheet to Gerber Workflow

```python
import requests
import json
import zipfile
import os

API_KEY = 'your_api_key_here'
BASE_URL = 'http://localhost:5000'
HEADERS = {
    'Content-Type': 'application/json',
    'X-API-Key': API_KEY
}

def process_datasheet_to_gerber(datasheet_path, project_name):
    """Complete workflow from datasheet to Gerber files"""
    
    # 1. Extract component information from datasheet
    with open(datasheet_path, 'rb') as f:
        files = {'file': f}
        response = requests.post(
            f'{BASE_URL}/api/extract_from_datasheet',
            headers={'X-API-Key': API_KEY},
            files=files
        )
    
    if response.status_code != 200:
        raise Exception(f"Datasheet extraction failed: {response.json()['message']}")
    
    component = response.json()['component']
    
    # 2. Generate requirements text from component
    requirements = f"Design a PCB for {component['name']} with the following specifications:\n"
    requirements += f"- Package: {component.get('package', 'Standard')}\n"
    requirements += f"- Pins: {component.get('pins', 0)}\n"
    requirements += f"- Power requirements: {component.get('power_specs', 'Standard')}\n"
    
    # 3. Analyze requirements
    analysis = requests.post(
        f'{BASE_URL}/api/analyze_requirements',
        headers=HEADERS,
        json={'text': requirements}
    ).json()
    
    # 4. Generate PCB design with Gerber files
    design_request = {
        'project_name': project_name,
        'requirements': requirements,
        'component': component,
        'board_params': {
            'layers': 2,
            'width': analysis['recommended_size']['width'],
            'height': analysis['recommended_size']['height']
        }
    }
    
    design = requests.post(
        f'{BASE_URL}/api/design_pcb',
        headers=HEADERS,
        json=design_request
    ).json()
    
    if design['status'] != 'success':
        raise Exception(f"PCB design generation failed: {design['message']}")
    
    # 5. Download and package Gerber files
    design_id = design['design_id']
    gerber_files = design['gerber_files']
    
    # Create output directory
    output_dir = f"{project_name}_gerber_files"
    os.makedirs(output_dir, exist_ok=True)
    
    # Download each Gerber file
    for gerber in gerber_files:
        response = requests.get(
            f"{BASE_URL}{gerber['url']}",
            headers=HEADERS
        )
        
        with open(f"{output_dir}/{gerber['name']}", 'wb') as f:
            f.write(response.content)
    
    # Create ZIP archive
    with zipfile.ZipFile(f"{project_name}_gerber.zip", 'w') as zipf:
        for root, dirs, files in os.walk(output_dir):
            for file in files:
                zipf.write(os.path.join(root, file), file)
    
    return {
        'gerber_zip': f"{project_name}_gerber.zip",
        'preview_url': design['preview_url'],
        'component': component,
        'board_specs': design['board_dimensions']
    }

# Example usage
if __name__ == '__main__':
    result = process_datasheet_to_gerber(
        datasheet_path='component_datasheet.pdf',
        project_name='MyComponent_PCB'
    )
    print(f"Generated Gerber files: {result['gerber_zip']}")
    print(f"Board dimensions: {result['board_specs']}")
```

This script demonstrates:
1. Extracting component data from a datasheet
2. Converting specifications into requirements
3. Analyzing requirements for optimal board size
4. Generating a complete PCB design
5. Downloading and packaging Gerber files

The output Gerber ZIP will contain:
- Copper layers (.GTL, .GBL)
- Solder mask (.GTS, .GBS)
- Silkscreen (.GTO, .GBO)
- Drill files (.TXT)
- Board outline (.GKO)

These files can be directly used for PCB manufacturing.

## Server Configuration

You can customize the server using environment variables:

- `PORT`: Change the port (default: 5000)
- `HOST`: Change the host (default: 127.0.0.1)
- `DEBUG`: Enable debug mode (default: False)

## Notes

- Maximum file upload size is 16MB
- Uploaded files are automatically cleaned up after processing
- The server includes caching for improved performance
- Usage statistics are tracked per API key
- All API responses follow a consistent format:
  ```json
  {
    "status": "success|error",
    "message": "Optional message",
    "data": {}  // Optional data object
  }
  ``` 
