---
author: Yu Jia Cheong
categories: ["python", "rust", "bacnet"]
date: '2026-05-01T00:00:00Z'
tags:
- python
- rust
- bacnet
title: Writing a Sans-IO Python library for BACnet in Rust
---

With the advent of AI tools for coding, random side projects that used to live in my head finally see the light of day.

I've primarily been playing around with [opencode](https://opencode.ai/) for my personal use, an AI coding agent which by default has both a planning and building mode.  

# Rust and Python
For a while, I've been meaning to understand how to publish libraries for Python written in Rust. The significant speed-up provided by compute heavy libraries written in Rust such as pydantic, ruff, and uv has significantly changed the landscape of Python tools.

In a simple set up, both Python and Rust source code live in the same repository, in two separate folders. Rust is configured with `Cargo.toml` and Python with `pyproject.toml`.

Configuring the `maturin` build backend in `pyproject.toml`, and using `pyo3` to provide bindings in the Rust code, compiling the Rust code and installing it into an existing virtual environment requires using just this commmand:
```bash
maturin develop
```

# BACnet
Enter... a library for BACnet.

BACnet is an industrial protocol used primarily for communication with HVAC systems.

The current library widely used for BACnet communication is bacpypes and bacpypes3, and it is by far the most complete. However, I find the client initialisation clunky for use. 

# Implementation
I wanted the following design:
* encoding-decoding portion of the library to be written in Rust
* Rust portion should be Sans-IO
* a light async wrapper in Python
* use an async context manager for the client

This was designed and implemented with the help of [opencode](https://opencode.ai/).  

The result handles only BACnet/IP and can be found [here](https://github.com/yujia21/libbacnet/).

A simple example of use is as follows:
```python
import asyncio
from libbacnet import BacnetClient, BacnetAddr, ObjectIdentifier, PropertyIdentifier

async def main():
    async with BacnetClient(local_addr=("0.0.0.0", 47808)) as client:
        # Discover devices (3-second collection window)
        devices = await client.who_is(wait=3.0)
        print(f"Found {len(devices)} device(s)")

        for ev in devices:
            addr = BacnetAddr(ev.src.addr, ev.src.port)
            obj = ObjectIdentifier(object_type=8, instance=ev.message.device_id.instance)

            # Read the device object's description — returns a ReadPropertyResult
            result = await client.read_property(
                addr=addr,
                obj_id=obj,
                prop_id=PropertyIdentifier.DESCRIPTION,
            )
            print(f"  {addr} description: {result.value}")

asyncio.run(main())
```

