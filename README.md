# Apigee Firebase Auth / Identity Platform Demo
This demo shows how to validate firebase auth tokens in an Apigee proxy.

## Deploy
1. Clone this repository with `git clone https://github.com/tyayers/apigee-firebase-auth-demo.git`.
2. Setup an Apigee instance in a Google Cloud region either as Evaluation, Pay-as-you-go, or Subscription, as documented [here](https://docs.cloud.google.com/apigee/docs/api-platform/get-started/provisioning-options).
3. Setup the Firebase Auth / Identity Platform in your Google Cloud project as documented [here](https://docs.cloud.google.com/identity-platform/docs/sign-in-user-email), and then register a user with email and password.
4. Set environment variables:
```sh
export GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
export APIGEE_ENV=YOUR_APIGEE_ENVIRONMENT
export FIREBASE_API_KEY=YOUR_FIREBASE_KEY
export USER_EMAIL=EMAIL
export USER_PASSWORD=PASSWORD
```
5. Deploy Apigee proxy to your Apigee project. [Apigee Feature Templater](https://github.com/apigee/apigee-templater) is used to deploy.
```sh
# install aft
npm i apigee-templater -g
# replace issuer and audience in proxy
sed -i "s,^        issuer: .*,        issuer: https://securetoken.google.com/$GOOGLE_CLOUD_PROJECT," FirebaseAuthProxy.yaml
sed -i "s,^        audience:.*,        audience: $GOOGLE_CLOUD_PROJECT," FirebaseAuthProxy.yaml

# deploy proxy
aft -i ./FirebaseAuthProxy.yaml -o "$GOOGLE_CLOUD_PROJECT:FirebaseAuthProxy:$APIGEE_ENV"
```
6. Get an id token from identity platform / firebase auth:
```sh
ID_TOKEN=$(curl "https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=$FIREBASE_API_KEY" \
  -H 'Content-Type: application/json' \
  --data-binary @- << EOF  | jq --raw-output '.idToken'

{
  "email": "$USER_EMAIL",
  "password": "$USER_PASSWORD",
  "returnSecureToken": true
}
EOF)
```
6. Call the Apigee proxy
```sh
# get apigee host
HOST=$(apigeecli envgroups list -o $GOOGLE_CLOUD_PROJECT --default-token | jq --raw-output '.environmentGroups[0].hostnames[-1]')
# call gateway proxy
curl -i https://$HOST/firebaseauthdemo -H "Authorization: Bearer $ID_TOKEN"
```
