# fluent-plugin-concat

[![Build Status](https://travis-ci.org/fluent-plugins-nursery/fluent-plugin-concat.svg?branch=master)](https://travis-ci.org/fluent-plugins-nursery/fluent-plugin-concat)

Fluentd Filter plugin to concatenate multiline log separated in multiple events.

## Requirements

| fluent-plugin-concat | fluentd    | ruby   |
|----------------------|------------|--------|
| >= 2.0.0             | >= v0.14.0 | >= 2.1 |
| < 2.0.0              | >= v0.12.0 | >= 1.9 |

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'fluent-plugin-concat'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install fluent-plugin-concat

## Plugin helpers

* [timer](https://docs.fluentd.org/v1.0/articles/api-plugin-helper-timer)
* [event_emitter](https://docs.fluentd.org/v1.0/articles/api-plugin-helper-event_emitter)

* See also: [Filter Plugin Overview](https://docs.fluentd.org/v1.0/articles/filter-plugin-overview)

## Configuration

### key (string) (required)

The key for part of multiline log.

### separator (string) (optional)

The separator of lines.

Default value: `"\n"`.

### n\_lines (integer) (optional)

The number of lines.
This is exclusive with following parameters:

* `multiline_start_regexp`
* `multiline_end_regexp`
* `partial_key`

### multiline\_start\_regexp (string) (optional)

The regexp to match beginning of multiline.
This is exclusive with following parameters:

* `n_lines`
* `partial_key`

### multiline\_end\_regexp (string) (optional)

The regexp to match ending of multiline.
This is exclusive with following parameters:

* `n_lines`
* `partial_key`

### continuous\_line\_regexp (string) (optional)

The regexp to match continuous lines.

### stream\_identity\_key (string) (optional)

The key to determine which stream an event belongs to

### flush\_interval (time) (optional)

The interval between data flushes, 0 means disable timeout (wait next line forever).

Default value: `60`.

### timeout\_label (string) (optional)

The label name to handle events caused by timeout.

### use\_first\_timestamp (bool) (optional)

Use timestamp of first record when buffer is flushed.

### partial\_key (string) (optional)

The field name that is the reference to concatenate records.
This is exclusive with following parameters:

* `n_lines`
* `multiline_start_regexp`
* `multiline_end_regexp`

### partial\_value (string) (optional)

The value stored in the field specified by partial_key that represent partial log.

### keep\_partial\_key (bool) (optional)

If true, keep partial_key in concatenated records.

Default value: `false`.

### buffer_limit_size (size) (optional)

The max size of each buffer

Default value: `512000`.

### buffer_overflow_method (enum) (optional)

The method if overflow buffer

Available values: `ignore`, `truncate`, `drop`, `new`

* `ignore`
  * Concatenate the current record in the buffer
* `truncate`
  * Drop the current record and flush the buffer
* `drop`
  * Drop the current record and clear the buffer
* `new`
  * Flush the buffer and store the current record in next buffer

Default value: `ignore`.

## Usage

Every 10 events will be concatenated into one event.

```aconf
<filter docker.log>
  @type concat
  key message
  n_lines 10
</filter>
```

Specify first line of multiline by regular expression.

```aconf
<filter docker.log>
  @type concat
  key message
  multiline_start_regexp /^Start/
</filter>
```

You can handle timeout events and remaining buffers on shutdown this plugin.

```aconf
<label @ERROR>
  <match docker.log>
    @type file
    path /path/to/error.log
  </match>
</label>
```

Handle timeout log lines the same as normal logs.

```aconf
<filter **>
  @type concat
  key message
  multiline_start_regexp /^Start/
  flush_interval 5
  timeout_label @NORMAL
</filter>

<match **>
  @type relabel
  @label @NORMAL
</match>

<label @NORMAL>
  <match **>
    @type stdout
  </match>
</label>
```

Handle single line JSON from Docker containers.

```aconf
<filter **>
  @type concat
  key message
  multiline_end_regexp /\n$/
</filter>
```

Handle Docker logs splitted in several parts (using `partial_message`), and do not add new line between parts.

```aconf
<filter>
  @type concat
  key message
  partial_key partial_message
  partial_value true
  separator ""
</filter>
```

Handle Docker logs splitted in several parts (using newline detection), and do not add new line between parts (prior to Docker 18.06).

```aconf
<filter **>
  @type concat
  key log
  multiline_end_regexp /\\n$/
  separator ""
</filter>
```

Handle containerd/cri in Kubernetes.

```aconf
<source>
  @type tail
  path /var/log/containers/*.log
  <parse>
    @type regexp
    expression /^(?<timestamp>\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\.\d+Z) (?<output>\w+) (?<partial_flag>[FP]) (?<message>.+)$/
  </parse>
  tag k8s
  @label @CONCAT
</source>

<label @CONCAT>
  <filter k8s>
    @type concat
    key message
    partial_key partial_flag
    partial_value P
  </filter>
  <match k8s>
    @type relabel
    @label @OUTPUT
  </match>
</label>

<label @OUTPUT>
  <match>
    @type stdout
  </match>
</label>
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

