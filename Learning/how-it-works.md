# Understanding the Weather Forecast Query System

This document explains in detail how our system processes and answers weather forecast queries, from user input to final response.

## System Architecture

![MCP Weather System Architecture](mcp-diagram.svg)

The system consists of four main components that work together:

1. **User Interface**: Where you enter your weather queries
2. **MCP Client**: Manages communication between you, Claude, and the available tools
3. **Claude AI**: Processes natural language and decides when to use tools
4. **Weather MCP Server**: Provides weather-specific tools to Claude
5. **National Weather Service API**: External data source that provides actual weather information

## Detailed Request Flow

When you ask a question about the weather forecast, here's exactly what happens:

### 1. Query Input

You type a natural language query like "What's the weather in San Francisco?" into the client interface.

### 2. Client Processes Query

The Python MCP client (`mcp-client-python/client.py`) takes your query and prepares to process it:

```python
async def process_query(self, query: str) -> str:
    """Process a query using Claude and available tools"""
    messages = [
        {
            "role": "user",
            "content": query
        }
    ]
    # ... rest of the function
```

### 3. Available Tools Discovery

The client asks the MCP server what tools are available:

```python
response = await self.session.list_tools()
available_tools = [{ 
    "name": tool.name,
    "description": tool.description,
    "input_schema": tool.inputSchema
} for tool in response.tools]
```

For the weather server, it discovers two tools:
- `get_forecast`: Gets weather forecast for a location (requires latitude and longitude)
- `get_alerts`: Gets weather alerts for a US state (requires state code)

### 4. Initial Claude API Call

The client sends your question to Claude along with information about the available tools:

```python
response = self.anthropic.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1000,
    messages=messages,
    tools=available_tools
)
```

### 5. Claude's Decision Process

Claude processes your query using its natural language understanding abilities. It decides whether:
- It can answer directly (for general weather information)
- It should use an available tool (for specific location forecasts)

Claude examines the question to determine if there's a specific location mentioned and whether current weather data is needed. If so, it will decide to use the appropriate tool.

### 6. Tool Execution (if needed)

If Claude decides to use a tool, the following happens:

a) Claude generates a tool call with the proper parameters. For a forecast request:
```python
tool_name = "get_forecast"
tool_args = {"latitude": 37.7749, "longitude": -122.4194}  # San Francisco coordinates
```

b) The client executes the tool call by sending it to the server:
```python
result = await self.session.call_tool(tool_name, tool_args)
```

c) The Weather server (`weather-server-python/weather.py`) processes the tool call:
```python
@mcp.tool()
async def get_forecast(latitude: float, longitude: float) -> str:
    """Get weather forecast for a location.

    Args:
        latitude: Latitude of the location
        longitude: Longitude of the location
    """
    # First get the forecast grid endpoint
    points_url = f"{NWS_API_BASE}/points/{latitude},{longitude}"
    points_data = await make_nws_request(points_url)

    # Get the forecast URL from the points response
    forecast_url = points_data["properties"]["forecast"]
    forecast_data = await make_nws_request(forecast_url)

    # Format the periods into a readable forecast
    periods = forecast_data["properties"]["periods"]
    forecasts = []
    for period in periods[:5]:  # Only show next 5 periods
        forecast = f"""
{period['name']}:
Temperature: {period['temperature']}°{period['temperatureUnit']}
Wind: {period['windSpeed']} {period['windDirection']}
Forecast: {period['detailedForecast']}
"""
        forecasts.append(forecast)

    return "\n---\n".join(forecasts)
```

d) The Weather server makes HTTP requests to the National Weather Service API:
- First to get grid points for the location
- Then to get the actual forecast data

e) The server formats the results and returns them to the client

### 7. Continued Conversation

a) The client adds the tool result to the conversation history:
```python
messages.append({
    "role": "user", 
    "content": result.content
})
```

b) The client sends the updated conversation back to Claude:
```python
response = self.anthropic.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1000,
    messages=messages,
)
```

c) Claude processes the tool results and generates a final, human-friendly response that interprets the weather data

### 8. Final Output

The client combines all the responses and presents the final answer to you:
```python
final_text.append(response.content[0].text)
return "\n".join(final_text)
```

## Technical Implementation Details

### MCP Client (Python)

The client manages the overall communication flow:

```python
class MCPClient:
    def __init__(self):
        # Initialize session and client objects
        self.session: Optional[ClientSession] = None
        self.exit_stack = AsyncExitStack()
        self.anthropic = Anthropic()

    async def connect_to_server(self, server_script_path: str):
        """Connect to an MCP server"""
        # Set up connection to server
        
    async def process_query(self, query: str) -> str:
        """Process a query using Claude and available tools"""
        # Handle the query and tool usage

    async def chat_loop(self):
        """Run an interactive chat loop"""
        # Interactive command-line interface
```

### Weather MCP Server 

The server implements specific tools using the FastMCP framework:

```python
# Initialize FastMCP server
mcp = FastMCP("weather")

@mcp.tool()
async def get_alerts(state: str) -> str:
    """Get weather alerts for a US state.

    Args:
        state: Two-letter US state code (e.g. CA, NY)
    """
    # Implementation

@mcp.tool()
async def get_forecast(latitude: float, longitude: float) -> str:
    """Get weather forecast for a location.

    Args:
        latitude: Latitude of the location
        longitude: Longitude of the location
    """
    # Implementation
```

## Key Technical Components

1. **Model Context Protocol (MCP)**: A standardized protocol for communication between AI models and external tools

2. **Anthropic Claude API**: The large language model that understands natural language and can use tools

3. **FastMCP Framework**: A Python implementation of the MCP protocol for creating tool servers

4. **Asynchronous Programming**: Both client and server use Python's async/await pattern for efficient I/O operations

5. **National Weather Service API**: The external data source for weather information

## Why This Architecture Is Powerful

- **Natural Language Interface**: Users can ask questions in natural language without needing to know the specific API parameters
- **Tool Decision Intelligence**: Claude decides when and how to use tools based on the query context
- **Separation of Concerns**: Each component has a specific role, making the system modular and maintainable
- **Extensibility**: New tools can be added to the server without changing the client or Claude
- **Error Handling**: The system includes error handling at multiple levels

This architecture bridges the gap between AI language models and real-world data sources, allowing for a seamless, conversation-like interface to obtain specific, up-to-date information.

## Example Usage

```
Query: What's the weather like in San Francisco? The coordinates are 37.7749, -122.4194.

[Claude processes the query and recognizes it needs current weather data]
[Calling tool get_forecast with args {"latitude": 37.7749, "longitude": -122.4194}]

Based on the National Weather Service forecast, here's the weather for San Francisco:

Tonight:
Temperature: 54°F
Wind: 5 to 10 mph West
Forecast: Partly cloudy, with a low around 54. West wind 5 to 10 mph.

Thursday:
Temperature: 68°F
Wind: 5 to 10 mph West
Forecast: Sunny, with a high near 68. West wind 5 to 10 mph.

[Additional forecast periods...]
```

## Further Resources

For more information about the components used in this system:

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)
- [Anthropic Claude API Documentation](https://docs.anthropic.com/)
- [National Weather Service API Documentation](https://www.weather.gov/documentation/services-web-api)