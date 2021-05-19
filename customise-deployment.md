# Customised Deployment

Magda is built in a modular way and you can customise your Magda deployment by creating your own Helm chart and add required Magda Module Helm Charts as [dependencies](https://helm.sh/docs/topics/charts/#chart-dependencies).

You can find the list of core Magda Helm charts and their docs from [here](https://github.com/magda-io/magda/blob/master/docs/docs/helm-charts-docs-index.md).

The previously deployed [magda](https://github.com/magda-io/magda/blob/master/deploy/helm/magda/README.md) chart doesn't include any [connectors](https://github.com/magda-io/magda/blob/master/docs/docs/how-to-build-your-own-connectors-minions.md) nor [any authentication plugins](https://github.com/magda-io/magda/blob/master/docs/docs/authentication-plugin-how-to-use.md).

Here is an example: [my-magda-app](./my-magda-app/) chart to deploy your Magda:
- with a connector that crawl datasets from a ckan instance
- with "internal" authentication plugin that allows user to login with local password

# Deployment

> Here, we assume you've already followed the [Simple Magda Deployment with Magda Helm Chart](./simple-deployment.md) and deploy the magda. Therefore, we won't repeat the steps of creating secrets and simply upgrade the existing `magda` deployment.

```bash
# Download & build the helm chart dependencies
helm dep up ./my-magda-app
```

```bash
# Deployment my-magda-app Chart.
helm upgrade --namespace magda --install --timeout 9999s magda ./my-magda-app
```

You can run:

```bash
echo $(kubectl get svc --namespace magda gateway --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
```

to find out the load balancer external IP. And access Magda via http://[External IP].

Once we are able to access the Magda site, we need to update Magda with correct external url:

```bash
helm upgrade --namespace magda --install --timeout 9999s --set global.externalUrl=http://[External IP]/ magda ./my-magda-app
```

# Create a local user

- Install [Node.js 12](https://nodejs.org/en/) & [yarn](https://classic.yarnpkg.com/en/docs/install/#mac-stable)
- Clone Repo: https://github.com/magda-io/magda-auth-internal
- Go to repo root, run `yarn install`
- port-forward pod `combined-db-0`: `kubectl -n magda port-forward combined-db-0 5432`
- run `yarn set-user-password -c myemail@email.com`

# Make the user as Admin

- Install [@magda/acs-cmd](https://www.npmjs.com/package/@magda/acs-cmd) Utility.
  - e.g. `npm install --global @magda/acs-cmd`
- Make sure you've port forwarded `combined-db-0`
  - `kubectl -n magda port-forward combined-db-0 5432`
- Run `acs-cmd list users` & note down the user ID
- Run `acs-cmd admin set <userId>`

# Create API key for the User

- Clone Repo: https://github.com/magda-io/magda
- Go to repo root, run `yarn install`
- port-forward pod `combined-db-0`: `kubectl -n magda port-forward combined-db-0 5432`
- run `yarn create-api-key -u [user id] -c`



