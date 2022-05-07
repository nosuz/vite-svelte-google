This a copy of [Quick Start sample of Google Calendar API](https://developers.google.com/calendar/api/quickstart/js) but implimented on Vite and Svelete framework.

これは、Google Calendar API のクイックスタートを Vite と Svelte 環境で動くようにした習作です。

Client Id is read from the `client_secret.json` file under `src` directory.

When you made your OAuth 2.0 client on the Google Developer Console

1. Save it as JSON file.
2. Rename the file to `client_secret.json`.
3. Move it under the `src` directory.

Client ID は、`src`ディレクトリーの`client_secret.json`から読み込みます。

Google Developer Console で OAuth 2.0 クライアントを作成したら、

1. クライアント情報を JSON ファイルに保存します。
2. そのファイル名を`client_secret.json`に変えます。
3. そして、`src`ディレクトリーに移動させます。

## Issue

Sign out makes CORS error. This error is same error as reported on the stackoverflow.

[CORS issue only when revoking Token by calling (Google Identity Services) GIS's google.accounts.oauth2.revoke](https://stackoverflow.com/questions/71532935/cors-issue-only-when-revoking-token-by-calling-google-identity-services-giss)

For now, there is no workaround for this issue.
