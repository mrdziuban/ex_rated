# ExRated

ExRated is:

1. A port of the Erlang '[raterlimiter](https://github.com/Gromina/raterlimiter)' project to Elixir.
2. An OTP GenServer process that allows you to rate limit calls to something like an external API.
3. The Hex.pm package with the naughty name.

You can learn more about the concept for this rate limiter in [ the Token Bucket article on Wikipedia](http://en.wikipedia.org/wiki/Token_bucket)


## Usage

Call the ExRated application with `ExRated.check_rate/3`.  This function takes three arguments:

1. A `bucket name` (String).  You can have as many buckets as you need.
2. A `scale` (Integer). The time scale in milliseconds that the bucket is valid for.
3. A `limit` (Integer). How many actions you want to limit your app to in the time scale provided.

For example, if you have to enforce a rate limit of no more than 5 calls in 10 seconds to your API:

```elixir
iex> ExRated.check_rate("my-rate-limited-api", 10_000, 5)
{:ok, 1}
```

The `ExRated.check_rate` function will return an `{:ok, Integer}` tuple if its OK to proceed with your rate limited function. The Integer returned is the current value of the incrementing counter showing how many times in the time scale window your function has already been called. If you are over limit a `{:fail, Integer}` tuple will be returned where the Integer is always the limit you have specified in the function call.

## Installation

You can use ExRated in your projects in two steps:

1. Add ExRated to your `mix.exs` dependencies:

    ```elixir
    def deps do
      [{:ex_rated, "~> 0.0.6"}]
    end
    ```

2. List `:ex_rated` in your application dependencies:

    ```elixir
    def application do
      [applications: [:ex_rated]]
    end
    ```

You can also start the GenServer manually, and pass it custom config, with something like:

```elixir
{:ok, pid} = GenServer.start_link(ExRated, [ {:timeout, 10_000}, {:cleanup_rate, 10_000}, {:ets_table_name, :ex_rated_buckets} ], [name: :ex_rated])
```

Where the args and their defaults are:

`{:timeout, 90_000_000}` : buckets older than this in milliseconds will be automatically pruned.

`{:cleanup_rate, 60_000}` : how often, in milliseconds, the bucket pruning process will be run.

`{:ets_table_name, :ex_rated_buckets}` : The atom name of the ETS table.

`[name: :ex_rated]` : The registered name of the ExRated GenServer.


## Testing

It is important that the OTP doesn't get automatically started by Mix.

```elixir
mix test --no-start
```

## Is it fast?

You can use the `Benchwarmer` library to do a quick performance test.

Temporarily add `Benchwarmer` to your dependencies in `mix.exs` as shown below and run `mix deps.get` and `iex -S mix`:

```
defp deps do
  [
    {:ex2ms, "~> 1.3.0"},
    {:benchwarmer, "~> 0.0.2"}
  ]
end
```

On my 2014 Macbook Pro I can do 262,000 checks in about 1.2 seconds.

```elixir
iex> Benchwarmer.benchmark fn -> {:ok, _} = ExRated.check_rate("my-bucket", 1000000, 10_000_000) end
*** #Function<20.90072148/0 in :erl_eval.expr/5> ***
1.2 sec   262K iterations   4.9 μs/op
```

## Changes

### v0.0.6

  - ExRated internally calls `:erlang.system_time(:milli_seconds)` provided by the new [Time API in OTP 18](http://www.erlang.org/doc/apps/erts/time_correction.html) and greater if available, and will fall back gracefully to the old `:erlang.now()` in older versions. Thanks to Mitchell Henke (mitchellhenke) for the enhancement.

## License

ExRated source code is released under Apache 2 License.
Check LICENSE file for more information.
