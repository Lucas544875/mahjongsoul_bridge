# Mahjong Soul Bridge - アーキテクチャ設計書

## 概要

mahjongsoul_snifferとmahjong-cppライブラリを連携させ、雀魂のゲーム進行中にリアルタイムで牌効率を計算・表示するPython仲介アプリケーション。

## アーキテクチャ

### システム構成図

```
[雀魂ブラウザ] 
    ↓ WebSocket通信
[mahjongsoul_sniffer]
    ↓ TCP/HTTP通信
[Mahjong Soul Bridge] ← 本アプリケーション
    ↓ HTTP JSON API
[mahjong-cpp server:50000]
    ↓
[牌効率計算結果表示]
```

### 設計原則

1. **疎結合**: 各システムの独立性を保持
2. **リアルタイム処理**: WebSocketによる即座の状態更新
3. **拡張性**: 新機能追加やUI変更への対応
4. **堅牢性**: エラーハンドリングと例外処理の充実

## 技術スタック

### Python ライブラリ

- **通信**
  - `websockets`: mahjongsoul_snifferからのWebSocket受信
  - `requests`: mahjong-cpp HTTPサーバーとの通信
  - `socket`: TCP通信サポート

- **UI/表示**
  - **オプション1**: `tkinter` - シンプルなデスクトップUI
  - **オプション2**: `fastapi` + HTML/CSS/JS - Web UI
  - **オプション3**: `PyQt5/6` - リッチなデスクトップUI

- **データ処理**
  - `json`: JSON形式のデータ変換
  - `asyncio`: 非同期処理
  - `logging`: ログ出力とデバッグ

### 外部システム連携

- **mahjongsoul_sniffer**: Python製通信キャプチャツール
- **mahjong-cpp**: C++製牌効率計算ライブラリ（HTTPサーバー機能付き）

## コンポーネント設計

### 1. 通信レイヤー

#### MahjongSoulReceiver
```python
class MahjongSoulReceiver:
    """mahjongsoul_snifferからの通信を受信"""
    
    async def connect_websocket(self, uri: str)
    async def handle_message(self, message: dict)
    def extract_game_state(self, message: dict) -> GameState
    def parse_hand_data(self, game_data: dict) -> HandInfo
```

#### MahjongCppClient
```python
class MahjongCppClient:
    """mahjong-cpp HTTPサーバーとの通信"""
    
    def __init__(self, base_url: str = "http://localhost:50000")
    async def calculate_efficiency(self, request: EfficiencyRequest) -> EfficiencyResponse
    def format_request(self, hand: Hand, config: Config) -> dict
    def parse_response(self, response: dict) -> CalculationResult
```

### 2. データ変換レイヤー

#### DataConverter
```python
class DataConverter:
    """雀魂とmahjong-cpp間のデータ形式変換"""
    
    def soul_tiles_to_mpsz(self, tiles: List[int]) -> str
    def soul_melds_to_cpp_format(self, melds: List[dict]) -> List[dict]
    def extract_dora_info(self, game_state: dict) -> List[int]
    def create_efficiency_request(self, game_data: GameData) -> EfficiencyRequest
```

### 3. ゲーム状態管理

#### GameStateManager
```python
class GameStateManager:
    """ゲーム状態の追跡と管理"""
    
    def update_game_state(self, new_data: dict)
    def get_current_hand(self) -> Hand
    def get_available_actions(self) -> List[Action]
    def is_calculation_needed(self) -> bool
```

### 4. 表示レイヤー

#### EfficiencyDisplay
```python
class EfficiencyDisplay:
    """計算結果の表示とUI管理"""
    
    def show_current_hand(self, hand: Hand)
    def display_efficiency_results(self, results: CalculationResult)
    def highlight_recommended_discard(self, tiles: List[Tile])
    def show_probability_stats(self, stats: Statistics)
```

### 5. 設定管理

#### ConfigManager
```python
class ConfigManager:
    """アプリケーション設定の管理"""
    
    def load_config(self) -> Config
    def save_config(self, config: Config)
    def get_calculation_options(self) -> CalculationOptions
    def update_ui_preferences(self, preferences: dict)
```

## データフロー

### 1. 初期化フロー
```
1. 設定ファイル読み込み
2. mahjong-cppサーバー接続確認
3. mahjongsoul_snifferとの通信確立
4. UI初期化
```

### 2. リアルタイム処理フロー
```
1. 雀魂ゲーム状態変更
2. mahjongsoul_snifferが変更を検出
3. Bridge アプリが変更を受信
4. データ形式変換 (雀魂 → mahjong-cpp)
5. 牌効率計算リクエスト送信
6. 計算結果受信・解析
7. UI更新・結果表示
```

### 3. エラーハンドリングフロー
```
1. 通信エラー → 再接続試行
2. データ形式エラー → ログ出力・スキップ
3. 計算エラー → デフォルト値表示
4. UI エラー → エラーメッセージ表示
```

## 主要な設計判断

### 1. 通信方式の選択

- **WebSocket**: リアルタイム性重視
- **HTTP**: mahjong-cppの既存APIを活用
- **非同期処理**: UIの応答性確保

### 2. データ形式

- **入力**: 雀魂の独自形式
- **変換**: mpsz記法（mahjong-cpp標準）
- **出力**: JSON形式（Web UI対応）

### 3. UI選択指針

- **シンプル重視**: tkinter
- **Web対応**: FastAPI + HTML
- **機能重視**: PyQt

## 拡張性への配慮

### 将来的な機能拡張

1. **複数ゲーム同時対応**
2. **統計情報の保存・分析**
3. **カスタム牌効率アルゴリズム**
4. **他の麻雀アプリケーション対応**

### プラグインアーキテクチャ

```python
class Plugin:
    def process_game_data(self, data: GameData) -> ProcessedData
    def customize_display(self, display: Display) -> Display
```

## セキュリティ考慮事項

1. **通信の暗号化**: HTTPS/WSS対応
2. **入力検証**: 不正なデータの排除
3. **権限管理**: 設定変更の制限
4. **ログ管理**: 機密情報の除外

## パフォーマンス要件

- **応答時間**: 500ms以内での効率計算
- **メモリ使用量**: 100MB以下
- **CPU使用率**: 通常時10%以下
- **同時接続**: 最大5セッション

## 運用・監視

### ログ出力

```python
import logging

# レベル別ログ
logging.info("ゲーム状態更新")
logging.warning("接続不安定")
logging.error("計算エラー")
logging.debug("詳細なデータ変換情報")
```

### 監視項目

- 通信状態（mahjongsoul_sniffer, mahjong-cpp）
- 計算処理時間
- エラー発生頻度
- メモリ使用量

この設計書に基づいて、段階的な実装を行い、各コンポーネントを独立してテスト可能な構造とします。