# Banana Auth Vault Plugin

This repository contains sample code for a Hashicorp Vault Auth Plugin based on article <https://www.hashicorp.com/blog/building-a-vault-secure-plugin> 


## Testing Implementation:

* Create a temporary directory to compile the plugin into and to use as the plugin directory for Vault:

```shell
$ mkdir -p /tmp/vault-plugins
```

* Compile the plugin into the temporary directory:

```shell
$ go build -o /tmp/vault-plugins/banana-auth-vault-plugin
```

* Create a configuration file to point Vault at this plugin directory:

```shell
$ tee /tmp/vault.hcl <<EOF
plugin_directory = "/tmp/vault-plugins"
EOF
```

* Start a Vault server in development mode with the configuration:

```shell
$ vault server -dev -dev-root-token-id="root" -config=/tmp/vault.hcl
```

* Leave this running and open a new tab or terminal window. Authenticate to Vault:

```shell
$ export VAULT_ADDR='http://127.0.0.1:8200'
$ vault auth root
```

* Calculate and register the SHA256 sum of the plugin in Vault's plugin catalog:

```shell
$ SHASUM=$(shasum -a 256 "/tmp/vault-plugins/banana-auth-vault-plugin" | cut -d " " -f1)
vault write sys/plugins/catalog/banana-auth-vault-plugin \
  sha_256="$SHASUM" \
  command="banana-auth-vault-plugin"
```

* Enable the auth plugin:

```shell
$ vault auth enable -path=banana -plugin-name=banana-auth-vault-plugin plugin
```

* At this point, the plugin is registered and enabled. To test the implementation, submit a login request with an invalid secret:

```shell
$ vault write auth/banana/login password="laalaladada"

Error writing data to auth/banana/login: Error making API request.

URL: PUT http://127.0.0.1:8200/v1/auth/banana/loginCode: 403. Errors:

* permission denied
```

* Now submit a login request with the correct shared secret:

```shell
$vault write auth/banana/login password="banana"  

Key                Value
---                -----
token              244b6510-99f2-a2dd-01c2-90a928a191be
token_accessor     279b6660-ce07-e57e-daaa-86a65d5950a6
token_duration     30stoken_renewable    true
token_policies     [default]token_meta_song    lucille
```
