# How it works: the network market protocol

This is the protocol the game's market screens speak. It was worked out from the game's packet code (`ChPacketFactory` and `ChNetwork`), and the private server inside the app implements it. For the other fixes, see the [README](../README.md#-how-it-was-done).

## Framing

Everything is big-endian, over one TCP connection to `127.0.0.1:10217`. The original server was `chseed.co.kr:10217`.

```
request = i32 total_length | u16 group | u16 command | body
reply   = i32 total_length | u16 group | u16 command+1 | i32 result | body
```

`result` 0 means success. Other values select the game's own error messages.

Strings come in two forms:

- **mbcs:** `u16` byte count, then EUC-KR bytes. Used for the player id, the phone and the device model.
- **utf16:** `u16` character count, then UTF-16BE characters. Used for names and text.

Times are six `u16` values: year, month, day, hour, minute, second.

An **item** is a 104-byte record, with its stack count as an `i32` at offset 9. In lists, each item has a 4-byte id in front of it.

Every request body starts with the player id (mbcs).

## Commands

| Group / command | What it does | Reply body |
|---|---|---|
| `3e8/65` | Connect | (none) |
| `3e8/67` | Synchronize | name (utf16), type (u8), device (mbcs), version (i32) |
| `3e8/69` | Update device info | (none) |
| `3e8/6b` | Register member | (none). The private server never requires it. |
| `3e8/64` | Disconnect | No reply |
| `3ef/ab` | Notices | count (u8), then per notice: id (i32), title (utf16), text (utf16), time |
| `3ef/af`, `3ef/b1`, `3ef/b5` | Gifts received: money, items, points | count (i32) = 0 |
| `3f0/b7`, `3f0/bb` | Points held on the server / withdraw | 0 |
| `3ea/79` | Network money balance | money (i32) |
| `3ea/7b`, `3ea/7d` | Deposit / withdraw money (i32) | new balance (i32) |
| `3eb/83` | Network storage | count (u8), then id (i32) + item per entry |
| `3eb/85` | Deposit item: name (utf16) + item | (none) |
| `3eb/87` | Withdraw item: id, count | (none) |
| `3eb/89` | Put on sale: id, count, price | (none) |
| `3ec/8d` | My sales | count (u8), then id, item, expiry time, price (i32), state (u8) |
| `3ec/8f`, `3ec/91` | Cancel / extend a sale | (none) / new expiry time |
| `3ed/95`, `3ed/97`, `3ed/99` | Search by name / type / seller | total (i32), count (i32), then seller (utf16), price, id, item, expiry time |
| `3ed/9d` | Buy | Refused. Only your own items exist. |
| `3ee/a1`, `3ee/a3` | Give money / item | Refused. There's nobody to give to. |
| `3e9/6f` | Register rank | (none) |
| `3e9/71`, `3e9/73` | My rank / rank list | rank, score and stats |
| `3e9/75` | Events | None |

## Inside the app

- `S3Application` extends the launcher's `Application` class and starts the server thread when the app starts.
- Whichever app process binds the port first serves it. The others retry every 3 seconds.
- Data is stored in one file, `seed3server.dat`, with a log alongside it. Both are in `Android/data/com.zettabridge.seed3/files/seed3server/`.
- The server was tested against a scripted client that replays every message type before it went into the app.
