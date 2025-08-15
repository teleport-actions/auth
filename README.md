<div align="center">
   <img src="https://github.com/gravitational/teleport-actions/raw/main/assets/img/readme-header.png" width=750/>
   <div align="center" style="padding: 25px">
      <a href="https://www.apache.org/licenses/LICENSE-2.0">
      <img src="https://img.shields.io/badge/Apache-2.0-red.svg" />
      </a>
   </div>
</div>
</br>

> Read our Blog: <https://goteleport.com/blog/>

> Read our Documentation: <https://goteleport.com/docs/getting-started/>

# `teleport-actions/auth@v2`

`auth` uses Teleport Machine ID to generate a set of credentials which can be
used with other Teleport client tools such as `tsh` and `tctl`.

Pre-requisites:

- **Teleport 16 or above must be used.** Use
  [`teleport-actions/auth@v1`](https://github.com/teleport-actions/auth/tree/v1)
  for compatability with older versions of Teleport.
- Teleport binaries must already be installed in the job environment.
- You must have created a bot and created a GitHub join token that allows that
  bot to join.
- A Linux based runner.

Example usage:

```yaml
on:
  workflow_dispatch: {}
jobs:
  demo-auth:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
    steps:
      - name: Install Teleport
        uses: teleport-actions/setup@v1
        with:
          # specify version as "auto" and provide the address of your Teleport
          # proxy using the "proxy" input.
          version: auto
          proxy: tele.example.com:443
      - name: Authorize against Teleport
        id: auth
        uses: teleport-actions/auth@v2
        with:
          # Specify the publically accessible address of your Teleport proxy.
          proxy: tele.example.com:443
          # Specify the name of the join token for your bot.
          token: my-github-join-token-name
          # Specify the length of time that the generated credentials should be
          # valid for. This is optional and defaults to "1h"
          certificate-ttl: 1h
          # Enable submission of anonymous usage telemetry to Teleport.
          # See https://goteleport.com/docs/machine-id/reference/telemetry/ for
          # more information.
          anonymous-telemetry: 1
      - name: List nodes (tsh)
        run: tsh -i ${{ steps.auth.outputs.identity-file }} ls
      - name: List nodes (tctl)
        run: tctl -i ${{ steps.auth.outputs.identity-file }} --auth-server tele.example.com:443 nodes ls
      - name: Use OpenSSH with output config
        run: ssh -F ${{ steps.auth.outputs.ssh-config }} user@host "echo foobar"
```

Note that `tsh` and `tctl` require the flag pointing at the identity file and
`tctl` also requires the address of the Proxy or Auth Server to be provided.

## Inputs

The following inputs are required:

- `proxy`: String. The publically accessible address of your Teleport Proxy.
- `token`: String. The name of the GitHub join token for your bot.

The following inputs are optional:

- `allow-reissue`: Boolean. If set to `true`, the action will issue an identity
  file that permits reissuance. This allows the identity file to be used with
  `tsh` commands that require new certificates to be issued, such as
  `tsh db login`.

## Environment Variables

By default, this action will set the following environment variables:

- `TELEPORT_AUTH_SERVER`: the address of the Teleport Auth Server.
- `TELEPORT_PROXY`: the address of the Teleport Proxy.
- `TELEPORT_IDENTITY_FILE`: the path to the identity file.

This will automatically configure tools like `tsh` and `tctl` to use the
generated credentials. However, this can cause issues if you intend to invoke
`tbot` multiple times.

You can disable this behaviour by setting the `disable-env-vars` input to
`true`.

## Outputs

This action will output the following values:

- `destination-dir`: the path to the tbot destination folder.
- `identity-file`: the path to the identity file.
- `ssh-config`: the path to the generated SSH config.

## Next steps

Read the `teleport-actions/auth` getting started guide:
<https://goteleport.com/docs/machine-id/guides/github-actions/>
