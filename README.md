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

# `teleport-actions/auth`

`auth` uses Teleport Machine ID to generate a set of credentials which can be
used with other Teleport client tools such as `tsh` and `tctl`.

The action has the following outputs:

- `identity-file`: the path to the identity file which can be used with `tctl` and `tsh`.
- `ssh-config`: the path to the generated SSH config which can be used with `ssh`.

Pre-requisites:

- Teleport 11 or above must be used.
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
          version: 12.1.0
      - name: Authorize against Teleport
        id: auth
        uses: teleport-actions/auth@v1
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

## Next steps

Read the `teleport-actions/auth` getting started guide:
<https://goteleport.com/docs/machine-id/guides/github-actions/>
