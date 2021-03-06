# Installation and Account Setup

To create an account, you first need to install the Mailscript CLI. The CLI is where you:

- Manage accounts.
- Claim addresses.
- Create keys for your addresses.
- Create workflows for your automations.
- Add triggers for your workflows.
- Add actions for your workflows.
- And more.

The first step is to download and install the executable.

## Install the CLI executable

Install the CLI executable from the npm registry: [mailscript](https://www.npmjs.com/package/mailscript) with

```sh
npm install -g mailscript
```

or

```sh
yarn global add mailscript
```

## Account setup

The next step is to initialize an account. The CLI uses a variety of identity providers, so choose the one you prefer.

```sh
mailscript login
```

### Offline support

The CLI offers a secondary login flow for setups without access to a browser instance (eg. a docker container). It will have ask you to go over a login flow elsewhere and return a code to resume the sign-in process within the CLI context.

```sh
mailscript login --offline
```

### Initialize


If this is the first time you are signing in you'll be prompted to pick a username. Mailscript automatically creates a new email address based on your username:

> username@mailscript.com

The cli will also prompt you to setup an `alias` workflow, that will alias any emails coming into `username@mailscript.com` to an email address you specify and verify.

With the initialization now complete you can test your new mailscript email by sending
an email to `username@mailscript.com` and receiving it at the aliasing address your specified.

## Addresses

`Addresses` allow you to send and receive email messages. You receive a top level domain address corresponding to your selected username (eg. _username@mailscript.com_) and you can create further addresses using your username as a first level subdomain (eg. _team@username.mailscript.com_).

You can create additional `addresses` at the cli:

```sh
mailscript addresses:add --address address@username.mailscript.com
```

### Keys

`Keys` allow you to share scoped access (_write_ and/or _read_) to addresses you control with other people. Whenever you add an address, a key is generated with full access for you.

You can list the keys for any address you control with the following command:

[your email address] will be the full address such as `username@mailscript.com` or `team@username.mailscript.com`

```
mailscript keys:list --address [your email address]
```

#### SMTP gateway

`Keys` with the _write_ access can be used to setup `smtp` access to allow 3rd party email clients to send messages from any mailscript address.

To do so use the following configuration in your email client:

```sh
host: smtp.mailscript.com
port: 465
user: full address (eg. username@mailscript.com)
pass: a key corresponding to the address with write access
```

Next, you can start setting up workflows whenever your addresses receive an email message.

## Workflows

Workflows allow you to setup automations when a message arrives at any of the addresses you control. A workflow consists of an input email address, optionally a trigger that filters down to the intended emails and an action. Whenever a trigger is met the corresponding actions will be executed. If there is not trigger then every email triggers the action.

### Triggers

A trigger is a named criteria used to filter down a stream of messages into the ones that should be acted upon.

Currently mailscript allows you to set up triggers based on incoming messages to addresses you control. You can set criteria to filter specific messages that the trigger will pass to the action (eg. messages sent from a specific address, messages that include attachments, that contain specific words in the subject or body, messages that contain attachments, mailscript even allows you to set up filters matching specific headers in your email message!).

Triggers can be composed together to form new triggers, for instance composing a trigger for all emails that both have the word 'alert' in their subject and come from the address of the website monitoring system.

### Actions

An action is definition of the effect a workflow should have when an email successfully passes the trigger contraints.

Mailscript offers outputs based on four different kinds of actions that can happen when a trigger is met:

- Email actions: send a new email message, forward the received email message, redirect the message to another address and reply to the sender or all participants in the received message.
- SMS action: send an sms text to a specified number.
- Webhook action: send a request to an endpoint. The request can be customized to suit your needs (eg. customize verbs, headers and payload; you can even use data from the received message into the delivered payload).
- Send to local Daemon: send the email parsed as json to a mailscript daemon you are running on a server/container/laptop you control. The daemon will invoke a script you specify on receiving a new email message.

## Daemon

The Mailscript cli allows you to use your local machine, or a server, as an output for an action in a workflow.

Using the `mailscript daemon` command you can run a local daemon that will execute a script you specify as part of a `workflow` and in response to an email.

### Setting up a daemon

First register a new daemon action:

```shell
mailscript actions:add --name "action name" --daemon "daemon name"
```

We can now include the daemon in a workflow:

```shell
mailscript workflows:add \
  --name "workflow name" \
  --trigger "trigger name" \
  --action "daemon name"
```

Any email sent to the input and that matches the trigger, will be forwarded to the daemon listening on the `daemon name` action. To setup such a daemon run:

```shell
mailscript daemon \
  --daemon "daemon name"
  --command "cowsay \$subject"
```

The command parameter specifies the shell script to run when the workflow sends an email to the daemon action, in this case the `cowsay` utility. The contents of the email are made available through the `$subject` and `$text` environment variables. The command will be executed each time on each received email.