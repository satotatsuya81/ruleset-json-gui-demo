# JSON管理・GUI手動適用の検証

GitHub rulesetの定義をJSONとしてGitで管理し、レビュー済みの内容をGitHubのGUIへ手動で反映する手順を確認するためのリポジトリです。

CIによる同期処理はありません。JSONをマージしてもGitHub側のrulesetは変わりません。

## 検証する内容

- JSONをGUIからインポートして、developを対象とするrulesetを作成する
- JSONの変更をPRで確認し、マージする
- マージ後もGitHub側の設定が変わらないことを確認する
- GUIで既存rulesetを編集し、JSONと同じ内容にする

個人リポジトリで実行できるよう、通常の「PR必須」「承認人数」を使います。組織のTeamを指定するRequired reviewers、ファイルごとの承認条件、CODEOWNERSとの組み合わせは、この検証には含めません。

## ファイルとブランチ

| 対象 | 内容 |
| --- | --- |
| [rulesets/develop.json](rulesets/develop.json) | 管理対象のruleset全体。初期状態はPR必須、承認人数0 |
| [examples/develop-approval-1.json](examples/develop-approval-1.json) | 承認人数を1に変更した比較用サンプル |
| [sample/application.txt](sample/application.txt) | 動作確認用の架空のファイル |
| codex/change-review-policy | 管理対象JSONの承認人数を0から1へ変更するブランチ |
| codex/sample-change | 架空のファイルを変更するブランチ |

変更する値は `rules[0].parameters.required_approving_review_count` です。ruleset名と対象ブランチは変更しません。

## 1. 初回のrulesetをGUIから作成する

1. [初期JSONをRawで開く](https://raw.githubusercontent.com/satotatsuya81/ruleset-json-gui-demo/develop/rulesets/develop.json)からJSONを保存する。承認人数が0であることを確認する。
2. [Rulesets設定](https://github.com/satotatsuya81/ruleset-json-gui-demo/settings/rules)を開く。
3. **New ruleset → Import a ruleset** を選び、保存したJSONを読み込む。
4. 以下の内容を確認し、**Create** を押す。

| GUIで確認する項目 | 値 |
| --- | --- |
| Ruleset name | demo-develop-review |
| Enforcement status | Active |
| Target branches | develop |
| Require a pull request before merging | 有効 |
| Required approvals | 0 |
| Bypass list | 空 |

JSONファイル全体をインポートします。レビュー条件だけの配列を渡す形式ではありません。

## 2. JSONの変更をPRで確認し、マージする

1. [設定変更の比較画面](https://github.com/satotatsuya81/ruleset-json-gui-demo/compare/develop...codex/change-review-policy?expand=1)を開き、develop向けのPRを作成する。
2. `rulesets/develop.json` の承認人数が0から1になる差分を確認する。
3. PRをマージする。初期rulesetは承認人数0なので、別のレビュアーのApproveは不要。
4. [Rulesets設定](https://github.com/satotatsuya81/ruleset-json-gui-demo/settings/rules)を開き、**Required approvalsがまだ0であること**を確認する。

この時点で、リポジトリ内のJSONは1、GitHub側の設定は0です。GUIへの反映はまだ完了していません。

## 3. GUIで既存rulesetを更新する

1. Rulesets設定で `demo-develop-review` を開く。
2. **Required approvals** を0から1に変更する。
3. **Save changes** を押す。
4. rulesetを開き直し、JSONと同じ対象ブランチ、承認人数、その他の条件になっていることを確認する。
5. 設定変更PRに、反映したJSONのコミット、rulesetのURL、反映した日時を記録する。

公式に案内されている **Import a ruleset** は新規作成の手順です。既存rulesetの更新では、今回のように編集画面から変更して保存します。同じ名前のrulesetを追加作成する操作は、この手順では行いません。

## 4. 承認条件が変わったことを確認する

1. [サンプル変更の比較画面](https://github.com/satotatsuya81/ruleset-json-gui-demo/compare/develop...codex/sample-change?expand=1)を開き、develop向けのPRを作成する。
2. Approveがない状態では承認条件が未達になることを確認する。

PRの作者は自分のPRをApproveできません。承認人数1の状態で通常のマージまで試す場合は、別アカウントにwrite権限を付けてレビューを依頼する必要があります。

## 5. 検証後に元へ戻す

1. GUIで **Required approvals** を0へ戻して保存する。
2. `rulesets/develop.json` の承認人数も0へ戻すPRを作り、マージする。
3. JSONとGUIの設定がどちらも0であることを確認し、サンプルPRを必要に応じてマージまたはクローズする。

## 運用で記録すること

JSONをレビュー・マージしただけでは適用完了になりません。適用担当者がGUIに反映し、同じ設定になったことを確認してから、次の項目をPRに記録します。

| 記録項目 | 記入する内容 |
| --- | --- |
| 管理するJSON | rulesets/develop.json |
| 適用した定義 | developへマージされたコミットSHA |
| 適用先 | rulesetのURLまたはID |
| 適用した人・日時 | 操作者と適用した日時 |
| 確認した内容 | 対象ブランチ、承認人数、設定の一致、動作確認結果 |

## 権限とこの検証の範囲

公開リポジトリなのでGitHub Freeでもrulesetを使用できます。GUIの操作にはリポジトリのadmin権限、または「リポジトリルールの編集」権限を持つカスタムロールが必要です。この個人リポジトリの所有者は設定を操作できます。

この手動運用には、CI用のGitHub AppやAdministration: writeトークン、秘密鍵を登録する必要はありません。

この検証で確認できるのは、JSONの管理、GUIインポート、手動更新、適用漏れの確認、通常の承認条件の動作です。Teamを指定するRequired reviewersの動作は、対象Teamが所属する組織のリポジトリで別途確認します。

社内コード、社内のパスやTeam ID、認証情報は含めていません。

## 参考資料

- [GitHub公式: リポジトリのルールセットの管理](https://docs.github.com/ja/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository)
- [GitHub公式: ruleset-recipes](https://github.com/github/ruleset-recipes)
- [GitHub公式: ルールセットの作成と利用できるプラン](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository)
- [GitHub公式: 組織のTeam](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams)
