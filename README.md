# Apollo Federation Subgraph Compatibility Testing

[![NPM Version](https://img.shields.io/npm/v/@apollo/federation-subgraph-compatibility)](https://www.npmjs.com/package/@apollo/federation-subgraph-compatibility)
[![Continuous Integration](https://github.com/apollographql/apollo-federation-subgraph-compatibility/workflows/Continuous%20Integration/badge.svg)](https://github.com/apollographql/apollo-federation-subgraph-compatibility/actions?query=workflow%3A"Continuous+Integration")
[![Join the community forum](https://img.shields.io/badge/Join%20The%20Community-Forum-blueviolet)](https://community.apollographql.com)
[![MIT License](https://img.shields.io/github/license/apollographql/apollo-federation-subgraph-compatibility)](https://github.com/apollographql/apollo-federation-subgraph-compatibility/blob/main/LICENSE)

`@apollo/federation-subgraph-compatibility` script is an NPM package that allows you to test given subgraph's implementation for compatibility against the [Apollo Federation Subgraph Specification](https://www.apollographql.com/docs/federation/subgraph-spec/). This testing suite verifies various Federation features against subgraph implementation. See [compatibility testing docs](./COMPATIBILITY.md) for details on the expected schema and the data sets as well as information about the executed tests.

This repository contains number of [example subgraph implementations](https://github.com/apollographql/apollo-federation-subgraph-compatibility/tree/main/implementations) based on various libraries and other solutions. See [latest compatibility results](https://www.apollographql.com/docs/federation/building-supergraphs/supported-subgraphs) for a list of Apollo Federation compatibible subgraph implementations.

- [Apollo Federation Subgraph Compatibility Testing](#apollo-federation-subgraph-compatibility-testing)
  - [Installation](#installation)
  - [Usage](#usage)
    - [PM2](#pm2)
    - [Docker Compose](#docker-compose)
    - [Test Results](#test-results)
    - [Debug Mode](#debug-mode)
  - [Contact](#contact)
  - [Contributing](#contributing)
    - [Contributing A New Implementation To The Compatibility Test Suite](#contributing-a-new-implementation-to-the-compatibility-test-suite)
  - [Security](#security)
  - [License](#license)

## Installation

Apollo Federation subgraph compatibility test script is published to NPM.

```shell
npm install --dev @apollo/federation-subgraph-compatibility
```

You can execute the script using [NPX](https://docs.npmjs.com/cli/v7/commands/npx)

```shell
# remote package
npx @apollo/federation-subgraph-compatibility --help

# package installed locally
npx fedtest --help

# package installed globally
fedtest --help
```

## Usage

`@apollo/federation-subgraph-compatibility` test script starts a supergraph that consists of your subgraph implementation, two reference subgraphs and [Apollo Router](https://github.com/apollographql/router). Since script needs to manage multiple processes, you need to specify which process management technology should be used to run the tests. Currently script supports [PM2](https://pm2.keymetrics.io/) and [Docker Compose](https://docs.docker.com/compose/).

```shell
npx fedtest --help
Usage: fedtest [options] [command]

Run Apollo Federation subgraph compatibility tests

Options:
  -h, --help                display help for command

Commands:
  pm2 [options]             Start supergraph using PM2
  docker [options]          Start supergraph using Docker Compose
  help [options] [command]  display help for command
```

### PM2

[PM2](https://pm2.keymetrics.io/) is a Node based process manager. When tests are run using `pm2` command, script will start the supergraph using [Apollo Rover](https://github.com/apollographql/rover) `dev` command.

```shell
npx fedtest pm2 --help
Usage: fedtest pm2 [options]

Start supergraph using PM2

Options:
  --config <subgraph.config.js>  optional PM2 configuration file
  --debug                        debug mode with extra log info
  --endpoint <url>               subgraph endpoint
  --format <json|markdown>       optional output file format (choices: "json", "markdown", default: "markdown")
  -h, --help                     display help for command
  --schema <schema.graphql>      optional path to schema file, if omitted composition will fallback to introspection
```

If your subgraph is already up and running, you can run compatibility tests using PM2 by specifying `--endpoint <url>` option. This will create supergraph that uses introspection to read subgraph schema.

```shell
npx fedtest pm2 --endpoint http://localhost:8080/graphql
```

If you have a subgraph schema available, you can also specify it on the command line. Schema path can be absolute or relative to the current directory.

```shell
npx fedtest pm2 --endpoint http://localhost:8080/graphql --schema /path/to/schema.graphql
```

PM2 can also be used to start your subgraph. You can specify an [ecosystem config file](https://pm2.keymetrics.io/docs/usage/application-declaration/) that contains information how to start your subgraph.

```js
// simple PM2 subgraph config that starts Node application
module.exports = {
    apps : [{
      name   : "subgraph product",
      script : "index.js",
      cwd: "/path/to/your/implementation"
    }]
}
```

```shell
npx fedtest pm2 --endpoint http://localhost:8080/graphql --config /path/to/subgraph.config.js
```

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is a tool for running multi-container Docker applications. When tests are run using `docker` command, script will compose supergraph using [Apollo Rover](https://github.com/apollographql/rover) `supergraph compose` command and run standalone router image.

```shell
npx fedtest docker --help
Usage: fedtest docker [options]

Start supergraph using Docker Compose

Options:
  --compose <docker-compose.yaml>  Path to docker compose file
  --debug                          debug mode with extra log info
  --format <json|markdown>         optional output file format (choices: "json", "markdown", default: "markdown")
  -h, --help                       display help for command
  --path <path>                    GraphQL endpoint path (default: "")
  --port <port>                    HTTP server port (default: "4001")
  --schema <schema.graphql>        Path to schema file
```

In order to run compatibility tests using `docker` command, you need to pass both compose file and subgraph schema file.
Paths to files can be absolute or relative to the current working directory.

```shell
npx fedtest docker --compose /path/to/docker-compose.yaml --schema /path/to/schema.graphql
```

When creating supergraph using Docker, it will create network with default subgraph endpoint as `http://localhost:4001`. You can override the default endpoint by specifying custom GraphQL endpoint path and port.

```shell
# docker compose file starts subgraph exposing http://localhost:8080/graphql
npx fedtest docker --compose /path/to/docker-compose.yaml --schema /path/to/schema.graphql --path /graphql --port 8080
```

### Test Results

Script logs the compatibility test results on the console and also generates corresponding results file. By default, script will generate results in a markdown format. You can change this behavior by specifying and `--format json` option.

```shell
# generate results.json file
npx fedtest pm2 --endpoint http://localhost:8080/graphql --format json
```

### Debug Mode

In order to enable debug mode that provides more verbose logs, specify `--debug` option on the command line.

```shell
npx fedtest pm2 --endpoint http://localhost:8080/graphql --debug
```

## Contact

If you have a specific question about the testing library or code, please start a discussion in the [Apollo community forums](https://community.apollographql.com/).

## Contributing

### Contributing A New Implementation To The Compatibility Test Suite

Fork this repository and navigate to the [Apollo Federation Subgraph Maintainers Implementation Guide](./CONTRIBUTORS.md) for implementation instructions. Once you've completed the implementations instructions, feel free to create a PR and we'll review it. If you have any questions please open a GitHub issue on this repository.

## Security

For more info on how to contact the team for security issues, see our [Security Policy](https://github.com/apollographql/.github/blob/main/SECURITY.md).

## License

This library is licensed under [The MIT License (MIT)](https://github.com/apollographql/apollo-federation-subgraph-compatibility/blob/main/LICENSE).
