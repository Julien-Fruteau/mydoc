---
title: Throttle api calls
tags:
  - python
  - api
  - api call
  - api rate limit
  - asyncio
  - asyncio throttle
---

## API Request Limit


Usage example of the ```asyncio_throttle``` module that can be useful to overcome api rate limit

> this example also display a good way to use async http requests by providing asyncio a *task* list

```py
import asyncio
import time
from asyncio_throttle.throttler import Throttler


async def wait_s(i: int, throttler: Throttler):
    async with throttler:
        await asyncio.sleep(1) # (1)
        print(f"{time.time()} : {i} done")

async def tasks(throttler: Throttler):
    t = [asyncio.create_task(wait_s(i, throttler)) for i in range(200)]
    await asyncio.wait(t, timeout=None, return_when=asyncio.ALL_COMPLETED)

async def main():
    # for a rate limit of 5 requests / sec :
    t = Throttler(rate_limit=5, period=1)
    await tasks(t)


if __name__ == "__main__":
    asyncio.run(main()) 
```

1. :man_raising_hand: make an http request here instead ;)
