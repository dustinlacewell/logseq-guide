Reference: https://github.com/logseq/logseq/tree/defclass/electron#set-up-development-environment

## Prerequisites
1. Make sure all requirements listed at https://github.com/logseq/logseq/tree/defclass/electron#1-requirements are met

2. Install PostgreSQL create a DB called "logseq" and a user with SUPERUSER permissions 

4. Install nginx & Certbot on your sever 

5. Create GitHub application as discribed in https://docs.github.com/en/free-pro-team@latest/developers/apps/creating-a-github-app
  in the App set the Homepage URL to https://yourdomain.com - call back URL to https://yourdomain.com/auth/github
  Create a "client secret" & a "Private Key" for your app
  in app permissions set "Contents" access to "read and Write"
  Copy from your app
    * App ID
    * Client ID
    * Client Secret
  upload the pem file that was automatically downloaded when you created your app "Private Key" to your sever


## Running LogSeq 
1. Clone the logseq repo git clone `https://github.com/logseq/logseq`
2. Download backend.jar from https://github.com/logseq/logseq/releases/tag/0.0.1 and place it in `~/logsec/resources`
3. In [~/logseq/src/main/frontend/config.cljs](https://github.com/logseq/logseq/blob/master/src/main/frontend/config.cljs)
   line 18 replace
```javascript
(def website
  (if dev?
    "http://localhost:3000"
    (util/format "https://%s.com" app-name)))
```
by
```javascript
(def website
  (if dev?
    "http://localhost:3000"
    (util/format "https://yourdomain.com" app-name)))
```
   line 38 replace 
```javascript
(def github-app-name (if dev? GITHUB_APP_NAME "logseq"))
```
by
```javascript
(def github-app-name (if dev? GITHUB_APP_NAME "yourappname"))
```

4. Set the envrionment variables
```shell
export GITHUB_REDIRECT_URI=https://yourdomain.com/auth/github
export ENVIRONMENT="production"
export JWT_SECRET="somepass"
export COOKIE_SECRET="54676c2c5324279480a6ee1af0792588"
export GITHUB_APP2_NAME="yourappname"
export GITHUB_APP2_ID="appID"
export GITHUB_APP2_KEY="AppClientID"
export GITHUB_APP2_SECRET="AppClientSecret"
export GITHUB_APP_PEM="/path/to/github/pem/file/yourappname.2021-01-19.private-key.pem"
export LOG_PATH="/tmp/logseq"
export DATABASE_URL=postgres://username:password@localhost:5432/logseq
export PG_USERNAME="usernmae"
export PG_PASSWORD="password"
export WEBSITE_URL=https://yourdomain.com/
export COOKIE_DOMAIN="yourdomain.com"
```

5. switch to `~/logsec` and run `yarn` then, compile using `yarn release`
6. run from `~/logsec` using `java -Duser.timezone=UTC -jar ./static/logseq.jar`

## Publishing LogSeq
Now we need to "server" your self hosted logseq to the internet to do that

1. Create 2 DNS records  `domain.com`  & `asset.domain.com` & get a certificate for each using [[Certbot]]
2. configure Nginx to 
- rewrite 'asset.logseq.com' to 'asset.domain.com';
- rewrite 'logseq.com' to 'domain.com';
- reverse proxy requests to  'domain.com' to 'logseq.jar' (http://localhost:3000)
- serve 'asset.domain.com' to `/path/to/logseq/public/`

A dummy copy of the ngix.conf file can be found [here](./ngnix.conf)
