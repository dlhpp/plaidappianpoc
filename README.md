# Integrating Plaid into Appian POC

- Uses an Appian "web content component".   

- Appian web content component embeds an external page as an iframe into an Appian interface.

- For this POC, the external page comes from github pages here:   [https://dlhpp.github.io/plaidappianpoc/index.html](https://dlhpp.github.io/plaidappianpoc/index.html)

- This github page is pure static html javascript which, when combined with a web service and database, is enough to create and manage plaid "items".

- The static github page initiates the Plaid "institution select" modal window which is under Plaid's control.   When the user selects a bank and consents for us to connect to his bank, the modal window returns a public_token to the static github page.

- The public_token is one-time-use and only good for 30 minutes.  We must call our own internal web service to exchange the public_token for an access_token/item_id within 30 minutes and we must persist the access_token/item_id to a database associated with the user.

- Why must we use our own internal web service for exchanging public_token as well as all other data retrieval calls to Plaid?   Because making the exchange call and all data retrieval calls to Plaid requires a client_id and secret which must be private to us and cannot be exposed publicly, meaning it cannot be present in any client-side html or javascript.

- Since the static github page is embedded in a iframe in the Appian interface, there is no communication between it and the rest of the appian interface.   Appian has no idea if the user is interacting with the iframe or if the user is done linking bank accounts.   The only way to communicate between the iframe and Appian is thru a persistent store that Appian can access - the database.  The static page will call web services which will store info in a database.

- Since the Appian interface doesn't know when the user is done linking accounts, the appian interface will need some buttons outside the iframe so the user can click to say, "I'm done linking my bank accounts."   Appian can then query a database to see if some bank accounts have been linked and then move on with the process flow or present an error message asking the user if they want to link more bank accounts.
