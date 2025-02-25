
# Default framework-side logging configuration

Kedro's [default logging configuration](https://github.com/kedro-org/kedro/blob/main/kedro/framework/project/default_logging.yml) defines a handler called `rich` that uses the [Rich logging handler](https://rich.readthedocs.io/en/stable/logging.html) to format messages. We also use the [Rich traceback handler](https://rich.readthedocs.io/en/stable/traceback.html) to render exceptions.

By default, Python only shows logging messages at level `WARNING` and above. Kedro's logging configuration specifies that `INFO` level messages from Kedro should also be emitted. This makes it easier to track the progress of your pipeline when you perform a `kedro run`.

## Project-side logging configuration

In addition to the `rich` handler defined in Kedro's framework, the [project-side `conf/base/logging.yml`](https://github.com/kedro-org/kedro/blob/main/kedro/templates/project/%7B%7B%20cookiecutter.repo_name%20%7D%7D/conf/base/logging.yml) defines two further logging handlers:
* `console`: show logs on standard output (typically your terminal screen) without any rich formatting
* `info_file_handler`: write logs of level `INFO` and above to `info.log`

The logging handlers that are actually used by default are `rich` and `info_file_handler`.

The project-side logging configuration also ensures that [logs emitted from your project's logger](#perform-logging-in-your-project) should be shown if they are `INFO` level or above (as opposed to the Python default of `WARNING`).

We now give some common examples of how you might like to change your project's logging configuration.

### Disable file-based logging

You might sometimes need to disable file-based logging, e.g. if you are running Kedro on a read-only file system such as [Databricks Repos](https://docs.databricks.com/repos/index.html). The simplest way to do this is to delete your `conf/base/logging.yml` file. With no project-side logging configuration specified, Kedro uses the default framework-side logging configuration, which does not include any file-based handlers.

Alternatively, if you would like to keep other configuration in `conf/base/logging.yml` and just disable file-based logging, then you can remove the file-based handlers from the `root` logger as follows:
```diff
 root:
-  handlers: [console, info_file_handler]
+  handlers: [console]
```

### Use plain console logging

To use plain rather than rich logging, swap the `rich` handler for the `console` one as follows:

```diff
 root:
-  handlers: [rich, info_file_handler]
+  handlers: [console, info_file_handler]
```

### Rich logging in a dumb terminal

Rich [detects whether your terminal is capable](https://rich.readthedocs.io/en/stable/console.html#terminal-detection) of displaying richly formatted messages. If your terminal is "dumb" then formatting is automatically stripped out so that the logs are just plain text. This is likely to happen if you perform `kedro run` on CI (e.g. GitHub Actions or CircleCI).

If you find that the default wrapping of the log messages is too narrow but do not wish to switch to using the `console` logger on CI then the simplest way to control the log message wrapping is through altering the `COLUMNS` and `LINES` environment variables. For example:

```bash
export COLUMNS=120 LINES=25
```

```{note}
You must provide a value for both `COLUMNS` and `LINES` even if you only wish to change the width of the log message. Rich's default values for these variables are `COLUMNS=80` and `LINE=25`.
```

### Rich logging in Jupyter

Rich also formats the logs in JupyterLab and Jupyter Notebook. The size of the output console does not adapt to your window but can be controlled through the `JUPYTER_COLUMNS` and `JUPYTER_LINES` environment variables. The default values (115 and 100 respectively) should be suitable for most users, but if you require a different output console size then you should alter the values of `JUPYTER_COLUMNS` and `JUPYTER_LINES`.

## Perform logging in your project

To perform logging in your own code (e.g. in a node), you are advised to do as follows:

```python
import logging

log = logging.getLogger(__name__)
log.warning("Issue warning")
log.info("Send information")
```

```{note}
The name of a logger corresponds to a key in the `loggers`  section in `logging.yml` (e.g. `kedro`). See [Python's logging documentation](https://docs.python.org/3/library/logging.html#logger-objects) for more information.
```

You can take advantage of rich's [console markup](https://rich.readthedocs.io/en/stable/markup.html) when enabled in your logging calls:
```python
log.error("[bold red blink]Important error message![/]", extra={"markup": True})
```
