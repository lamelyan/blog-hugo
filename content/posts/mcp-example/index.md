+++
title = "Understanding the Model Context Protocol (MCP)"
date = 2025-05-21
featured_image = 'mcp-explained.jpg'
tags = ["mcp", "ai"]
+++

Recently, there’s been a lot of talk about the Model Context Protocol. I started looking into some practical examples
of how it’s used and came across a video that really helped clarify things for me. 
It was a simple example, but it opened up my understanding of how to apply the protocol effectively.

Hopefully, it’s useful for you as well. I’m looking forward to implementing something similar myself. 
It should also help deepen my understanding of how to use these tools more practically—especially for 
projects that can't be exposed to the internet but still want to take advantage of LLMs.

{{< youtube sahuZMMXNpI >}}


This video explains the **Model Context Protocol (MCP)** and how it gives Large Language Models (LLMs) access to specific data. Here's a quick rundown:

### Problem
LLMs like Claude can't directly access data sources like GitHub repositories.  
[Watch from 0:34](https://www.youtube.com/watch?v=sahuZMMXNpI&t=34s)

### Solution
MCP allows Claude to access data through an MCP server.  
[Watch from 1:01](https://www.youtube.com/watch?v=sahuZMMXNpI&t=61s)

### Key Components
The video identifies:
- The **client** (e.g., Claude Desktop)
- The **LLM** (Claude)
- The **MCP server**, which can access data sources like GitHub  
  [Watch from 1:55](https://www.youtube.com/watch?v=sahuZMMXNpI&t=115s)

### How It Works
The MCP server:
- Exposes available tools (e.g., listing commits) via an endpoint
- The client asks for available tools
- Sends a query to the LLM
- The LLM requests a tool
- The client uses the tool through the MCP server
- The LLM uses the resulting data to answer the query  
  [Watch from 6:42](https://www.youtube.com/watch?v=sahuZMMXNpI&t=402s)

### Configuration
Configure Claude Desktop to use an MCP server using a config file and Docker commands.  
[Watch from 9:24](https://www.youtube.com/watch?v=sahuZMMXNpI&t=564s)

### Additional Points
- MCP servers can do more than just tools (e.g., sample routes, template prompts)  
  [Watch from 11:44](https://www.youtube.com/watch?v=sahuZMMXNpI&t=704s)

- The next video will explain how to implement a client  
  [Watch from 12:06](https://www.youtube.com/watch?v=sahuZMMXNpI&t=726s)
