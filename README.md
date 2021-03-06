**This tool has now been deprecated. Thank you to everyone who contributed and made it so useful while it was in use. :blue_heart:**

---

# Exercism Local Tooling Webserver

This small webhook wrapper is used when tooling (analyzers/representers/test runners) runs inside the Docker development environment. Since they cannot be launched via `runc` in this environment we need an alternative. The [Tooling Invoker](https://github.com/exercism/tooling-invoker/) instead dispatches a web request. This tiny web server responds to those requests, wraps the underlying `./run.sh` test runner script, and returns the JSON output file as a simple JSON response.

## Installation (Docker)

The compiled stand-alone binary should be added directly into your tooling's Docker image. Here are some Dockerfile commands to use `curl` or `wget` to include the binary into your Docker image:

```dockerfile
# inside your Dockerfile
RUN curl -L -o /usr/local/bin/tooling_webserver \
      https://github.com/exercism/tooling-webserver/releases/download/latest/tooling_webserver && \
    chmod +x /usr/local/bin/tooling_webserver
```

or

```dockerfile
# inside your Dockerfile
RUN wget -P /usr/local/bin https://github.com/exercism/tooling-webserver/releases/latest/download/tooling_webserver && \
    chmod +x /usr/local/bin/tooling_webserver
```

As an alternative, you could also include a specific version of the tooling webserver in your Dockerfile. The main advantage of this is that updating the version will trigger a rebuild of the Docker image, where this might not be the case with the latest version (depending on how you've setup your Dockerfile).

```dockerfile
RUN wget -P /usr/local/bin https://github.com/exercism/tooling-webserver/releases/download/0.10.0/tooling_webserver && \
    chmod +x /usr/local/bin/tooling_webserver
```

See [this Dockerfile](https://github.com/exercism/javascript-test-runner/blob/master/Dockerfile#L33) for an example on how to include the tooling into a Docker image.

The above allows the [development environment](https://github.com/exercism/development-environment/) to run the tooling as a Docker container but with a modified entrypoint:

```yaml
javascript-test-runner:
  entrypoint: tooling_webserver
```

## Testing with Curl

An example:

```bash
curl 'http://localhost:4567/job' -H 'Expect:' \
  -d exercise=two-fer \
  -d results_filepath=results.json \
  --data-urlencode zipped_files@test.zip
```

The zip archive should include the exercise solution and tests. For this to work you'll have to expose port 4567. If you're using the [development-environment](https://github.com/exercism/development-environment/), you can simply modify your `stack.yml`:

```yaml
configure:
  javascript-test-runner:
    build: true
    ports:
      - 4567:4567
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/exercism/tooling-webserver.

