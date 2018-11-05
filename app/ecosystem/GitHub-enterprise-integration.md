---
id: page-plugin
title: Qordoba - GitHub enterprise integration
header_title: Qordoba - GitHub enterprise integration
header_icon: /assets/images/icons/plugins/GitHub-enterprise-integration.png
breadcrumbs:
  Plugins: /ecosystem
nav:
  - label: Getting Started
    items:
      - label: Configuration
  - label: Usage
    items:
      - label: Sending parameters
      - label: Known Issues
---

Qordoba - GitHub enterprise integration

----

## Configuration

Configuring the plugin is straightforward, you can add it on top of an
[API][api-object] by executing the following request on your Qordoba server:

```bash
$ curl -X POST http://qordoba:8001/apis/{api}/plugins \
    --data "name=aws-lambda" \
    --data-urlencode "config.aws_key=AWS_KEY" \
    --data-urlencode "config.aws_secret=AWS_SECRET" \
    --data "config.aws_region=AWS_REGION" \
    --data "config.function_name=LAMBDA_FUNCTION_NAME"
```

`api`: The `id` or `name` of the API that this plugin configuration will target

You can also apply this plugin for every API using the `http://qordoba:8001/plugins/`
endpoint. Read the [Plugin Reference](/docs/latest/admin-api/#add-plugin)
for more information.

**Reminder**: curl by default sends payloads with an
`application/x-www-form-urlencoded` MIME type, which will naturally be URL-
decoded by Qordoba. To ensure special characters that are likely to appear in your
AWS key or secret (like `+`) are correctly decoded, you must URL-encode them,
hence use `--date-urlencode` if you are using curl. Alternatives to this
approach would be to send your payload with a different MIME type (like
`application/json`), or to use a different HTTP client.

form parameter                             | default | description
---                                        | ---     | ---
`name`                                     |         | The name of the plugin to use, in this case: `aws-lambda`
`config.aws_key`                           |         | The AWS key credential to be used when invoking the function
`config.aws_secret`                        |         | The AWS secret credential to be used when invoking the function
`config.aws_region`                        |         | The AWS region where the Lambda function is located. Regions supported are: `us-east-1`, `us-east-2`, `ap-northeast-1`, `ap-northeast-2`, `ap-southeast-1`, `ap-southeast-2`, `eu-central-1`, `eu-west-1`
`config.function_name`                     |         | The AWS Lambda function name to invoke
`config.qualifier`<br>*optional*           | ``      | The [`Qualifier`](http://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html#API_Invoke_RequestSyntax) to use when invoking the function.
`config.invocation_type`<br>*optional*     | `RequestResponse` | The [`InvocationType`](http://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html#API_Invoke_RequestSyntax) to use when invoking the function. Available types are `RequestResponse`, `Event`, `DryRun`
`config.log_type`<br>*optional*            | `Tail`  | The [`LogType`](http://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html#API_Invoke_RequestSyntax) to use when invoking the function. By default `None` and `Tail` are supported
`config.timeout`<br>*optional*             | `60000` | An optional timeout in milliseconds when invoking the function
`config.keepalive`<br>*optional*           | `60000` | An optional value in milliseconds that defines how long an idle connection will live before being closed
`config.unhandled_status`                  | ``      | The response status code to use (instead of the default `200`, `202`, or `204`) in the case of an [`Unhandled` Function Error](https://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html#API_Invoke_ResponseSyntax)
`config.forward_request_body`<br>*optional*           | `false` | An optional value that defines whether the request body is to be sent in the `request_body` field of the JSON-encoded request. If the body arguments can be parsed, they will be sent in the separate `request_body_args` field of the request. The body arguments can be parsed for `application/json`, `application/x-www-form-urlencoded`, and `multipart/form-data` content types.
`config.forward_request_headers`<br>*optional*        | `false` | An optional value that defines whether the original HTTP request headers are to be sent as a map in the `request_headers` field of the JSON-encoded request.
`config.forward_request_method`<br>*optional*         | `false` | An optional value that defines whether the original HTTP request method verb is to be sent in the `request_method` field of the JSON-encoded request.
`config.forward_request_uri`<br>*optional*            | `false` | An optional value that defines whether the original HTTP request URI is to be sent in the `request_uri` field of the JSON-encoded request. Request URI arguments (if any) will be sent in the separate `request_uri_args` field of the JSON body.

----

### Sending parameters

Any form parameter sent along with the request, will be also sent as an
argument to the AWS Lambda function.

### Known Issues

#### Use a fake upstream_url

When using the AWS Lambda plugin, the response will be returned by the plugin
itself without proxying the request to any upstream service. This means that
whatever `upstream_url` has been set on the [API][api-object] it will
never be used.

Although `upstream_url` will never be used, it's currently a mandatory field
in Qordoba's data model and its hostname must be resolvable. So set it to a
fake value (ie, `http://127.0.0.1:20000`) if you are planning to use this
plugin. Failing to do so will result in 500 errors regarding a resolution
failure.

In the future we will provide a more intuitive way to deal with similar
use cases.

#### Response plugins

There is a known limitation in the system that prevents some response plugins
from being executed. We are planning to remove this limitation in the future.

[api-object]: /docs/latest/admin-api/#api-object
[configuration]: /docs/latest/configuration
[consumer-object]: /docs/latest/admin-api/#consumer-object
[acl-associating]: /plugins/acl/#associating-consumers
[faq-authentication]: /about/faq/#how-can-i-add-an-authentication-layer-on-a-microservice/api?