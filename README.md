# MailSlurp GraphQL Email API

MailSlurp has a powerful free GraphQL email API that let's you fetch and read emails with GraphQL in React, Apollo, iOS, Android or server to server. If you are unfamiliar with GraphQL don't worry -  MailSlurp has a traditional [REST API](https://www.mailslurp.com/docs/api/) and [compiled SDK Clients](https://www.mailslurp.com/developers/) in many languages.

```bash
curl -lX POST https://graphql.mailslurp.com -H 'x-api-key:YOUR_API_KEY' -H "Content-Type: application/json" -d '{ "query": "{ inboxes { totalElements } }"}'
```

See the github for [graphql schema definitions](https://github.com/mailslurp/mailslurp-client-graphql/blob/master/mailslurp/) or go to the [MailSlurp GraphQL Playground](https://graphql.mailslurp.com) to explore yourself.

## Getting started
Please see the main [guide for using MailSlurp with GraphQL](https://www.mailslurp.com/guides/graphql-email-api/)