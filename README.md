[![Netlify Status](https://api.netlify.com/api/v1/badges/45274e25-8717-4a51-8650-4cabb838aea5/deploy-status)](https://app.netlify.com/sites/vue-drupal-todo/deploys)


# Drupal

It doesn't take much time configuring Drupal, but understanding nuances of REST requests and creating tests is very time-consuming. So I've tried to write important this here, but in case I've missed something, there are links below each section.



## Content Type

Todo app needs to store it in Drupal. For that I've created **Todo** `todo` content type and attached **Completed** `field_completed` field with type <u>Boolean</u>. I even removed **Body** `body` field.



## Installation & configuration

I've decided to use OAuth 2.0 as an authentication method using a Simple OAuth module and built-in modules for REST. There are detailed installation instructions on modules official page. Here are the steps:

1. Enable JSON:API module and also install the Simple OAuth module using the following command and enable it.

```sh
composer require drupal/simple_oauth
```

2. Generate keys using the following command and store them one folder up from where Drupal is installed.

```sh
openssl genrsa -out private.key 2048
openssl rsa -in private.key -pubout > public.key
```

3. Save the path to your keys in: `/admin/config/people/simple_oauth`. If you followed my instruction, then the paths should be `../public.key` and `../private.key`. Also setting token expiration time from 300 to 86400 is a good idea.
4. Create a Client Application by going to: `/admin/config/services/consumer/add`. Remember what you've set in `New Secret` field value and `UUID` value, which is shown after saving. These values will be used in the Vue app.
5. To make REST endpoints (POST, PATCH, DELETE) work, go to `/admin/config/services/jsonapi` and select **Accept all JSON:API create, read, update, and delete operations.**. Make sure that user has permissions to do just that.

After this, Drupal should be ready to go.

### Links

- https://www.drupal.org/project/simple_oauth



## CORS

This is an important issue I've encountered. If Vue app and Drupal REST endpoints are hosted in different domains, you need to enable CORS in Drupal. Here is how you do it:

1. Copy `sites/default/default.services.yml` to `sites/default/services.yml`.
2. Edit the `sites/default/services.yml` to look like this: https://gist.github.com/elaman/9bda6ba947fecd8569c655282439a776
3. Flush all caches.

### Links

- https://medium.com/@emerico/cors-cross-origin-resource-sharing-in-drupal-8-19778cf2838a



## REST Authentication

Since we are using OAuth 2.0 as our authentication method, the only REST endpoint for auth we will use is `/oauth/token`. This endpoint can be used to login and to refresh the token.

#### Login

To use `/oauth/token` endpoint for login you must do `POST` request, with headers `Content-Type: application/x-www-form-urlencoded` and request body must contain following keys:

- `grant_type` with value `password`
- `username` and `password`, which user that will be logging it.
- `client_id` and `client_secret`, which are values from step 4 in Drupal / Installation & configuration section.

This will return `access_token` and `refresh_token`. `access_token` should be used for all further HTTP request as `Bearer {access_token}`.

#### Refresh

To use `/oauth/token` endpoint for a token refresh you must do `POST` request, same as for Login, but two changes:

- No `username` and `password` in the request body.
- Add `refresh_token` stored from the Login request in the request body.

### Links

- https://dev.acquia.com/blog/decoupled-drupal-authentication-oauth-20



## REST API

JSON:API has only two requirements:

- All the requests need authentication, which is `Bearer {access_token}` in the request header.
- All requests must have `Content-Type: application/vnd.api+json` in the header.

Please look at the link bellow for more detailed descriptions of GET, POST, PATCH and DELETE requests.

### Links

- https://www.drupal.org/docs/8/modules/jsonapi/get-post-patch-and-delete

# Vue

I assume you know how Vue, Vuex, and Axios works, so the rest will be explained in code comments. Instead, I'm going to describe the configuration and building of existing code.

## Configure & Build

1. Clone the git repository I've provided and run `yarn install`.
2. Create `.env.local` file in the folder.
3. Set `VUE_APP_API_URL` as your Drupal installation URL.
4. Set `VUE_APP_CLIENT_ID` as `UUID` value from step 4 in Drupal / Authentication & Rest section.
5. Set `VUE_APP_CLIENT_SECRET` as `New Secret` value from step 4 in Drupal / Installation & configuration section.
6. Run `yarn serve` or `yarn build`.
