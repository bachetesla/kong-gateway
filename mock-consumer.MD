---

### `kong-consumer.md`

```markdown
# Mock Consumer for Kong Gateway

This guide adds a mock consumer to your existing Kong Gateway setup for testing purposes. It’s designed to work with your `docker-compose.yml` and `kong.yml`, focusing on the `key-auth` plugin.

## What’s a Mock Consumer?

A mock consumer is a test entity in Kong with an API key, used to simulate a client accessing secured services like `admin-api`, `billing-service`, or `notifications-service`.

## Step 1: Update Your `kong.yml`

Add this `consumers` section to your existing `declarative/kong.yml`:

```yaml
consumers:
  - username: mock-consumer
    custom_id: mock-123
    keyauth_credentials:
      - key: mock-key-xyz123
```

### Example Full `kong.yml` (Just Add the Consumers Part)
Your file might look like this after adding it:

```yaml
_format_version: "3.0"
services:
  - name: admin-api
    url: http://localhost:8002
    host: localhost
    routes:
      - name: admin-api
        paths:
          - /admin-api
    plugins:
      - name: key-auth
  # ... your other services ...

consumers:
  - username: mock-consumer
    custom_id: mock-123
    keyauth_credentials:
      - key: mock-key-xyz123

plugins:
  - name: prometheus
  - name: file-log
    config:
      path: /logs/kong.log
```

- Save it in `declarative/kong.yml`.

## Step 2: Apply the Changes

With your Docker Compose already running (`docker-compose up -d`), apply the updated config:

1. **Restart Kong to Load the New Config**:
   ```bash
   docker-compose restart kong-gateway
   ```

2. **Verify Logs**:
   ```bash
   docker-compose logs kong-gateway
   ```
    - Look for "Kong started" and no errors about `kong.yml`.

## Step 3: Test the Mock Consumer

Test your `key-auth` services with the mock consumer’s API key (`mock-key-xyz123`):

### Admin API
- **Without Key** (Should Fail):
  ```bash
  curl http://localhost:8000/admin-api
  ```
    - Output: `{"message":"Unauthorized"}`
- **With Key** (Should Succeed):
  ```bash
  curl http://localhost:8000/admin-api -H "apikey: mock-key-xyz123"
  ```
    - Output: Response from `http://localhost:8002` (or error if no service runs there).

### Billing Service
- **Without Key**:
  ```bash
  curl http://localhost:8000/billing
  ```
    - Output: `{"message":"Unauthorized"}`
- **With Key**:
  ```bash
  curl http://localhost:8000/billing -H "apikey: mock-key-xyz123"
  ```
    - Output: Response from `http://nginx2` (per your `kong.yml`).

### Notifications Service
- **Without Key**:
  ```bash
  curl http://localhost:8000/notifications
  ```
    - Output: `{"message":"Unauthorized"}`
- **With Key**:
  ```bash
  curl http://localhost:8000/notifications -H "apikey: mock-key-xyz123"
  ```
    - Output: Response from `http://nginx1`.

## Notes

- **Prerequisites**: Assumes your `docker-compose.yml` and `.env` are set up as in your main project.
- **Services**: Your `kong.yml` uses `nginx1` and `nginx2` as mock backends for `billing` and `notifications`. Ensure they’re running.
- **Troubleshooting**:
    - 401 persists? Check the key matches `mock-key-xyz123`.
    - No response? Verify service URLs in `kong.yml` match your Docker network (e.g., `http://nginx1`).

This mock consumer lets you test your secured endpoints easily!
```

---

### Copy-Paste Instructions

1. **Add to `kong.yml`**:
   - Open `declarative/kong.yml` and append this at the end:
     ```yaml
     consumers:
       - username: mock-consumer
         custom_id: mock-123
         keyauth_credentials:
           - key: mock-key-xyz123
     ```

2. **Run Commands**:
   ```bash
   # Restart Kong to apply the config
   docker-compose restart kong-gateway

   # Check logs
   docker-compose logs kong-gateway

   # Test Admin API
   curl http://localhost:8000/admin-api  # Should fail
   curl http://localhost:8000/admin-api -H "apikey: mock-key-xyz123"  # Should work

   # Test Billing Service
   curl http://localhost:8000/billing  # Should fail
   curl http://localhost:8000/billing -H "apikey: mock-key-xyz123"  # Should work

   # Test Notifications Service
   curl http://localhost:8000/notifications  # Should fail
   curl http://localhost:8000/notifications -H "apikey: mock-key-xyz123"  # Should work
   ```

---

### Notes
- **Scope**: This focuses only on the mock consumer, using your existing setup.
- **Repo**: Add this as `kong-consumer.md` to `https://github.com/bachetesla/openlens`.
- **Customization**: Change `mock-key-xyz123` to any key you prefer—just update it in `kong.yml` and your tests.

