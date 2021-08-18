!!! Summary

    :white_check_mark: Use the Loguru [loguru] library if possible

    :white_check_mark: Use the standard python [logging] library otherwise

    :white_check_mark: Expose single logger accessible at package level.
    
    :white_check_mark: Expose logging configuration method.

    :x: Avoid multiple loggers.

    :x: Avoid DO NOT LOG PASSWORDS OR ANY OTHER SENSITIVE INFORMATION.
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

[loguru]: https://github.com/Delgan/loguru
[logging]: https://docs.python.org/3/library/logging.html
