# MailSlurp GraphQL Email API

MailSlurp has a powerful free GraphQL email API that lets you fetch and read emails with GraphQL in React, Apollo, iOS, Android or server to server. You can create disposable test email accounts on demand or permanent custom domains. Send and receive emails and attachments from any client that uses GraphQL.

If you are unfamiliar with GraphQL don't worry -  MailSlurp has a traditional [REST API](https://www.mailslurp.com/docs/api/) and [compiled SDK Clients](https://www.mailslurp.com/developers/) in many languages.

## Quick links 
- API server root is [https://graph.mailslurp.com/](https://graph.mailslurp.com/)
- See the Github for [graphql schema definitions](https://github.com/mailslurp/mailslurp-client-graphql/blob/master/mailslurp/)
- Go to the [MailSlurp GraphQL Playground](https://graphql.mailslurp.com) to explore yourself
- Read the [graphQL getting started guide](https://www.mailslurp.com/guides/graphql-email-api/) for more help

## Getting started
MailSlurp's API is free but requires an API_KEY. [Obtain an API_KEY](https://app.mailslurp.com) at app.mailslurp.com first. **Include you api key in an `x-api-key` header** to authenticate you requests.

### HTTP requests
You can query the GraphQL api using any client. Here is an example using Curl in a terminal:

```bash
curl -lX POST https://graphql.mailslurp.com -H 'x-api-key:YOUR_API_KEY' -H "Content-Type: application/json" -d '{ "query": "{ inboxes { totalElements } }"}'
```

For simpler usage please see the Javascript examples.

### Javascript usage
Add the `graphql-request` library to your node project. 

```bash
npm install --save graphql-request
```

Create a client using the `https://graphql.mailslurp.com` root endpoint and pass your api key as a header. 

```javascript
import {GraphQLClient} from "graphql-request";

export function getClient(apiKey) {
  if (!apiKey) {
    throw "Please set missing API_KEY"
  }
  // create a new graphql-request client using the MailSlurp graphql endpoint
  // and passing a headers map including your MailSlurp API Key using "x-api-key" header
  return new GraphQLClient('https://graphql.mailslurp.com', {
    headers: {
      'x-api-key': apiKey,
    },
  });
}
```

## Create email inbox
Using the client function we created above we can create a new email address using the `createInbox` mutation.

Note: the examples that follow use the AVA test framework to demonstrate usage - you can use any frameworks you wish with GraphQL.

```javascript
import test from 'ava';
import {gql} from "graphql-request";
import {getClient} from "./index.mjs";

const apiKey = process.env.API_KEY;

test('can create inbox', async t => {
    const {createInbox} = await getClient(apiKey).request(gql`
        mutation {
            createInbox {
                id
                emailAddress
            }
        }
    `);
    t.is(!!createInbox.id, true);
    t.is(createInbox.emailAddress.indexOf('@mailslurp') > -1, true);
});
```

## Send email in GraphQL

Use the `sendEmail` mutation to send a real email from your inbox.

```javascript
test('can send an email', async t => {
    // create an inbox
    const {createInbox} = await getClient(apiKey).request(gql`
        mutation {
            createInbox {
                id
                emailAddress
            }
        }
    `);
    // send an email using mutation
    const {sendEmail} = await getClient(apiKey).request(gql`
        mutation {
            sendEmail(fromInboxId: "${createInbox.id}", to: ["${createInbox.emailAddress}"], subject: "Test subject") {
                from
                to
                subject
            }
        }
    `);
    t.is(sendEmail.from, createInbox.emailAddress)
    t.is(sendEmail.subject, "Test subject")
    t.is(sendEmail.to.indexOf(createInbox.emailAddress) > -1, true)
});
```

## Receive and read email using GraphQL
You can query emails that you know already exist using the `email` and `emails` queries. To block until an expected email either arrives or is found in an inbox use the `waitForX` queries:

```javascript
test('can send an email and receive the contents', async t => {
    // create an inbox
    const {createInbox} = await getClient(apiKey).request(gql`
        mutation {
            createInbox {
                id
                emailAddress
            }
        }
    `);
    // send an email using mutation
    const {sendEmail} = await getClient(apiKey).request(gql`
        mutation {
            sendEmail(fromInboxId: "${createInbox.id}", to: ["${createInbox.emailAddress}"], subject: "Test subject", body: "Hello") {
                from
                to
                subject
            }
        }
    `);

    // wait for the email to arrive
    const {waitForLatestEmail} = await getClient(apiKey).request(gql`
        query {
            waitForLatestEmail(inboxId: "${createInbox.id}", timeout: 60000, unreadOnly: true) {
                id
                from
                to
                subject
                body
            }
        }
    `);
    t.is(waitForLatestEmail.body.indexOf("Hello") > -1 , true)
    t.is(waitForLatestEmail.subject.indexOf(sendEmail.subject) > -1 , true)
    t.is(waitForLatestEmail.from.indexOf(sendEmail.from) > -1 , true)

    // delete the email afterwards
       const {deleteEmail} = await getClient(apiKey).request(gql`
        mutation {
            deleteEmail(emailId: "${waitForLatestEmail.id}")
        }
    `);
    t.is(deleteEmail === "deleted", true)

});
```
