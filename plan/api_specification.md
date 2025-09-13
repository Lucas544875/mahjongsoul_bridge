# Mahjong Soul Bridge - API仕様書

## 概要

本ドキュメントでは、mahjongsoul_snifferとmahjong-cppライブラリを連携させる際のAPI仕様について詳述します。

## 1. mahjongsoul_sniffer連携API

### 1.1 WebSocket接続

#### 接続情報
- **URL**: `ws://localhost:8080/mahjong` (設定可能)
- **プロトコル**: WebSocket
- **データ形式**: JSON

#### 接続確立
```javascript
// JavaScript側での接続例
const ws = new WebSocket('ws://localhost:8080/mahjong');
```

### 1.2 受信データ形式

#### ゲーム状態更新メッセージ
```json
{
  "type": "game_state_update",
  "timestamp": "2024-01-15T10:30:45Z",
  "data": {
    "seat": 0,
    "hand": [11, 12, 13, 21, 22, 23, 31, 32, 33, 41, 42, 43, 51],
    "melds": [
      {
        "type": "pong",
        "tiles": [11, 11, 11],
        "from_player": 1
      }
    ],
    "dora_indicators": [41],
    "round_wind": 41,
    "seat_wind": 42,
    "turn": 1,
    "remaining_tiles": 70
  }
}
```

#### 手牌変更メッセージ
```json
{
  "type": "hand_change",
  "timestamp": "2024-01-15T10:30:50Z", 
  "data": {
    "action": "draw",
    "tile": 14,
    "new_hand": [11, 12, 13, 14, 21, 22, 23, 31, 32, 33, 41, 42, 43, 51]
  }
}
```

#### 副露メッセージ
```json
{
  "type": "meld_declared",
  "timestamp": "2024-01-15T10:31:00Z",
  "data": {
    "player": 0,
    "meld_type": "chow",
    "tiles": [21, 22, 23],
    "called_tile": 22,
    "from_player": 3
  }
}
```

### 1.3 雀魂タイル形式

#### タイルID対応表
```python
# 萬子 (1m-9m: 11-19, 赤5m: 15)
# 筒子 (1p-9p: 21-29, 赤5p: 25) 
# 索子 (1s-9s: 31-39, 赤5s: 35)
# 字牌 (東南西北白発中: 41-47)

SOUL_TILE_MAP = {
    11: "1m", 12: "2m", 13: "3m", 14: "4m", 15: "0m", 16: "5m", 
    17: "6m", 18: "7m", 19: "8m", 
    21: "1p", 22: "2p", 23: "3p", 24: "4p", 25: "0p", 26: "5p",
    27: "6p", 28: "7p", 29: "8p",
    31: "1s", 32: "2s", 33: "3s", 34: "4s", 35: "0s", 36: "5s",
    37: "6s", 38: "7s", 39: "8s",
    41: "1z", 42: "2z", 43: "3z", 44: "4z", 45: "5z", 46: "6z", 47: "7z"
}
```

## 2. mahjong-cpp API連携

### 2.1 HTTP接続情報

- **Base URL**: `http://localhost:50000`
- **Method**: POST
- **Content-Type**: `application/json`
- **Accept**: `application/json`

### 2.2 リクエスト仕様

#### 期待値計算リクエスト
```json
{
  "hand": "222567m345p33667s",
  "melds": [],
  "seat_wind": 1,
  "round_wind": 1, 
  "dora_indicators": [4],
  "enable_reddora": true,
  "enable_uradora": true,
  "enable_shanten_down": true,
  "enable_tegawari": true,
  "enable_riichi": false,
  "version": "0.9.1"
}
```

#### リクエストパラメータ詳細

| パラメータ | 型 | 必須 | 説明 |
|-----------|---|------|------|
| hand | string | ✓ | 手牌（mpsz記法） |
| melds | array | ✓ | 副露情報 |
| seat_wind | int | ✓ | 自風（1:東, 2:南, 3:西, 4:北） |
| round_wind | int | ✓ | 場風（1:東, 2:南, 3:西, 4:北） |
| dora_indicators | array | ✓ | ドラ表示牌 |
| enable_reddora | bool | - | 赤ドラ有効（default: true） |
| enable_uradora | bool | - | 裏ドラ有効（default: true） |
| enable_shanten_down | bool | - | シャンテン戻し（default: true） |
| enable_tegawari | bool | - | 手替わり考慮（default: true） |
| enable_riichi | bool | - | リーチ考慮（default: false） |
| version | string | - | APIバージョン |

#### 副露情報形式
```json
{
  "melds": [
    {
      "type": "pong",  // "pong", "chow", "closed_kong", "open_kong", "added_kong"
      "tiles": [1, 1, 1]  // mpsz記法での牌
    },
    {
      "type": "chow",
      "tiles": [1, 2, 3]
    }
  ]
}
```

### 2.3 レスポンス仕様

#### 成功レスポンス
```json
{
  "success": true,
  "request": { /* 送信したリクエストのecho */ },
  "response": {
    "config": {
      "t_min": 1,
      "t_max": 18,
      "enable_reddora": true,
      "enable_uradora": true,
      "enable_shanten_down": true,
      "enable_tegawari": true,
      "enable_riichi": false
    },
    "shanten": {
      "all": 3,
      "regular": 3,
      "seven_pairs": 6,
      "thirteen_orphans": 13
    },
    "stats": [
      {
        "tile": 22,
        "shanten": 3,
        "necessary": [
          {"tile": 21, "count": 4},
          {"tile": 23, "count": 4},
          {"tile": 24, "count": 4}
        ],
        "tenpai_prob": [0.0, 0.774, 0.712, 0.645, ...],
        "win_prob": [0.0, 0.210, 0.182, 0.156, ...],  
        "exp_score": [0.0, 1977.33, 1708.40, 1450.86, ...]
      }
    ],
    "time": 1250,
    "searched": 145832
  }
}
```

#### エラーレスポンス
```json
{
  "success": false,
  "request": { /* リクエスト内容 */ },
  "err_msg": "手牌はすでに和了形です。"
}
```

#### レスポンスフィールド詳細

| フィールド | 型 | 説明 |
|-----------|---|------|
| success | bool | 処理成功フラグ |
| shanten.all | int | 最小シャンテン数 |
| stats[].tile | int | 打牌候補 |
| stats[].shanten | int | その牌を切った後のシャンテン数 |
| stats[].necessary | array | 有効牌一覧 |
| stats[].tenpai_prob | array | ターン別聴牌確率 |
| stats[].win_prob | array | ターン別和了確率 |
| stats[].exp_score | array | ターン別期待値 |
| time | int | 計算時間（マイクロ秒） |
| searched | int | 探索した手牌数 |

## 3. データ変換仕様

### 3.1 雀魂 → mahjong-cpp変換

#### Python実装例
```python
class DataConverter:
    # 雀魂タイル→mpsz記法変換
    SOUL_TO_MPSZ = {
        11: "1m", 12: "2m", 13: "3m", 14: "4m", 15: "0m", 16: "5m",
        17: "6m", 18: "7m", 19: "8m",
        21: "1p", 22: "2p", 23: "3p", 24: "4p", 25: "0p", 26: "5p", 
        27: "6p", 28: "7p", 29: "8p",
        31: "1s", 32: "2s", 33: "3s", 34: "4s", 35: "0s", 36: "5s",
        37: "6s", 38: "7s", 39: "8s",
        41: "1z", 42: "2z", 43: "3z", 44: "4z", 45: "5z", 46: "6z", 47: "7z"
    }
    
    def soul_tiles_to_mpsz(self, soul_tiles: List[int]) -> str:
        """雀魂タイルリスト→mpsz記法文字列"""
        mpsz_tiles = [self.SOUL_TO_MPSZ.get(tile, "?") for tile in soul_tiles]
        return "".join(sorted(mpsz_tiles))
    
    def soul_melds_to_cpp_format(self, soul_melds: List[dict]) -> List[dict]:
        """雀魂副露→mahjong-cpp形式"""
        cpp_melds = []
        for meld in soul_melds:
            cpp_meld = {
                "type": self._convert_meld_type(meld["type"]),
                "tiles": [self.SOUL_TO_MPSZ[tile] for tile in meld["tiles"]]
            }
            cpp_melds.append(cpp_meld)
        return cpp_melds
        
    def _convert_meld_type(self, soul_type: str) -> str:
        """副露種別変換"""
        type_map = {
            "pong": "pong",
            "chow": "chow", 
            "closed_kong": "closed_kong",
            "open_kong": "open_kong",
            "added_kong": "added_kong"
        }
        return type_map.get(soul_type, "pong")
```

### 3.2 mahjong-cpp → 表示用変換

#### 結果データの整形
```python
class ResultFormatter:
    def format_calculation_result(self, cpp_response: dict) -> DisplayResult:
        """mahjong-cppレスポンス→表示用データ"""
        stats = cpp_response["response"]["stats"]
        formatted_stats = []
        
        for stat in stats:
            tile_name = self._get_tile_display_name(stat["tile"])
            formatted_stat = {
                "discard_tile": tile_name,
                "shanten": stat["shanten"],
                "useful_tiles": self._format_useful_tiles(stat["necessary"]),
                "tenpai_probability": stat["tenpai_prob"][1],  # 1巡目
                "win_probability": stat["win_prob"][1],
                "expected_score": stat["exp_score"][1]
            }
            formatted_stats.append(formatted_stat)
            
        return DisplayResult(
            current_shanten=cpp_response["response"]["shanten"]["all"],
            recommendations=formatted_stats,
            calculation_time=cpp_response["response"]["time"],
            searched_hands=cpp_response["response"]["searched"]
        )
```

## 4. エラーハンドリング

### 4.1 通信エラー

| エラー種別 | 対処法 |
|-----------|-------|
| WebSocket接続失敗 | 再接続リトライ（指数バックオフ） |
| HTTP通信タイムアウト | リクエスト再送（最大3回） |
| JSONパースエラー | ログ出力・次メッセージ処理継続 |

### 4.2 データエラー

| エラー種別 | 対処法 |
|-----------|-------|
| 未知のタイル形式 | デフォルト値設定・警告表示 |
| 不正な手牌構成 | エラーメッセージ表示 |
| 計算結果異常 | 前回結果表示維持 |

### 4.3 エラー応答例

```python
class BridgeException(Exception):
    """Bridge固有例外基底クラス"""
    pass

class CommunicationError(BridgeException):
    """通信エラー"""
    pass
    
class DataConversionError(BridgeException):  
    """データ変換エラー"""
    pass
    
class CalculationError(BridgeException):
    """計算エラー"""
    pass
```

## 5. 実装サンプルコード

### 5.1 WebSocket受信処理
```python
import asyncio
import websockets
import json

class MahjongSoulReceiver:
    async def connect_and_listen(self, uri: str):
        try:
            async with websockets.connect(uri) as websocket:
                while True:
                    message = await websocket.recv()
                    await self.handle_message(json.loads(message))
        except Exception as e:
            print(f"WebSocket error: {e}")
            # 再接続ロジック
            
    async def handle_message(self, data: dict):
        if data["type"] == "game_state_update":
            await self.process_game_state(data["data"])
        elif data["type"] == "hand_change":
            await self.process_hand_change(data["data"])
```

### 5.2 HTTP API呼び出し
```python
import requests
from typing import Dict, Any

class MahjongCppClient:
    def __init__(self, base_url: str = "http://localhost:50000"):
        self.base_url = base_url
        
    async def calculate_efficiency(self, request_data: Dict[str, Any]) -> Dict[str, Any]:
        try:
            response = requests.post(
                self.base_url,
                json=request_data,
                headers={"Content-Type": "application/json"},
                timeout=10
            )
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            raise CommunicationError(f"mahjong-cpp API error: {e}")
```

この仕様書に基づいて、確実で効率的なAPI連携を実現できます。