---
description: A list of any actions you may need to take on upgrades of Directus.
---

# Breaking Changes

As we continue to build Directus, we occasionally make changes that change how certain features works. We try and keep
these to a minimum, but rest assured we only make them with good reason.

[Learn more about Versioning ->](/getting-started/architecture#versioning)

Starting with Directus 10.0, here is a list of potential breaking changes with remedial action you may need to take.

## Version 10.4

### Consolidated Environment Variables for Redis Use

Directus had various different functionalities that required you to use Redis when running Directus in a horizontally
scaled environment such as caching, rate-limiting, realtime, and flows. The configuration for these different parts have
been combined into a single set of `REDIS` environment variables that are reused across the system.

::: details Migration/Mitigation

Combine all the `*_REDIS` environment variables into a single shared one as followed:

::: code-group

```ini [Before]
CACHE_STORE="redis"
CACHE_REDIS_HOST="127.0.0.1"
CACHE_REDIS_PORT="6379"
...
RATE_LIMITER_STORE="redis"
RATE_LIMITER_REDIS_HOST="127.0.0.1"
RATE_LIMITER_REDIS_PORT="6379"
...
SYNCHRONIZATION_STORE="redis"
SYNCHRONIZATION_REDIS_HOST="127.0.0.1"
SYNCHRONIZATION_REDIS_PORT="6379"
...
MESSENGER_STORE="redis"
MESSENGER_REDIS_HOST="127.0.0.1"
MESSENGER_REDIS_PORT="6379"
```

```ini [After]
REDIS_HOST="127.0.0.1"
REDIS_PORT="6379"

CACHE_STORE="redis"
RATE_LIMITER_STORE="redis"
SYNCHRONIZATION_STORE="redis"
MESSENGER_STORE="redis"
```

:::

### Dropped Support for Memcached

Directus used to support either memory, Redis, or Memcached for caching and rate-limiting storage. Given a deeper
integration with Redis, and the low overall usage/adoption of Memcached across Directus installations, we've decided to
sunset Memcached in favor of focusing on Redis as the primary solution for pub/sub and hot-storage across load-balanced
Directus installations.

### Updated Errors Structure for Extensions

As part of standardizing how extensions are built and shipped, you must replace any system exceptions you extracted from
`exceptions` with new errors created within the extension itself. We recommend prefixing the error code with your
extension name for improved debugging, but you can keep using the system codes if you relied on that in the past.

::: details Migration/Mitigation

::: code-group

```js [Before]
export default (router, { exceptions }) => {
	const { ForbiddenException } = exceptions;

	router.get('/', (req, res) => {
		throw new ForbiddenException();
	});
};
```

```js [After]
import { createError } from '@directus/errors';

const ForbiddenError = createError('MY_EXTENSION_FORBIDDEN', 'No script kiddies please...');

export default (router) => {
	router.get('/', (req, res) => {
		throw new ForbiddenError();
	});
};
```

:::

## Version 10.2

### Removed Fields from Server Info Endpoint

As a security precaution, we have removed the following information from the `/server/info` endpoint:

- Directus Version
- Node Version and Uptime
- OS Type, Version, Uptime, and Memory
