# Integrating Plaid into Appian POC

- Use an Appian "web content component" to embed a simple static html page into an iframe within an Appian interface.

- For convenience and to demonstrate how simple it is to create the static javascript page, this POC will use an external page residing on github pages.   You can run this static javascript page stand-alone as well, it is not required to be embedded in an iframe:   [https://dlhpp.github.io/plaidappianpoc/index.html](https://dlhpp.github.io/plaidappianpoc/index.html), or for a slightly more dynamic verion:  https://dlhpp.github.io/plaidappianpoc/plaidIntegration.html?poc=Y&align=left&tok=

- Here's the Appian interface that embeds the above external page into an iframe:
    ```
    {
    a!sectionLayout(
        label: "Test WebComponent",
        contents: {
        =a!webContentField(
            source: "https://dlhpp.github.io/plaidappianpoc/index.html",
            height: "TALL",
            showBorder: false,
            altText: "DLH Test WebContentComponent"
        )
        }
    )
    }
    ```

- This github page is pure static html javascript which working in tandem with web services is enough to create plaid "items".

- The static github page initiates the Plaid "institution select" modal window which is under Plaid's control.   When the user selects a bank and consents for us to connect to his bank, the modal window returns a public_token to the static github page.

- The public_token is one-time-use and only good for 30 minutes.  Upon receiving the public_token, the static page must call our own internal web service to exchange the public_token for an access_token/item_id within 30 minutes and we must persist the access_token/item_id to a database associated with the user.

- Why must we use our own internal web service for exchanging public_token as well as all other data retrieval calls to Plaid?   Because making the exchange call as well as all data retrieval calls to Plaid requires a client_id and secret which must be private to us and cannot be exposed publicly, meaning it cannot be present in any client-side html or javascript.

- Since the static github page is embedded in an iframe in the Appian interface, there is no communication between it and the rest of the appian interface.   Appian has no idea if the user is interacting with the iframe or if the user is done linking bank accounts.   The only way to communicate between the iframe and Appian is thru a persistent store that Appian can access - the database.  The static page will call web services which will store info in a database.

- Since the Appian interface doesn't know when the user is done linking accounts, the appian interface will need some buttons outside the iframe so the user can click to say, "I'm done linking my bank accounts."   Appian can then query a database to see if some bank accounts have been linked and then move on with the process flow or present an error message asking the user if they want to link more bank accounts.

- Our internal web services will need to be reachable from Plaid because some of the calls, such as retrieving transactions and retrieving assets are asynchronous in nature.   We will make a restful call to Plaid to say we want transactions or assets and Plaid will call us back at a later time at the webhook url we configure to give us the data.   The "item_id" will be included with the webhook call from Plaid and that's how we know where to put the data that Plaid sends to us.
