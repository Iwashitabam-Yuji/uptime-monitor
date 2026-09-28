# uptime-monitor

GitHub Actionsによる外形監視です。5分ごとにHTTPでアクセスして、サイトが落ちていないかを見ています。

サーバー自体は生きていてもサイトが5xxやタイムアウトになる状態を拾うのが目的で、AWSのCloudWatchやLightsailの組み込みアラームでは見えない層を補っています。

## 監視しているサイト

| サイト | URL |
|---|---|
| value.celm.co.jp | https://value.celm.co.jp/ |
| ポラリスホテルズ＆リゾーツ | https://www.polaris-hotels.com/ |
| FEEL THE LOCAL | https://feel-the-local.polaris-hotels.com/ |

## しくみ

- 5分ごとに `curl` でステータスコードを見ます（`.github/workflows/uptime.yml`）
- 2xxと3xxを正常とみなします。リダイレクトは追従します
- 一時的なブリップでの誤報を避けるため、15秒間隔で最大3回リトライします。3回とも異常なら落ちたと判定します
- 判定はサイトごとに独立しています。1サイトが落ちても、ほかのサイトの監視は続きます

## 通知

- リポジトリのSecretsに `ALERT_WEBHOOK`（SlackのIncoming Webhook）を入れると、そこへ通知します
- 未設定の場合は、GitHubからのワークフロー失敗メールで届きます

## 監視先を増やすには

`.github/workflows/uptime.yml` の `matrix.site` に、1ブロック足すだけです。

```yaml
- key: example
  name: サイト名
  url: https://example.com/
  allow_401: "no"
```

`allow_401` を `"yes"` にすると、401（Basic認証）も正常とみなします。**公開前でBasic認証をかけているサイト用の暫定設定です。** 認証を外したら `"no"` に戻してください。戻し忘れると、誤って認証がかかり直しても検知できません。

## 制約

- GitHub Actionsのスケジュールは、混雑時に5〜15分ほど遅れたり、まれに飛んだりします。秒単位の精度は期待できません
- ステータスコードだけを見ているので、200が返るが中身が壊れている（CSSが当たっていない、記事が消えているなど）状態は検知できません
