## Logging Update for Multi-Worker Setup (`DVUtils.py`)

We’ve updated the logging mechanism to ensure **each worker process** in a multi-worker environment has its **own dedicated logger instance**. This prevents issues like shared state, overlapping log messages, or logger conflicts.

---

### Service-specific changes:

Previously, services used a single shared logger, which can break when multiple workers are running with the same logger instance writing to the same file.

We now use:
- **Lazy initialization** of loggers (only created when needed)
- **One logger per process (PID)** using `os.getpid()`

---

### How to Implement in Your Service

#### 1. Add this function to `DVUtils.py` for creating logger instances based on the worker process ID.
```
logger_instances = {}

def get_logger():
    pid = str(os.getpid())
    # If the process id is not in the cache, create a new logger instance
    if pid not in logger_instances:
        logger_instances[pid] = dv_logger.get_dv_logger(
            env=DVConstants.ENV,
            service_name=DVConstants.SERVICE_NAME,
            file_name=DVConstants.LOG_FILE_NAME,
            slack_notify=True,
            channel_name=DVConstants.SLACK_CHANNEL,
            developers="<USER>",
        )
    return logger_instances[pid]
```
#### 2. Replace the usual log instance with Lazy initialization in `DVUtils.py`
```
class _LoggerProxy:
    def __getattr__(self, name):
        return getattr(get_logger(), name)


log = _LoggerProxy()
```

