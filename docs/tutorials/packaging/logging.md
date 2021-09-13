!!! Summary

    :white_check_mark: Use the Loguru [loguru] library if possible.

    :white_check_mark: Use the standard python [logging] library otherwise.

    :white_check_mark: Expose single logger accessible at package level.
    
    :white_check_mark: Expose logging configuration method.

    :x: Avoid multiple loggers.

    :x: Avoid using `print`.

    :x: Avoid LOG ANY SENSITIVE INFORMATION.
# Logging

> With good program architecture debugging is a breeze,
>   because bugs will be where they should be.
> 
> -- <cite>David May</cite>


- use a single logger.
- use `lazy` logging option if supported instead of checking if logger is enabled for a certain level. 
- consistent naming. The recommended name for your logger is `logger`.
- Use the Correct Levels When Logging.
- Include a Timestamp for each log entry.
- Adopt the ISO-8601 Format for Timestamps.
- DO NOT create new methods for handling logs. 


### Use the correct levels when logging

- DEBUG: You should use this level for debugging purposes in development.
- INFO: You should use this level when something interesting—but expected—happens (e.g., a user sends a new document of certain type to our application).
- WARNING: You should use this level when something unexpected or unusual happens. It’s not an error, but you should pay attention to it.
- ERROR: This level is for things that go wrong but are usually recoverable (e.g., internal exceptions you can handle or APIs returning error results).
- CRITICAL: You should use this level in a doomsday scenario. The application is unusable. At this level, someone should be woken up at 2 a.m.


### DON'T REINVNET THE WHEEL

`Print` statement is so easy but it comes with a price. 

You rarely need to subclass the logging module.  Most of the time, you can achieve what you need through `Structured logging` or creating a new `Record` for the logger or create a new `Handler` to handle the record. 


### When to log

A rule of thumb when it comes to wehn to log is to think of logs as a story.  If you are trying to tell a story, you should have a beginning, middle and end section.  

- **Begining** of an operation. (e.g. Preparing connection to a service)
- **Middle** of an operation. (e.g. update relevant progress on download/upload files, etc.)
- **End** of an operation. (e.g. conclude an operation is either succeeded or failed.)


### What to log

Logs are stories, what to log usually boils down to one of the two themes (Auditing or Diagnostic purpose). 
>
> `If I read this log, I know what is going on internally with the system?` 
> 

or 
>
>`If I read this log, I know what I need to do to next.`

A word of advice is knowning what you are logging and not log anything senstive.  **Assuming what you log is a public record all the time.** 


### Loguru common configuration parameters

- **sink：** You can pass in a file object （file-like object）, Or a str String or pathlib.Path object , Or a way （coroutine function）, or logging Modular Handler（logging.Handler）.
- **level (int or str, optional) ：** The lowest severity level that recorded messages should be sent to the receiver .
- **format (str or callable, optional) ：** Format module , Before sending it to the receiver , Use templates to format recorded messages .
- **filter (callable, str or dict, optional) ：** Used to determine whether each recorded message should be sent to the receiver .
- **colorize (bool, optional) :** – Whether the color tags contained in the formatted message should be converted to Ansi Code , Or otherwise . If None, According to whether the sink is TTY Make a choice automatically .
- **serialize (bool, optional) ：** Before sending it to the receiver , Whether the recorded message and its record should first be converted to JSON character string .
- **backtrace (bool, optional) ：** Whether the formatted exception trace should be extended up , Beyond capture point , To display the full stack trace of the build error .
- **diagnose (bool, optional) ：** Whether exception tracking should display variable values to simplify debugging . In production , This should be set to “False”, To avoid leaking sensitive data .
- **enqueue (bool, optional) ：** Whether the message to be recorded should pass through the multiprocess secure queue before reaching the receiver . When logging to a file through multiple processes , This is very useful . This also has the advantage of making log calls non blocking .
- **catch (bool, optional) ：** Whether the errors that occur when the receiver processes log messages should be automatically captured . If True The exception message is displayed on the sys.stderr. however , The exception does not propagate to the caller , To prevent the application from crashing .


### Loguru formatting keys
formatting template properties as follows ：

| Key         |  Description        |
| ------------| ------------------- |
| `elapsed` | The time difference from the beginning of the program   |
| `exception` | Formatting exception ( If there is ), Otherwise ' None '   |
| `extra` | User bound property Dictionary ( See bind())   |
| `file` | The file that makes the logging call   |
| `function` | Functions for logging calls   |
| `level` | Used to record the severity of the message   |
| `line` | Line number in source code   |
| `message` | Recorded messages ( It's not formatted yet )   |
| `module` | Used to record the severity of the message   |
| `name` | Logging calls __name__  |
| `process` | The name of the process making the logging call  |
| `thread` | The name of the thread making the logging call  |
| `time` | The perceived local time when a log call is made  |


### Recommendated Loguru formatting string

```python
from loguru import logger, _defaults as loguru_defaults

def log_format(record):
    if 'tid' in record["extra"]:
        return (
                "[{time:YYYY-MM-DD HH:mm:ss.SSS}] | "
                "[{extra[tid]}] | "
                "[{level: <8}] | " 
                "{process} - {thread} | "
                "{name}:{function: <15}:{line} | "
                "- {message} | "
                "{extra}"
            )
    else:
        return loguru_defaults.LOGURU_FORMAT+'\n'

logger.add(sys.stdout, format=log_format)

config = {
    "handlers": [
        {
            "sink": sys.stderr, 
            "format": log_format,
            "backtrace": False,
            "diagnose": True,
            "encoding": "utf-8",
            'level': 'DEBUG',
        },
    ],
     "extra": {
         "version": "GITHASH"
     },
}
logger.configure(**config)

with logger.contextualize(tid=current_tid):
    logger.info(...)
```









[loguru]: https://github.com/Delgan/loguru
[logging]: https://docs.python.org/3/library/logging.html
