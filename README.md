# Apollo OAuth Token Refresh Link

## Purpose
An Apollo Link that performs OAuth access token renewal when the token is expired

## Installation

`npm install apollo-link-oauth-token-refresh --save`

## Usage
Token Refresh Link is `non-terminating` link, which means that this link shouldn't be the last link in the composed chain.

```js
import { TokenRefreshLink } from "apollo-link-oauth-token-refresh";

const link = new TokenRefreshLink({
  isTokenValidOrUndefined: () => boolean,
  fetchAccessToken: () => Promise<Response>,
  handleResponse: (operation,) => response,
  handleError?: (err: Error) => void,
});
```

## Options
Token Refresh Link takes an object with four options on it to customize the behavior of the link

|name|value|explanation|
|---|---|---|
|isTokenValidOrUndefined|(...args: any[]) => boolean|Indicates the current state of access token expiration. If token not yet expired or user doesn't have a token (guest) `true` should be returned|
|fetchAccessToken|(...args: any[]) => Promise<Response>|Function covers fetch call with request fresh access token|
|handleResponse|(operation) => response. Used to handle the response status & extract your token|
|handleError?|(err: Error) => void|Token fetch error callback. Allows to run additional actions like logout. Don't forget to handle Error if you are using this option|

## Example
```js
import { TokenRefreshLink } from 'apollo-link-token-refresh';

link: ApolloLink.from([
  new TokenRefreshLink({
    isTokenValidOrUndefined: () => !isTokenExpired() || typeof getAccessToken() !== 'string',
    fetchAccessToken: () => {
      return fetch(getEndpoint('getAccessTokenPath'), {
        method: 'POST',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: `grant_type=refresh_token&client_id=${getOauthClientId()}&client_secret=${getOauthClientSecret()}&refresh_token=${getRefreshToken()}`
      });
    },
    handleResponse: () => (response) => {
      // here you can handle response & save the token
      if (response.status === 401) {
        console.warn('token has been revoked');
        user.logout();
      } else {
        return response.json().then(data => saveToken(data));
      }
    },
    handleError: err => {
    	// full control over handling token fetch Error
    	console.warn('Your refresh token is invalid. Try to relogin');
    	console.error(err);

    	// your custom action here
    	user.logout();
    }
  }),
  errorLink,
  requestLink,
  ...
])
```

## Context
The Token Refresh Link does not use the context for anything
