# BEATLES DB スキーマ設計・ER図

このドキュメントは、ビートルズ関連データベースのスキーマ設計およびER図（Entity-Relationship Diagram）をまとめたものです。  
設計方針や正規化基準、主なテーブル・カラム構成も記載します。

---

## 設計方針・正規化基準

- 一対多・多対多の関係は中間テーブルで正規化
- アルバム/シングル/EP/映画/MVなどは`releases`テーブルで一元管理
- 役割（パート名など）は`roles`テーブルで管理し、表記ゆれを排除
- 選択肢が少なく増減の少ない属性（format/type）はENUM型または定数管理を想定
- 備考や情報源はtracksまたはreleasesテーブルに付随

---

## ER図（Mermaid記法）

```mermaid
erDiagram
    TRACKS {
        int id PK
        string title
        int release_id FK
        int number
        int disc
        string side
        int year
        date date
        string remarks
        string source
    }
    RELEASES {
        int id PK
        string title
        enum type
        enum format
        date release_date
        int label_id FK
        int artist_id FK
        string country
        string category
        string icon
        string path
    }
    ARTISTS {
        int id PK
        string name
    }
    LABELS {
        int id PK
        string name
    }
    MUSICIANS {
        int id PK
        string name
    }
    ROLES {
        int id PK
        string name
    }
    SONGWRITERS {
        int id PK
        string name
    }
    ARTWORKS {
        int id PK
        string name
    }
    PRODUCERS {
        int id PK
        string name
    }
    ENGINEERS {
        int id PK
        string name
    }

    %% 中間テーブル
    TRACK_MUSICIANS {
        int track_id FK
        int musician_id FK
        int role_id FK
    }
    TRACK_SONGWRITERS {
        int track_id FK
        int songwriter_id FK
        string role
    }
    RELEASE_ARTWORKS {
        int release_id FK
        int artwork_id FK
        int role_id FK
    }
    TRACK_PRODUCERS {
        int track_id FK
        int producer_id FK
    }
    TRACK_ENGINEERS {
        int track_id FK
        int engineer_id FK
    }

    %% 関連
    TRACKS }o--|| RELEASES : "belongs to"
    RELEASES }o--|| ARTISTS : "main artist"
    RELEASES }o--|| LABELS : "label"
    TRACKS ||--o{ TRACK_MUSICIANS : ""
    MUSICIANS ||--o{ TRACK_MUSICIANS : ""
    ROLES ||--o{ TRACK_MUSICIANS : ""
    TRACKS ||--o{ TRACK_SONGWRITERS : ""
    SONGWRITERS ||--o{ TRACK_SONGWRITERS : ""
    TRACKS ||--o{ TRACK_PRODUCERS : ""
    PRODUCERS ||--o{ TRACK_PRODUCERS : ""
    TRACKS ||--o{ TRACK_ENGINEERS : ""
    ENGINEERS ||--o{ TRACK_ENGINEERS : ""
    RELEASES ||--o{ RELEASE_ARTWORKS : ""
    ARTWORKS ||--o{ RELEASE_ARTWORKS : ""
    ROLES ||--o{ RELEASE_ARTWORKS : ""
```

---

## テーブル一覧・補足

- `TRACKS` … 曲（トラック）ごとの基本情報
- `RELEASES` … リリース（アルバム、シングル、EP、映画、MV等）全般を管理
- `ARTISTS` … メインアーティスト
- `LABELS` … レーベル
- `MUSICIANS` … 参加ミュージシャン（ジョン、ポールなど含む）
- `ROLES` … パート名・役割（guitar, vocals, photography等）
- `SONGWRITERS` … 作詞作曲者
- `ARTWORKS` … アートワーク担当者
- `PRODUCERS` … プロデューサー
- `ENGINEERS` … エンジニア
- 中間テーブル（`TRACK_MUSICIANS`等）は多対多や役割付与のため

---

## 備考

- format/typeの詳細はENUMまたは対応表で管理
- 必要に応じてフィールド追加や関係調整してください