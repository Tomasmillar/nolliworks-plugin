# Privacy notice: the nolliworks-graph plugin

Last updated: 9 November 2026

This notice covers the `nolliworks-graph` plugin for Claude and the Nolliworks
MCP server it connects to. It is provided by **Ruris (UK) Ltd**, which runs
Nolliworks. Everything else about how Nolliworks handles personal data, such as
your account, your practice's records, retention and your rights, is in the
Nolliworks privacy policy in the app, which also applies here.

## What the plugin is

The plugin is a set of text files installed on your computer: a connection
setting, instructions for Claude (a skill), and some commands. The plugin itself
collects nothing and sends nothing to Ruris (UK) Ltd. Data only moves when you
connect it to Nolliworks and Claude uses the tools it offers.

## Signing in

When you connect, your browser opens Nolliworks. You sign in there and approve
the connection. Sign-in uses Supabase Auth, the same as signing in to the app.
Your AI client is then given an access token. Your password is never given to
the client. Afterwards, every request acts as you, with exactly the permissions
you have in Nolliworks. You can disconnect at any time from your AI client.

## What passes through the connection

- **From Nolliworks to your AI client:** what Claude asks for, limited to what
  you could see in the app yourself. That can include job records, people's
  names and roles, and the text of sources such as emails and meeting notes.
- **From your AI client to Nolliworks:** what Claude adds on your behalf. It
  proposes additions to a job's inbox for a person to confirm, and makes direct
  changes only when you have told it to. Everything it writes is recorded as
  written by you, through Claude, with the same history and audit trail as the
  rest of the app.

## Your AI provider

Whatever Nolliworks sends to your AI client is then handled by that client's
provider, for example Anthropic for Claude, under your or your organisation's
agreement with them. Ruris (UK) Ltd does not control what the provider does with
it. Before you connect, check that your practice is content for its records to
be processed by that provider.

## What Ruris (UK) Ltd keeps

Nothing extra. Requests through the connection are handled like requests from
the app, and what Claude writes is stored in your practice's Nolliworks records.
The connection itself is recorded so that it can be confirmed, for people who
use an authenticator app, and disconnected.

## Contact

Questions about this notice or your data: Ruris (UK) Ltd, [privacy contact to be confirmed].
