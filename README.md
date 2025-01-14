# slack-backup

This Python script downloads a backup of all Slack users, all channels
(*including private channels and Direct Messages*), and all messages in those channels.
The produced dump is in the same format produced by an
[official Slack export of workspace data](https://slack.com/help/articles/201658943-Export-your-workspace-data)
(which works well for public channels but not private channels).

The intended use-case is for moving old Slack content over to a new
Discord server via bluescopedata's [slack-to-discord](https://github.com/bluescopedata/slack-to-discord) (This is an updated fork of [slack-to-discord](https://github.com/pR0Ps/slack-to-discord). It is necessary to use this fork with this update as it it imports files from your local backup, rather than from the slack url, perfoming some image shrinking and zipping in case of oversized files, before falling back to slack. It also supports the Direct Message uploads.)

One motivation is Slack's
[change in limits to the free plan](https://slack.com/help/articles/7050776459923-Pricing-changes-for-the-Pro-plan-and-updates-to-the-Free-plan).

## Usage

### Running slack-backup

1. Clone this repository.
2. Install Python 3 if needed.
3. `pip install slack_sdk`
4. [Create a Slack app](https://api.slack.com/apps/new)
5. Under OAuth &amp; Permissions, in the User Token Scopes section
   (I've found the user token to work better than the bot token, but YMMV),
   add the following scopes:

   * `channels:history`
   * `channels:read`
   * `files:read`
   * `groups:history`
   * `groups:read`
   * `users:read`
   * `users:read.email`
   * `im:history`
   * `im:read`
   * `mpim:history`
   * `mpim:read`

   Also write down the Bot User OAuth Token.
   (These directions are based on
   [this documentation](https://github.com/docmarionum1/slack-archive-bot).)
6. Under Install App, be sure to install the bot to the workspace
   you'd like to backup.
7. Assuming you're using the User Token, be sure that you have been added to
   all private channels that you want to backup.  (Even admins/owners cannot
   see all private channels in the channel list; they need to be invited.)
   If you're using the Bot User Token, you probably need to invite the bot
   to all desired channels.
8. Then I suggest creating a `run` script with the following contents:

   ```sh
   #!/bin/sh
   export TOKEN='xoxp-...'  # Bot User OAuth Token
   export DOWNLOAD=1  # download all message files locally too
   python slack_backup.py
   ```
9. Run the `run` script via `./run` and wait.
10. The output will be in a created `backup` subdirectory.
11. To produce a `backup.zip` file in the same format as a Slack export,
    do the following in a shell (assuming you have `zip` installed):

    ```sh
    cd backup
    zip -9r ../backup.zip *
    ```

### Running slack-to-discord

If you want to use [slack-to-discord](https://github.com/bluescopedata/slack-to-discord)
to convert your export to Discord, follow [instructions in the project's
README](https://github.com/bluescopedata/slack-to-discord#instructions).

A slight tweak to the last few instructions on running slack-to-discord:
I suggest creating a `run` script with the following contents:

```sh
#!/bin/sh
slack-to-discord --zipfile path/to/backup.zip --guild 'Name of Server' --token MTA...
```

where the last part is your Discord bot token.

After creating that file, run it via `./run` and wait.
You can watch the messages roll into the server.
Occasionally the script may pause because of Discord's rate limits.

## History

This code was initially based on a
[gist](https://gist.github.com/benoit-cty/a5855dea9a4b7af03f1f53c07ee48d3c)
by [Benoit Courty](https://gist.github.com/benoit-cty).
This fork adds channel listing (including Direct Messages), user listing, file downloads, and
making the output format compatible with standard Slack dumps.
