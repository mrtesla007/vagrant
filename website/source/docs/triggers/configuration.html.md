---
layout: "docs"
page_title: "Vagrant Triggers Configuration"
sidebar_current: "triggers-configuration"
description: |-
  Documentation of various configuration options for Vagrant Triggers
---

# Configuration

Vagrant Triggers has a few options to define trigger behavior.

## Execution Order

The trigger config block takes two different operations that determine when a trigger
should fire:

* `before`
* `after`

These define _how_ the trigger behaves and when it should fire off during
the Vagrant life cycle. A simple example of a _before_ operation could look like:

```ruby
config.trigger.before :up do |t|
  t.info = "Bringing up your Vagrant guest machine!"
end
```

Triggers can also be used with [_commands_](#commands), [_actions_](#actions), or [_hooks_](#hooks).
By default triggers will be defined to run before or after a Vagrant guest. For more
detailed examples of how to use triggers, check out the [usage section](/docs/triggers/usage.html).

## Trigger Options

The trigger class takes various options.

* `action` (symbol, array) - Expected to be a single symbol value, an array of symbols, or a _splat_ of symbols. The first argument that comes after either __before__ or __after__ when defining a new trigger. Can be any valid Vagrant command. It also accepts a special value `:all` which will make the trigger fire for every action. An action can be ignored with the `ignore` setting if desired. These are the valid action commands for triggers:

  - `destroy`
  - `halt`
  - `provision`
  - `reload`
  - `resume`
  - `suspend`
  - `up`

* `ignore` (symbol, array) - Symbol or array of symbols corresponding to the action that a trigger should not fire on.

* `info` (string) - A message that will be printed at the beginning of a trigger.

* `name` (string) - The name of the trigger. If set, the name will be displayed when firing the trigger.

* `on_error` (symbol) - Defines how the trigger should behave if it encounters an error. By default this will be `:halt`, but can be configured to ignore failures and continue on with `:continue`.

* `only_on` (string, regex, array) - Limit the trigger to these guests. Values can be a string or regex that matches a guest name.

* `ruby` (block) - A block of Ruby code to be executed on the host. The block accepts two arguments that can be used with your Ruby code: `env` and `machine`. These options correspond to the Vagrant environment used (note: these are not your shell's environment variables), and the Vagrant guest machine that the trigger is firing on. This option can only be a `Proc` type, which must be explicitly called out when using the hash syntax for a trigger.

    ```ruby
    ubuntu.trigger.after :up do |trigger|
      trigger.info = "More information"
      trigger.ruby do |env,machine|
        greetings = "hello there #{machine.id}!"
        puts greetings
      end
    end
    ```

* `run_remote` (hash) - A collection of settings to run a inline or remote script with on the guest. These settings correspond to the [shell provisioner](/docs/provisioning/shell.html).

* `run` (hash) - A collection of settings to run a inline or remote script on the host. These settings correspond to the [shell provisioner](/docs/provisioning/shell.html). However, at the moment the only settings `run` takes advantage of are:
  + `args`
  + `inline`
  + `path`

* `warn` (string) - A warning message that will be printed at the beginning of a trigger.

* `exit_codes` (integer, array) - A set of acceptable exit codes to continue on. Defaults to `0` if option is absent. For now only valid with the `run` option.

* `abort` (integer,boolean) - An option that will exit the running Vagrant process once the trigger fires. If set to `true`, Vagrant will use exit code 1. Otherwise, an integer can be provided and Vagrant will it as its exit code when aborting.

## Trigger Types

Optionally, it is possible to define a trigger that executes around Vagrant commands,
hooks, and actions.

<div class="alert alert-warning">
  <strong>Warning!</strong> This feature is still experimental and may break or
  change in between releases. Use at your own risk.

  This feature currently reqiures the experimental flag to be used. To explicitly enable this feature, you can set the experimental flag to:

  ```
  VAGRANT_EXPERIMENTAL="typed_triggers"
  ```

  Please note that `VAGRANT_EXPERIMENTAL` is an environment variable. For more
  information about this flag visit the [Experimental docs page](/docs/experimental/)
  for more info. Without this flag enabled, triggers with the `:type` option
  will be ignored.
</div>


A trigger can be one of three types:

* `type` (symbol) - Optional
  - `:action` - Action triggers run before or after a Vagrant action
  - `:command` - Command triggers run before or after a Vagrant command
  - `:hook` - Action hook triggers run before or after a Vagrant hook

These types determine when and where a defined trigger will execute.

```ruby
config.trigger.after :destroy, type: :command do |t|
  t.warn = "Destroy command completed"
end
```

#### Quick Note

Triggers _without_ the type option will run before or after a Vagrant guest.

Older Vagrant versions will unfortunetly not be able to properly parse the new
`:type` option. If you are worried about older clients failing to parse your Vagrantfile,
you can guard the new trigger based on the version of Vagrant:

```ruby
if Vagrant.version?(">= 2.3.0")
  config.trigger.before :status, type: :command do |t|
    t.info = "before action!!!!!!!"
  end
end
```

### Commands

Command typed triggers can be defined for any valid Vagrant command. They will always
run before or after the command.

The difference between this and the default behavior is that these triggers are
not attached to any specific guest, and will always run before or after the given
command. A simple example might be running a trigger before the up command to give
a simple message to the user:

```ruby
config.trigger.before :up, type: :command do |t|
  t.info = "Before command!"
end
```

For a more detailed example, please check out the [examples](/docs/triggers/usage.html#commands)
page for more.

### Hooks

<div class="alert alert-warning">
  <strong>Advanced topic!</strong> This is an advanced topic for use only if
  you want to execute triggers around Vagrant hooks. If you are just getting
  started with Vagrant and triggers, you may safely skip this section.
</div>

Hook typed triggers can be defined for any valid Vagrant action hook that is defined.

A simple example would be running a trigger on a given hook called `action_hook_name`.

```ruby
config.trigger.after :action_hook_name, type: :hook do |t|
  t.info = "After action hook!"
end
```

For a more detailed example, please check out the [examples](/docs/triggers/usage.html#hooks)
page for more.

### Actions

<div class="alert alert-warning">
  <strong>Advanced topic!</strong> This is an advanced topic for use only if
  you want to execute triggers around Vagrant actions. If you are just getting
  started with Vagrant and triggers, you may safely skip this section.
</div>

Action typed triggers can be defined for any valid Vagrant action class. Actions
in this case refer to the Vagrant class `#Action`, which is used internally to
Vagrant and in every Vagrant plugin.

```ruby
config.trigger.before :"Action::Class::Name", type: :action do |t|
  t.info = "Before action class!
end
```

For a more detailed example, please check out the [examples](/docs/triggers/usage.html#actions)
page for more.
