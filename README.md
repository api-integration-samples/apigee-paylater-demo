# Apigee Pay Later Car Demo
This is a demo of creating an API for pay later transactions when buying a car.

## Prerequisites
- Visual Studio Code, Google Cloud Code extension installed.
- [Apigeecli](https://github.com/apigee/apigeecli) tool installed for automating Apigee.

## Instructions

1. Create an API using Gemini and the Google Cloud Code Apigee extension using natural language. We will use the prompt **"API for pay later for buying cars"**.

![Create API with Gemini, Apigee and Google Cloud Code](/img/create-api.gif)

2. Create an Apigee API Proxy from the Open API Specification that was created.

![Create API Proxy from API Hub Open API Specification](/img/create-proxy.gif)]

3. Set a backend target to call in the API proxy. We can use this test backend (https://mirror-service-323709580283.europe-west4.run.app/).

![Set the target service for the API proxy.](/img/set-target-server.gif)

4. Deploy API proxy to Apigee using `[apigeecli](https://github.com/apigee/apigeecli)`. You could also do this in the [Apigee console](https://console.cloud.google.com/apigee/proxies) or with [Terraform](https://github.com/apigee/terraform-modules).

```sh
cd src/main/apigee/apiproxies/CarPayLaterAPI-v1
API_NAME=CarPayLaterAPI-v1
PROJECT_ID=apigee-hub-demo
APIGEE_ENV=dev
apigeecli apis create bundle -f apiproxy -n $API_NAME -o $PROJECT_ID -e $APIGEE_ENV -t $(gcloud auth print-access-token) --ovr
cd ../../../../..
```

![Deploy Apigee API proxy](/img/deploy-api.gif)

5. Create an API Product to manage access to the API. You could also do this in the [Apigee console](https://console.cloud.google.com/apigee/apiproducts) or with [Terraform](https://github.com/apigee/terraform-modules).

```sh
PRODUCT_NAME="Car Pay Later API v1"
apigeecli products create -n "$PRODUCT_NAME" \
  -m "$PRODUCT_NAME" \
  -o "$PROJECT_ID" -e $APIGEE_ENV \
  -f auto -p "$API_NAME" -t $(gcloud auth print-access-token)
```

6. Create a test developer and app to call API with. You could also do this in the [Apigee console](https://console.cloud.google.com/apigee/developers) or with [Terraform](https://github.com/apigee/terraform-modules).

```sh
// create developer
DEVELOPER_EMAIL="example-developer@cymbalgroup.com"
apigeecli developers create -n "$DEVELOPER_EMAIL" \
  -f "Example" -s "Developer" \
  -u "$DEVELOPER_EMAIL" -o "$PROJECT_ID" -t $(gcloud auth print-access-token)

// create app and get key or token
APP_NAME=car-web-app1
API_KEY=$(apigeecli apps create --name "$APP_NAME" \
  --email "$DEVELOPER_EMAIL" \
  --prods "$PRODUCT_NAME" \
  --org "$PROJECT_ID" --token $(gcloud auth print-access-token) | jq --raw-output '.credentials[0].consumerKey')
echo $API_KEY
```

7. Get API endpoint and test API. Start a [debug session in Apigee](https://console.cloud.google.com/apigee/proxies/CarPayLaterAPI-v1/debug) to see the request & processing.

```sh
HOST_NAME=$(apigeecli envgroups list -o $PROJECT_ID -t $(gcloud auth print-access-token) | jq --raw-output '.environmentGroups[] | select(.name == '\""$APIGEE_ENV\""') | .hostnames[0]')
API_ENDPOINT="https://$HOST_NAME/"

echo ""
echo "Calling API endpoint $API_ENDPOINT/cars"
curl "$API_ENDPOINT/cars"
```

8. Exercise - add the API key validation policy to the API proxy pre-flow, and retest with the API key that we retrieved in step 6.