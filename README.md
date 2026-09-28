
# Real-Time Stock Watchlist

Jetpack Compose app that shows live crypto prices from Binance public WebSocket streams.

## What it does

- Shows 8 pairs: BTC, ETH, SOL, BNB, XRP, ADA, DOGE, AVAX
- Live price updates over WebSocket
- Green/red flash when price goes up or down
- Connection status, loading, error + retry

## Architecture

```text
WebSocket → Repository → ViewModel (StateFlow) → Compose
```

- `data/websocket` – OkHttp WebSocket + JSON parsing
- `data/repository` – keeps latest prices, handles reconnect
- `domain` – models + repository interface
- `presentation` – ViewModel, UI state, Compose screen

DI is manual in `WatchlistApp`. No Hilt — project is small enough without it.

## Performance

- 'LazyColumn' with `key = symbol`
- Each row is its own composable (`StockRow`)
- WebSocket work is not in Compose; UI only collects `StateFlow`
- Previous price / direction calculated in repository before UI
- Flash animation state lives inside the row
- Uses `collectAsStateWithLifecycle()`

## Error handling

- Bad JSON messages are skipped
- On disconnect, repository retries up to 5 times (delay 2s–10s)
- After that UI shows error + Retry
- Retry cancels the old Flow (`flatMapLatest`) and starts again
- When ViewModel is cleared, collection stops and socket closes in `awaitClose`

## How to run

1. Open in Android Studio
2. Sync Gradle
3. Run on emulator/device with internet

```bash
./gradlew :app:assembleDebug
./gradlew :app:testDebugUnitTest
```


## Structure

```text
app/src/main/java/.../
├── WatchlistApp.kt
├── MainActivity.kt
├── data/
│   ├── websocket/
│   │   ├── BinanceWebSocketDataSource.kt
│   │   ├── BinanceTickerParser.kt
│   │   └── TickerUpdate.kt
│   └── repository/
│       └── WatchlistRepositoryImpl.kt
├── domain/
│   ├── model/
│   └── repository/
└── presentation/
    └── watchlist/
```

## Tests

- Price direction (up / down / none)
- Ticker JSON parsing
- ViewModel success / error / retry

