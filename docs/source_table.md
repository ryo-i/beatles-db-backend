# BEATLES DB 元データカラム構成（Googleスプレッドシート由来）

このドキュメントは、Googleスプレッドシートからエクスポートされた元データ（JSON形式）のカラム構成・データ例・分割要否をまとめたものです。  
PostgreSQL等のデータベース設計・データ移行前の現状把握資料として利用します。

---

## カラム一覧・用途・分割要否

| カラム名       | 用途・内容                                        | 値の例                                             | 分割要否・扱い         |
|:--------------|:--------------------------------------------------|:---------------------------------------------------|:-----------------------|
| id            | レコードのユニークID                              | 1, 2, 3                                            | 通常の主キー           |
| year          | 年                                                | 1961, 1962                                         | 単一値                 |
| date          | 日付                                              | 1961/10/23                                         | 単一値                 |
| icon          | アイコン種別記号                                  | T&B, BE                                            | 単一値                 |
| category      | カテゴリ                                          | Tony & Beatles, Beatles                            | 単一値                 |
| path          | パス・分類                                        | tony-beatles, beatles                              | 単一値                 |
| artist        | アーティスト名                                    | Tony Sheridan and The Beat Brothers, The Beatles   | 単一値                 |
| country       | 国                                                | GER, UK                                            | 単一値                 |
| label         | レーベル                                          | Polydor, Parlophone                                | 単一値                 |
| format        | 形式                                              | Single, Album                                      | 単一値                 |
| title         | 作品タイトル                                      | My Bonnie, Love Me Do                              | 単一値                 |
| order         | カタログ番号等                                    | TB-S-01, BE-S-01                                   | 単一値                 |
| disc          | ディスク番号                                      | 1                                                  | 単一値                 |
| side          | レコード面                                        | A, B, 1, 2                                         | 単一値                 |
| number        | 曲順                                              | 1, 2, 3                                            | 単一値                 |
| track         | 曲名                                              | My Bonnie, Love Me Do                              | 単一値                 |
| vocal         | ボーカル担当                                      | Tony Sheridan, Paul McCartney / (John Lennon)      | `/`で分割要検討        |
| playing       | 演奏担当                                          | John Lennon / Paul McCartney / George Harrison     | `/`で分割要検討        |
| john          | ジョンのパート                                    | rhythm guitar, backing vocals                      | `,`区切りで分割も可    |
| paul          | ポールのパート                                    | bass, backing vocals                               | `,`区切りで分割も可    |
| george        | ジョージのパート                                  | lead guitar, backing vocals                        | `,`区切りで分割も可    |
| ringo         | リンゴのパート                                    | drums, maracas                                     | `,`区切りで分割も可    |
| musician      | 参加ミュージシャン（役割含む）                    | Pete Best : drums / Tony Sheridan : lead vocals, guitar | `/`区切り＋`: `でkey-value型 |
| songwriter    | 作詞作曲者                                        | Lennon-McCartney / Paul McCartney / (John Lennon)  | `/`区切りで一対多      |
| original      | 原曲名・原作者                                    | traditional, The Beatles                           | 単一値（複数の可能性） |
| producer      | プロデューサー                                    | Bert Kaempfert / George Martin                     | `/`区切りで一対多      |
| engineer      | エンジニア                                        | Karl Hinze / Norman Smith                          | `/`区切りで一対多      |
| artwork       | アートワーク担当（役割含む）                      | Angus McBean : photography                         | `/`区切り＋`: `でkey-value型 |
| film          | 映画                                              | -                                                  | 単一値（空欄/複数可）  |
| mv            | ミュージックビデオ等                              | -                                                  | 単一値（空欄/複数可）  |
| remarks       | 備考                                              | Polydorから発売されたトニーのシングルのA面 / ...   | `/`区切りで一対多（文脈で分ける）|
| source        | 情報源URL等                                       | https://en.wikipedia.org/... / ...                 | `/`区切りで一対多      |

---

## 備考

- `/` 区切り：一つのセルに複数データが入っている場合に使用。DB化時は中間テーブルや配列型、JSON型カラムなどで一対多の構造に分割することを推奨。
- `: ` 区切り：`musician`や`artwork`では「担当者: 役割」の形。DB化時は「担当者」と「役割」を分離して格納することを推奨。
- `,` 区切り：`john`などパート名の羅列。必要に応じて分割し、役割ごとに持たせることも可能。

---

## テーブル分割・正規化のための考え方・判断基準

- **一対多（1:N）や多対多（N:M）の関係が発生する場合はテーブルを分ける**  
  例：1曲に複数の作曲者やミュージシャンが参加する場合は、中間テーブルで管理

- **何度も繰り返し使われる情報（例：アルバム名・アーティスト名・レーベル名）はテーブルを分けてID参照にする**  
  → 表記揺れや重複データを防ぎ、データの一貫性を保つ

- **一対一の情報やほとんど使い回しがない属性は1テーブルにまとめることが多い**

- **情報どうしの”まとまり感”が強い場合は1テーブルにまとめる（例：john/paul/george/ringoの担当パートをtrack_membersテーブルとしてまとめる等）**

- **今後の拡張や検索パターン（例：ミュージシャンごとに曲を検索したいか等）を想定してテーブル分割を検討する**

- **JOIN数が多くなりすぎないようにバランスを取る**  
  目安：3〜5テーブルまでのJOINなら実用的、10以上になる場合は設計を見直す

- **正規化しすぎてパフォーマンスや実装・運用が複雑にならないよう”ほどほど”を意識する**

- **一対多でも種類が少なく、今後も増減がほぼない場合は、PostgreSQLのenum型やアプリ側の定数(enum)で管理する**

---

## 参考

- 元データはGoogleスプレッドシートのヘッダー行＋データをJSON形式でエクスポートしたもの。
- 本ファイルはDB設計・データ移行（マッピング）時の参照資料とする。
