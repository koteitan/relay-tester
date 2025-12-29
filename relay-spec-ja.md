# Nostr リレー仕様

## リレーランキング (2025/11/10)

出典: [nostr.watch](https://nostr.watch/relays/software)

| 名前 | リレー数 | バージョン数 | シェア | ベース |
|------|--------|----------|-------|--------|
| hoytech/strfry | 237 | 35 | 26.9% | - |
| ~gheartsfield/nostr-rs-relay | 205 | 10 | 23.3% | - |
| bitvora/haven | 71 | 15 | 8.1% | khatru |
| cameri/nostream | 55 | 8 | 6.3% | - |
| bitvora/wot-relay | 25 | 5 | 2.8% | khatru |
| fiatjaf/khatru | 18 | 3 | 2.0% | - |

## limit パラメータの動作

このドキュメントでは、異なるNostrリレー実装における `limit` フィルターパラメータの動作を比較します。

### 概要テーブル

| リレー実装 | デフォルト最大limit | 設定パラメータ | 動作 |
|-----------|-------------------|----------------|------|
| strfry | [500](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L91) | `relay.maxFilterLimit` | フィルターごとに min(client_limit, 500) 件のイベントを返す |
| nostream | [5000](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L162) | `limits.client.subscription.maxLimit` | サブスクリプションごとに min(client_limit, 5000) 件のイベントを返す |
| nostr-rs-relay | 明示的な制限なし | デフォルト設定に見つからず | limit未指定時は全てのマッチするイベントを返す |
| khatru (フレームワーク) | デフォルト制限なし | 該当なし | 実装依存、組み込みの最大limitなし |
| haven (khatruベース) | 制限なし | 未設定 | eventstoreを通じて全てのマッチするイベントを返す |
| wot-relay (khatruベース) | 制限なし | 未設定 | eventstoreを通じて全てのマッチするイベントを返す |

## レート制限

このテーブルは、各リレーがクライアントリクエストに対して適用するレート制限を示します。

| リレー | タイプ | 最大サブスクリプション数 | イベント送信レート | フィルター/REQレート | 接続レート | 備考 |
|-------|------|----------------------|-------------------|---------------------|------------|------|
| strfry | - | [20](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L89) | デフォルトでは未設定 | 未設定 | 未設定 | 全ての制限は設定可能 |
| nostream | Kind [0](https://github.com/nostr-protocol/nips/blob/master/01.md),[3](https://github.com/nostr-protocol/nips/blob/master/02.md),[40](https://github.com/nostr-protocol/nips/blob/master/28.md),[41](https://github.com/nostr-protocol/nips/blob/master/28.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [6 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L108-L115) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) および [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | メタデータ、コンタクト、チャンネルイベント |
| nostream | Kind [1](https://github.com/nostr-protocol/nips/blob/master/01.md),[2](https://github.com/nostr-protocol/nips/blob/master/01.md),[4](https://github.com/nostr-protocol/nips/blob/master/04.md),[42](https://github.com/nostr-protocol/nips/blob/master/28.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [12 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L116-L123) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) および [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | ノート、DM、チャンネルメッセージ |
| nostream | Kind [5](https://github.com/nostr-protocol/nips/blob/master/09.md)-[7](https://github.com/nostr-protocol/nips/blob/master/25.md),[43](https://github.com/nostr-protocol/nips/blob/master/28.md)-[49](https://github.com/nostr-protocol/nips/blob/master/49.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [30 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L124-L131) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) および [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | 削除、リアクション、チャンネルイベント |
| nostream | Kind [10000-19999,30000-39999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [24 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L132-L141) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) および [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | 置換可能イベント |
| nostream | Kind [20000-29999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [60 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L142-L147) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) および [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | 一時イベント |
| nostream | 全イベント | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [720 events/hour](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L148-L150) 全体制限 | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) および [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | 全kindの合計 |
| nostr-rs-relay | - | 制限なし | 設定可能 (デフォルト: 無制限) | 設定可能 (デフォルト: 無制限) | 未設定 | オプション: [messages_per_sec](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L115), [subscriptions_per_min](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L121) |
| khatru | - | 制限なし | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L12) (max 10 tokens) | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L17) (max 100 tokens) | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L21) (max 100 tokens) | フレームワーク。[`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9)経由 |
| haven | Private | 制限なし | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L61) (max 100 tokens) | 個別設定なし | [3 conn/5min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L66) | Khatruベース |
| haven | Chat | 制限なし | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L72) (max 100 tokens) | 個別設定なし | [3 conn/3min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L77) | Khatruベース |
| haven | Inbox | 制限なし | [10 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L83) (max 20 tokens) | 個別設定なし | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L88) | Khatruベース |
| haven | Outbox | 制限なし | [10 events/60min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L94) (max 100 tokens) | 個別設定なし | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L99) | Khatruベース |
| wot-relay | - | 制限なし | [5 events/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L131) (max 30 tokens) | [5 filters/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L137) (max 30 tokens) | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L141) (max 30 tokens) | Khatruベース。khatruデフォルトより厳格 |

**重要な注意事項:**
- **最大サブスクリプション数**: 接続ごとの最大同時REQサブスクリプション数。「制限なし」はリレーが上限を強制しないことを意味する（フレームワーク依存）
- **トークンバケットアルゴリズム**: ほとんどのリレーはトークンが時間とともに補充されるトークンバケットレート制限を使用
- **レート制限 ≠ 結果制限**: レート制限はリクエスト頻度を制御し、リクエストごとに返されるイベント数ではない
- **IP単位 vs 接続単位**: ほとんどの実装はIPアドレス単位で制限を適用

---

## 時間ベースの制限

### イベント送信時刻の検証

このテーブルは、クライアントが新しいイベントを送信する際に、リレーが `created_at` タイムスタンプをどのように検証するかを示します。

| リレー | 最大未来オフセット | 最大過去オフセット | 備考 |
|-------|------------------|------------------|------|
| strfry | [+900秒 (15分)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L24) | [-94,608,000秒 (約3年)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L27) | この範囲外のイベントを拒否 |
| nostream | [+900秒 (15分)](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L90) | 制限なし | 未来のイベントのみ拒否 |
| nostr-rs-relay | [+1,800秒 (30分)](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L105) | 制限なし | 未来のイベントのみ拒否 |
| khatru | 強制なし | 強制なし | フレームワークはデフォルトでは強制しない |
| haven | 強制なし | 強制なし | khatruの動作を継承 |
| wot-relay | 強制なし | 強制なし | khatruの動作を継承 |

**時刻検証の目的:**
- **未来オフセット制限**: クライアントが遠い未来のタイムスタンプを持つイベントを作成することを防止
- **過去オフセット制限**: 悪意のあるユーザーが非常に古いタイムスタンプでイベントを作成するバックデート攻撃を防止

### イベント保存/削除ポリシー

このテーブルは、リレーが時間の経過とともに保存されたイベントをどのように管理するかを示します。

| リレー | 一時イベント経過時間 | 一時イベント生存期間 | 通常イベント最大経過時間 | 備考 |
|-------|-------------------|-------------------|----------------------|------|
| strfry | [60秒](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L30)より古い場合拒否 | [300秒](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L33)後に自動削除 | - | Kind 20000-29999のみ |
| nostream | - | - | - | 自動削除なし |
| nostr-rs-relay | - | - | - | 自動削除なし |
| khatru | - | - | - | 実装依存 |
| haven | - | - | - | 自動削除なし |
| wot-relay | - | - | [365日](https://github.com/bitvora/wot-relay/blob/24b51de9/.env.example#L27)後に削除 | MAX_AGE_DAYSで設定可能 |

**主な違い:**
- **一時イベント** (kind 20000-29999): 長期保存を想定していない一時的なイベント
- **通常イベント**: リレーが通常無期限に保存する標準イベント
- **自動削除**: 一部のリレーはストレージを管理するために古いイベントを自動的に削除

---


## 詳細分析

### 1. strfry (C++)

**設定ファイル:** `strfry.conf`

**デフォルト設定:**
```conf
relay {
    # フィルターごとに返せる最大レコード数
    maxFilterLimit = 500
}
```

**動作:**
- クライアントが `limit: N` でイベントをリクエストした場合、strfryはフィルターごとに **min(N, 500)** 件のイベントを返す
- クライアントがlimitを指定しないか、500より大きいlimitを指定した場合、strfryは500に制限
- これはサブスクリプションごとではなく、フィルターごとの制限

**サイズ制限:**
- イベントサイズ: 65,536バイト (64 KB) - 正規化JSON
- WebSocketペイロード: 131,072バイト (128 KB)
- タグ値サイズ: 1,024バイト
- REQごとの最大フィルター数: 200

**時間制限:**
- 未来に900秒 (15分) を超えるイベントを拒否
- 94,608,000秒 (約3年) より古いイベントを拒否
- 60秒より古い一時イベントを拒否
- 一時イベント生存期間: 300秒 (5分)

**サポートNIP:** 1, 2, 4, 9, 11, 22, 28, 40, 70, 77

---

### 2. nostream (TypeScript)

**設定ファイル:** `resources/default-settings.yaml`

**デフォルト設定:**
```yaml
limits:
  client:
    subscription:
      maxSubscriptions: 10
      maxFilters: 10
      maxFilterValues: 2500
      maxSubscriptionIdLength: 256
      maxLimit: 5000
      minPrefixLength: 4
```

**動作:**
- クライアントが `limit: N` でイベントをリクエストした場合、nostreamは **min(N, 5000)** 件のイベントを返す
- `maxLimit: 5000` 設定が許可される最大limit値を制御
- 以下も強制:
  - クライアントごとの最大同時サブスクリプション数: 10
  - サブスクリプションごとの最大フィルター数: 10
  - 合計最大フィルター値: 2500

**サイズ制限:**
- ネットワークペイロード: 524,288バイト (512 KB)
- イベントコンテンツ: kind範囲0-10、40-49、11-39、50-maxで102,400バイト (100 KB)

**時間制限:**
- イベントcreated_at: 未来に最大900秒 (15分)
- イベントcreated_at: 過去に最大0秒 (時間による過去イベントの拒否なし)

**レート制限 (イベントkindごと):**
- Kind 0, 3, 40, 41: 6 events/分
- Kind 1, 2, 4, 42: 12 events/分
- Kind 5-7, 43-49: 30 events/分
- Kind 10000-19999, 30000-39999: 24 events/分 (置換可能)
- Kind 20000-29999: 60 events/分 (一時)
- 全イベント: 720 events/時

**サポートNIP:** 01, 02, 04, 09, 11, 11a, 12, 13, 15, 16, 20, 22, 28, 33, 40

**追加機能:**
- 決済プロセッサ統合 (ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL)
- イベントkindごとの詳細なレート制限

---

### 3. nostr-rs-relay (Rust)

**設定ファイル:** `config.toml`

**デフォルト設定:**
- デフォルト設定に明示的な `maxLimit` または類似のパラメータは見つからず
- 設定はレート制限、接続制限、イベントサイズ制限に焦点

**動作:**
- フィルターに `limit` が指定された場合: その値でSQL LIMIT句を適用
- `limit` が指定されていない場合: SQLクエリにLIMIT句を追加しない
- **limit未指定時は全てのマッチするイベントを返す** (潜在的に無制限)

**ソースコードの証拠:**
```rust
// src/repo/sqlite.rs:1093-1094
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

**サイズ制限 (デフォルト値):**
- イベントサイズ: 131,072バイト (128 KB)
- WebSocketメッセージ: 131,072バイト (128 KB)
- WebSocketフレーム: 131,072バイト (128 KB)

**時間制限:**
- 未来に1,800秒 (30分) を超えるイベントを拒否

**レート制限 (設定可能):**
- 秒あたりメッセージ数: 設定可能 (デフォルト: 無制限)
- 分あたりサブスクリプション数: 設定可能 (デフォルト: 無制限)

**サポートNIP:** 01, 02, 05, 09, 11, 12, 15, 16, 20, 22, 28, 33, 40, 42

**注目すべき設定オプション:**
```toml
[options]
reject_future_seconds = 1800

[limits]
# messages_per_sec = 5
# subscriptions_per_min = 0
# max_event_bytes = 131072
# max_ws_message_bytes = 131072
# max_ws_frame_bytes = 131072
```

---

### 4. khatru (Goフレームワーク)

**設定:** コードベース、設定ファイルなし

**デフォルト設定:**
- フレームワークに組み込みの最大limit強制なし
- 開発者は独自のlimitポリシーを実装する必要がある

**動作:**
- フレームワークは `LimitZero` フラグを処理して `limit: 0` のクエリをスキップ
- 実際のlimit強制はKhatruを使用するリレー実装に依存
- レート制限ヘルパーを提供するが、結果制限キャップはなし

**フレームワーク機能:**
- カスタムイベント/フィルター受け入れポリシー
- カスタムAUTHハンドラー
- プラガブルストレージバックエンド
- `policies` パッケージの組み込みポリシーヘルパー

**デフォルトサイズ制限:**
- 最大メッセージサイズ: 512,000バイト (500 KB)

**デフォルトレート制限 (`ApplySaneDefaults`経由):**
- イベントレート: 3分あたり2イベント (max 10 tokens)
- フィルターレート: 分あたり20フィルター (max 100 tokens)
- 接続レート: 5分あたり1接続 (max 100 tokens)

**ソースからの例:**
```go
// responding.go:21-24
if filter.LimitZero {
    return nil, fmt.Errorf("invalid limit 0")
}

// relay.go:45
MaxMessageSize: 512000,
```

---

### 5. haven (Go, Khatruベース)

**設定ファイル:** `.env.example`

**デフォルト設定:**
- 明示的なlimit設定は見つからず
- Khatruフレームワーク + eventstoreライブラリを使用
- データベースバックエンド: BadgerDBまたはLMDB

**動作:**
- Khatruフレームワークから動作を継承
- QueryEvents実装に `eventstore` ライブラリを使用
- `limit` 未指定時: **データベースから全てのマッチするイベントを返す**
- **結果サイズのハード制限なし** - 大規模データセットでは潜在的に危険

**ソースコードの証拠:**
```go
// init.go:134
privateRelay.QueryEvents = append(privateRelay.QueryEvents, privateDB.QueryEvents)
// github.com/fiatjaf/eventstore v0.17.1を使用
```

**特別な機能:**
- 4つのリレーを1つに (Private, Chat, Inbox, Outbox)
- Blossomメディアサーバー
- Web of Trustフィルタリング
- クラウドバックアップ (S3互換)
- BadgerDBまたはLMDBストレージ

**サイズ制限:**
- Khatruの最大メッセージサイズを継承: 512,000バイト (500 KB)

**時間制限:**
- オーナーノートインポートフェッチタイムアウト: 30秒 (デフォルト)
- タグ付きノートインポートフェッチタイムアウト: 120秒 (デフォルト)
- Web of Trustフェッチタイムアウト: 30秒 (デフォルト)

**レート制限 (リレータイプごと):**
- Private: 50 events/interval、max 100 tokens
- Chat: 50 events/interval、max 100 tokens
- Outbox: 10 events/60秒、max 100 tokens
- Inbox: 10 events/1秒、max 20 tokens

**警告:** レート制限はリクエストレートに適用され、結果サイズには適用されない。limitなしの単一リクエストでも無制限のイベントを返す可能性がある。

---

### 6. wot-relay (Go, Khatruベース)

**設定ファイル:** `.env.example`

**デフォルト設定:**
- 明示的なlimit設定は見つからず
- Khatruフレームワーク + eventstoreライブラリを使用
- データベースバックエンド: BadgerDB

**動作:**
- Khatruフレームワークから動作を継承
- QueryEvents実装に `eventstore` ライブラリを使用
- `limit` 未指定時: **データベースから全てのマッチするイベントを返す**
- **結果サイズのハード制限なし** - 大規模データセットでは潜在的に危険

**ソースコードの証拠:**
```go
// main.go:145
relay.QueryEvents = append(relay.QueryEvents, db.QueryEvents)
// github.com/fiatjaf/eventstoreを使用
```

**特別な機能:**
- Web of Trustリレー (フォローしているユーザーのノートのみ保存)
- 設定可能なWoT深度と最小フォロワー数
- オプションの他リレーからのアーカイブ同期
- オプションの経過時間ベースのノート削除

**サイズ制限:**
- Khatruの最大メッセージサイズを継承: 512,000バイト (500 KB)

**時間制限:**
- 最大イベント経過時間: 365日 (デフォルト、MAX_AGE_DAYSで設定可能)
- MAX_AGE_DAYSより古いイベントは削除

**レート制限:**
```go
policies.EventIPRateLimiter(5, time.Minute*1, 30)      // 5 events/min, max 30 tokens
policies.FilterIPRateLimiter(5, time.Minute*1, 30)     // 5 filters/min, max 30 tokens
policies.ConnectionRateLimiter(10, time.Minute*2, 30)  // 10 connections/2min, max 30 tokens
```

**設定オプション:**
```bash
REFRESH_INTERVAL_HOURS=1
MINIMUM_FOLLOWERS=5
ARCHIVAL_SYNC="FALSE"
MAX_AGE_DAYS=365
```

**警告:** レート制限はリクエストレートに適用され、結果サイズには適用されない。limitなしの単一リクエストでも無制限のイベントを返す可能性がある。

---


## 参考文献

- strfry: https://github.com/hoytech/strfry
- nostream: https://github.com/cameri/nostream
- nostr-rs-relay: https://git.sr.ht/~gheartsfield/nostr-rs-relay
- khatru: https://github.com/fiatjaf/khatru
- haven: https://github.com/bitvora/haven
- wot-relay: https://github.com/bitvora/wot-relay
- eventstore: https://github.com/fiatjaf/eventstore
- nostr.watch (リレー統計): https://nostr.watch/relays/software

---

## バージョン情報

**ドキュメントバージョン:** 1.3
**確認日:** 2025-11-10
**分析したリレーバージョン:**
- strfry: `542552ab0f5234f808c52c21772b34f6f07bec65` (2025-01-10)
- nostream: `6a8ccb491973b2470e3b332bef61ed7ea1143091` (2024-10-23)
- nostr-rs-relay: `d72af96d5f884e916cd3374dddd5550f8a45bfaf` (2025-02-23)
- khatru: `9f99b9827a6e030bbcefc48f7af68bfe7eea1a27` (2025-09-22)
- haven: `81781b36aaafe5895bb612a452b7c0f44cde6e67` (2025-10-26)
- wot-relay: `24b51de9fdd8631f4795be85983666e76f220960` (2025-06-16)
