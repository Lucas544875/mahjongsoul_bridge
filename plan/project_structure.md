# Mahjong Soul Bridge - プロジェクト構造

## ディレクトリ構成

```
mahjong-soul-bridge/
├── README.md                    # プロジェクト概要
├── requirements.txt             # Python依存関係
├── config.yaml                  # 設定ファイル
├── main.py                      # アプリケーションエントリーポイント
├── 
├── src/
│   ├── __init__.py
│   ├── 
│   ├── bridge/                  # 核となるブリッジ機能
│   │   ├── __init__.py
│   │   ├── receiver.py          # mahjongsoul_sniffer通信受信
│   │   ├── client.py            # mahjong-cpp API クライアント
│   │   ├── converter.py         # データ変換ロジック
│   │   ├── game_state.py        # ゲーム状態管理
│   │   └── exceptions.py        # カスタム例外クラス
│   │   
│   ├── ui/                      # ユーザーインターフェース
│   │   ├── __init__.py
│   │   ├── base_display.py      # 基底表示クラス
│   │   ├── tkinter_ui.py        # Tkinter GUI
│   │   ├── web_ui.py            # FastAPI Web UI
│   │   └── console_ui.py        # コンソール出力
│   │   
│   ├── config/                  # 設定管理
│   │   ├── __init__.py
│   │   ├── settings.py          # 設定管理クラス
│   │   └── constants.py         # 定数定義
│   │   
│   ├── models/                  # データモデル
│   │   ├── __init__.py
│   │   ├── game_models.py       # ゲーム関連モデル
│   │   ├── api_models.py        # API通信モデル
│   │   └── ui_models.py         # UI表示モデル
│   │   
│   └── utils/                   # ユーティリティ
│       ├── __init__.py
│       ├── logger.py            # ログ設定
│       ├── validators.py        # データ検証
│       └── helpers.py           # 汎用ヘルパー関数
│
├── tests/                       # テストコード
│   ├── __init__.py
│   ├── test_converter.py        # データ変換テスト
│   ├── test_client.py           # APIクライアントテスト
│   ├── test_game_state.py       # ゲーム状態テスト
│   └── fixtures/                # テストデータ
│       ├── sample_soul_data.json
│       └── sample_cpp_response.json
│
├── docs/                        # ドキュメント
│   ├── setup.md                 # セットアップガイド
│   ├── usage.md                 # 使用方法
│   └── api_reference.md         # API リファレンス
│
├── logs/                        # ログファイル保存先
│   └── .gitkeep
│
└── assets/                      # 静的リソース
    ├── images/                  # 画像ファイル
    │   └── tiles/               # 牌画像
    ├── css/                     # スタイルシート
    └── js/                      # JavaScript
```

## 主要ファイルの詳細

### main.py
```python
"""
アプリケーションのエントリーポイント
- 設定の読み込み
- UI方式の選択
- ブリッジの起動
"""
```

### src/bridge/receiver.py
```python
"""
mahjongsoul_snifferからの通信受信
- WebSocket接続管理
- メッセージパースing
- ゲーム状態の更新通知
"""

class MahjongSoulReceiver:
    async def connect(self, uri: str)
    async def listen(self)
    def parse_message(self, message: str) -> dict
    def extract_hand_info(self, data: dict) -> HandInfo
```

### src/bridge/client.py
```python
"""
mahjong-cpp HTTPサーバーとの通信
- HTTP リクエスト送信
- レスポンス解析
- エラーハンドリング
"""

class MahjongCppClient:
    def __init__(self, base_url: str = "http://localhost:50000")
    async def post_calculation_request(self, data: dict) -> dict
    def build_request_payload(self, hand: Hand, config: Config) -> dict
    def parse_calculation_result(self, response: dict) -> CalculationResult
```

### src/bridge/converter.py
```python
"""
データ形式変換
- 雀魂 → mahjong-cpp形式変換
- mpsz記法への変換
- エラー時のフォールバック処理
"""

class DataConverter:
    SOUL_TO_MPSZ_MAP = {...}  # タイル変換マップ
    
    def convert_soul_tiles(self, tiles: List[int]) -> str
    def convert_soul_melds(self, melds: List[dict]) -> List[dict]
    def extract_dora_indicators(self, game_state: dict) -> List[int]
```

### src/bridge/game_state.py
```python
"""
ゲーム状態の管理
- 状態の保持・更新
- 変更の検出
- 計算トリガー判定
"""

class GameStateManager:
    def __init__(self)
    def update_state(self, new_data: dict) -> bool
    def get_current_hand(self) -> Hand
    def should_calculate(self) -> bool
    def get_calculation_context(self) -> CalculationContext
```

### src/ui/base_display.py
```python
"""
UI基底クラス
- 共通インターフェース定義
- 表示データの標準化
"""

class BaseDisplay(ABC):
    @abstractmethod
    def show_hand(self, hand: Hand)
    
    @abstractmethod
    def show_results(self, results: CalculationResult)
    
    @abstractmethod
    def show_error(self, error: str)
```

### src/models/game_models.py
```python
"""
ゲーム関連のデータモデル
"""

@dataclass
class Tile:
    id: int
    suit: str  # 'm', 'p', 's', 'z'
    rank: int
    is_red: bool = False

@dataclass 
class Hand:
    tiles: List[Tile]
    melds: List[Meld]
    
@dataclass
class GameState:
    current_hand: Hand
    dora_indicators: List[Tile]
    round_wind: Tile
    seat_wind: Tile
    turn_number: int
```

### src/models/api_models.py
```python
"""
API通信用のデータモデル
"""

@dataclass
class CalculationRequest:
    hand: str  # mpsz記法
    melds: List[dict]
    dora_indicators: List[int]
    enable_reddora: bool
    enable_uradora: bool
    # ... その他の設定

@dataclass
class CalculationResult:
    stats: List[TileEfficiency]
    shanten: int
    calculation_time: float
    searched_hands: int
```

## モジュール間の依存関係

```
main.py
├─→ src/config/settings.py
├─→ src/bridge/receiver.py
├─→ src/bridge/client.py
├─→ src/ui/[ui_type].py
└─→ src/utils/logger.py

src/bridge/receiver.py
├─→ src/bridge/converter.py  
├─→ src/bridge/game_state.py
└─→ src/models/game_models.py

src/bridge/client.py
├─→ src/models/api_models.py
└─→ src/bridge/converter.py

src/ui/[各UI]
├─→ src/models/ui_models.py
└─→ src/utils/helpers.py
```

## 設定ファイル構造

### config.yaml
```yaml
# 通信設定
communication:
  soul_sniffer:
    websocket_uri: "ws://localhost:8080"
    reconnect_interval: 5
    
  mahjong_cpp:
    base_url: "http://localhost:50000"
    timeout: 10
    retry_count: 3

# 計算設定  
calculation:
  enable_reddora: true
  enable_uradora: true
  enable_shanten_down: true
  enable_tegawari: true
  enable_riichi: false

# UI設定
ui:
  type: "tkinter"  # "tkinter", "web", "console"
  theme: "default"
  auto_update: true
  show_probabilities: true

# ログ設定
logging:
  level: "INFO"
  file_path: "logs/bridge.log"
  max_size: "10MB"
  backup_count: 5
```

## 開発・テスト戦略

### 単体テスト
- 各モジュールの独立したテスト
- モックを使用した外部依存の分離
- データ変換ロジックの詳細テスト

### 結合テスト  
- mahjongsoul_snifferとの通信テスト
- mahjong-cppとのAPI連携テスト
- UI表示の動作確認

### パフォーマンステスト
- 大量データ処理の性能測定
- メモリリーク検出
- 応答時間の計測

この構造により、保守性と拡張性を両立させ、各コンポーネントが独立して動作・テスト可能な設計となります。