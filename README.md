# LoggerTelegramBackend

[![Build Status](https://github.com/adriankumpf/logger-telegram-backend/workflows/CI/badge.svg)](https://github.com/adriankumpf/logger-telegram-backend/actions)
[![Docs](https://img.shields.io/badge/hex-docs-green.svg?style=flat)](https://hexdocs.pm/logger_telegram_backend)
[![Hex.pm](https://img.shields.io/hexpm/v/logger_telegram_backend?color=%23714a94)](http://hex.pm/packages/logger_telegram_backend)

<!-- MDOC !-->

A logger backend for [Telegram](https://telegram.org/).

## Installation

Add `:logger_telegram_backend` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:logger_telegram_backend, "~> 2.0"},
    {:hackney, "~> 1.17"}, # optional, but recommended adapter
  ]
end
```

## Configuration

First of all you need to create a [Telegram bot](https://core.telegram.org/bots). Follow the [instructions here](https://core.telegram.org/bots#6-botfather) to create one and get the `token` for the bot. Since bots are not allowed to contact users, you need to send a message first. Afterwards, retrieve your `chat_id` with `$ curl -X GET https://api.telegram.org/botYOUR_TOKEN/getUpdates`.

Then add `LoggerTelegramBackend` to the `:backends` configuration, configure the telegram `chat_id` and bot `token`:

```elixir
config :logger, backends: [LoggerTelegramBackend, :console]

config :logger, :telegram,
  chat_id: "$chatId",
  token: "$botToken"
```

The logger configuration is read at runtime from the application environment so that you can provide it via [distillerys](https://github.com/bitwalker/distillery) dynamic configuration with environment variables.

### Options

The following options are available:

- `:level` - the level to be logged by this backend (either `:debug`,
  `:info`, `:warn` or `:error`). Note that messages are filtered by the general `:level` configuration for the `:logger` application first. If not explicitly configured all levels are logged.
- `:metadata` - the metadata to be included in the telegram message. Defaults to `[:line, :function, :module, :application, :file]`. Setting `:metadata` to `:all` gets all metadata.
- `:metadata_filter` - the metadata which is required in order for a message to be logged. Example: `metadata_filter: [application: :ui]`.
- `:proxy` - connect via an HTTP tunnel or a socks5 proxy. See the [hackney docs](https://github.com/benoitc/hackney#proxy-a-connection) for further information. (Only available with the default hackney adapter).
- `:adapter` - the [Tesla adapter](https://hexdoks.pm/tesla/readme.html) for the Telegram client (default: `{Tesla.Adapter.Hackney, pool: :logger_telegram_backend}`)

#### Examples

##### With Metadata Filter

```elixir
config :logger, :telegram,
  chat_id: "$chatId",
  token: "$botToken",
  level: :info,
  metadata: :all
  metadata_filter: [application: :ui]
```

##### With Finch Adapter

```elixir
config :logger, :telegram,
  chat_id: "$chatId",
  token: "$botToken",
  adapter: {Tesla.Adapter.Finch, name: MyFinch}
```

Note: You'll need to add Finch instead of hackney to your list of dependencies:

```elixir
  {:finch, "~> 0.6"}
```

##### With Proxy

```elixir
config :logger, :telegram,
  chat_id: "$chatId",
  token: "$botToken",
  proxy: "socks5://127.0.0.1:9050"
```

### Multiple logger handlers

Like the [LoggerFileBackend](https://github.com/onkel-dirtus/logger_file_backend) multiple logger handlers may be configured, each with different `:chat_id`s, `:level`s etc. Each handler has to be configured as a separate logger backend:

```elixir
config :logger,
  backends: [
    {LoggerTelegramBackend, :telegram_filter},
    {LoggerTelegramBackend, :telegram_level},
    :console
  ]

config :logger, :telegram_filter,
  chat_id: "$chatId",
  token: "$botToken",
  metadata_filter: [application: :ui],
  metadata: [:line, :function, :module, :pid]

config :logger, :telegram_level,
  chat_id: "$chatId",
  token: "$botToken",
  level: :warn
```
