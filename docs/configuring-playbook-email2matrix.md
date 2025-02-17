# Setting up Email2Matrix (optional)

The playbook can install and configure [email2matrix](https://github.com/devture/email2matrix) for you.

See the project's [documentation](https://github.com/devture/email2matrix/blob/master/docs/README.md) to learn what it does and why it might be useful to you.


## Preparation

### Port availability

Ensure that port 25 is available on your Matrix server and open in your firewall.
If you have `postfix` or some other email server software installed, you may need to manually remove it first (unless you need it, of course).

If you really need to run an email server on the Matrix machine for other purposes, it may be possible to run Email2Matrix on another port (with a configuration like `matrix_email2matrix_smtp_host_bind_port: "127.0.0.01:2525"`) and have your other email server relay messages there.

### Creating a user

Before enabling Email2Matrix, you'd most likely wish to create a dedicated user (or more) that would be sending messages on the Matrix side.
Refer to [Registering users](registering-users.md) for ways to do that. A regular (non-admin) user works best.

### Creating a shared room

After creating a sender user, you should create one or more Matrix rooms that you share with that user.
It doesn't matter who creates and owns the rooms and who joins later (you or the sender user).

What matters is that both you and the sender user are part of the same room and that the sender user has enough privileges in the room to be able to send messages there.
Inviting additional people to the room is okay too.

Take note of each room's room id (different clients show the room id in a different place).
You'll need the room id when doing [Configuration](#configuration) below.


### Obtaining an access token for the sender user

In order for the sender user created above to be able to send messages to the room, we'll need to obtain an access token for it.

To do this, you can execute a command like this:

```
curl \
--data '{"identifier": {"type": "m.id.user", "user": "email2matrix" }, "password": "MATRIX_PASSWORD_FOR_THE_USER", "type": "m.login.password", "device_id": "Email2Matrix", "initial_device_display_name": "Email2Matrix"}' \
https://matrix.DOMAIN/_matrix/client/r0/login
```

Take note of the `access_token` value. You'll need the access token when doing [Configuration](#configuration) below.


## Configuration

After doing the preparation steps above, adjust your `inventory/host_vars/matrix.DOMAIN/vars.yml` configuration like this:

```yaml
matrix_email2matrix_enabled: true

matrix_email2matrix_matrix_mappings:
  - MailboxName: "my-mailbox"
    MatrixRoomId: "!someRoom:DOMAIN"
    MatrixHomeserverUrl: "https://matrix.DOMAIN"
    MatrixUserId: "@email2matrix:DOMAIN"
    MatrixAccessToken: "ACCESS_TOKEN_GOES_HERE"
    IgnoreSubject: false
    IgnoreBody: false
    SkipMarkdown: false

  - MailboxName: "my-mailbox2"
    MatrixRoomId: "!anotherRoom:DOMAIN"
    MatrixHomeserverUrl: "https://matrix.DOMAIN"
    MatrixUserId: "@email2matrix:DOMAIN"
    MatrixAccessToken: "ACCESS_TOKEN_GOES_HERE"
    IgnoreSubject: true
    IgnoreBody: false
    SkipMarkdown: true
```

You can also set `MatrixHomeserverUrl` to `http://matrix-synapse:8008`, instead of the public `https://matrix.DOMAIN`.
However, that's more likely to break in the future if you switch to another server implementation than Synapse.

Re-run the playbook (`--tags=setup-email2matrix,start`) and try sending an email to `my-mailbox@matrix.DOMAIN`.
