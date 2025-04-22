# MCP Official Tutorial

This repository contains the code for the Model Context Protocol (MCP) tutorial, demonstrating how to create and use MCP clients and servers to integrate AI models with external tools and services.

## Overview

The repository includes:
- A Python client for interacting with MCP servers using Claude (`mcp-client-python`)
- A Python weather server implementing MCP tools (`weather-server-python`)

This README focuses on the Python implementation.

## Prerequisites

- Python 3.10 or higher
- Git
- An Anthropic API key (instructions below)

## Getting an Anthropic API Key

To use the Claude API, you need an Anthropic API key:

1. **Create an Anthropic account**:
   - Go to [https://console.anthropic.com/](https://console.anthropic.com/)
   - Sign up for a new account if you don't have one

2. **Set up billing**:
   - After logging in, navigate to "Settings" > "Billing" in the console
   - Add a payment method (credit card)
   - Add credits to your account (note that this is separate from Claude Pro subscription)

3. **Create an API key**:
   - Go to "API Keys" in the console
   - Click "Create API Key"
   - Give your key a name (e.g., "MCP Tutorial")
   - Copy the key immediately (you won't be able to see it again)

## Installation

Clone the repository and set up the environment:

```bash
# Clone the repository
git clone git@github.com:Private-Super-Dev/mcp-official-tutorial.git
cd mcp-official-tutorial

# Create and activate a virtual environment
python -m venv .venv

# On macOS/Linux:
source .venv/bin/activate

# Install client dependencies
cd mcp-client-python
pip install -e .

# Install server dependencies
cd ../weather-server-python
pip install -e .

# Return to the project root
cd ..
```

## Configuration

Set up your Anthropic API key:

```bash
# Go to the client directory
cd mcp-client-python

# Create a .env file from the example
touch .env

# Edit the .env file and add your API key
echo "ANTHROPIC_API_KEY=your_api_key_here" > .env

# Return to the project root
cd ..
```

Replace `your_api_key_here` with the actual API key you obtained from the Anthropic console.

## Running the Project

You'll need two terminal windows - one for the server and one for the client.

### Terminal 1: Run the Weather Server

```bash
# Make sure you're in the project root with the virtual environment activated
cd mcp-official-tutorial
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Run the weather server
python weather-server-python/weather.py
```

The server will start and wait for client connections.

### Terminal 2: Run the MCP Client

```bash
# Make sure you're in the project root with the virtual environment activated
cd mcp-official-tutorial
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Run the client, pointing to the weather server script
python mcp-client-python/client.py weather-server-python/weather.py
```

You should see a message like:

```
Connected to server with tools: ['get_alerts', 'get_forecast']
MCP Client Started!
Type your queries or 'quit' to exit.

Query: 
```

## Example Queries

Once the client is running, you can try these example queries:

1. **Get weather forecast**:
   ```
   What's the weather like in San Francisco? The coordinates are 37.7749, -122.4194.
   ```

2. **Check for weather alerts**:
   ```
   What's the forecast for latitude 34.0522, longitude -118.2437?
   ```

Claude will process your query, determine when to use the weather tools, and return the results.

## How It Works

1. The client sends your query to Claude along with information about available tools
2. Claude decides whether to respond directly or use a tool
3. If Claude chooses to use a tool, the client executes the tool call
4. The tool result is sent back to Claude for further processing
5. The final response is displayed to you

## Troubleshooting

### API Key Issues

If you see an error about insufficient credit balance:
```
Error: Error code: 400 - {'type': 'error', 'error': {'type': 'invalid_request_error', 'message': 'Your credit balance is too low to access the Anthropic API. Please go to Plans & Billing to upgrade or purchase credits.'}}
```

Make sure you've:
1. Added a payment method in the Anthropic console
2. Added credits to your account
3. Waited a few minutes for the credits to be applied

### Other Common Issues

- **ImportError**: Make sure you've installed all dependencies correctly
- **Connection Issues**: Ensure both the server and client are running
- **Python Version**: Confirm you're using Python 3.10+

## Additional Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)
- [Anthropic API Documentation](https://docs.anthropic.com/)
- [MCP Quickstart Tutorial](https://modelcontextprotocol.io/quickstart)
- [Building MCP Clients Tutorial](https://modelcontextprotocol.io/tutorials/building-a-client)
