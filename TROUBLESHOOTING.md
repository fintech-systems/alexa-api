# Troubleshooting

All troubleshooting queues to `alexa-api` will be documented in this file.

## 2021-11-07

Q. Question:

"Alexa, ask the system what's the most important task?"

A. Answer:

A slight delay, and then:

"There was a problem with the requested skill's response"

Resolution:

Log into Alexa Developer Portal to see if there are any obvious problems.

Issue:

It's not obvious what this URL is, so we have to Google it.
log into amazon alexa developer account
https://developer.amazon.com › alexa › console › ask
https://developer.amazon.com/alexa/console/ask?

This console shows everything you need and some tabs are documented below.

In this case we were able to:

- Deduce the endpoint by logging on
- Check the Laravel log file and noted:

1. Alexa calls our API on BASDEV. This works.
2. Our server then calls a live server, BAS, to obtain task information.

The sequence of events (and the error) is this:

BASDEV:
`Alexa enquiry about MostImportant`

BASDEV:
`VoiceApi GET command: https://bas.fintechsystems.net/api/v1/tasks/most-important`

```bash
cURL error 60: SSL: no alternative certificate subject name matches target host name 'bas.fintechsystems.net' (see https://curl.haxx.se/libcurl/c/libcurl-errors.html) for https://bas.fintechsystems.net/api/v1/tasks/most-important {"exception":"[object] (GuzzleHttp\\Exception\\RequestException(code: 0): cURL error 60: SSL: no alternative certificate subject name matches target host name 'bas.fintechsystems.net' (see https://curl.haxx.se/libcurl/c/libcurl-errors.html) for https://bas.fintechsystems.net/api/v1/tasks/most-important at /home/forge/basdev.fintechsystems.net/vendor/guzzlehttp/guzzle/src/Handler/CurlFactory.php:211)
```

What's interesting at this point is BAS has never run with an SSL certificate because of prior issue installing one, and there might be implications when you install one. But we did it anyway and it the installation of Let's Encrypt via Forge seemed to work.

REVERT:
Installing a certificate on BAS doesnt work, it redirects to the default site on the Forge server.

Next steps:

Investigate post to BAS

We were able to post to the API endpoint without a key and saw that it worked so we could deduce the round trip is working.

Fix:

Add this URL. The default existed, but had HTTPS instead of HTTP

```
VOICE_API_URL="http://bas.fintechsystems.net/api/v1/"
```

Now the response is:

`stdClass Object`

`The system responded: The most important task is Online Shop Signup Procedure.`



# Alexa Developer Console

## Build Tab

### Endpoint menu

This has the API endpoint, in our case:
https://basdev.fintechsystems.net/api/v1/alexa

## Test Tab

The test menu allows you to type stuff which is much quicker than talking to Alexa.
This is great when you need to quickly find a problem.

https://developer.amazon.com/alexa/console/ask/test/amzn1.ask.skill.d9740e82-9641-4528-b427-4756d16d7256/development/en_US/

Try this:

alexa ask the system what's the most important task

Analytics:
https://developer.amazon.com/alexa/console/ask/measure/amzn1.ask.skill.d9740e82-9641-4528-b427-4756d16d7256/development/en_US

Useful to see if things are getting to Alexa.

