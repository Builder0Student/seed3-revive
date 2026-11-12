# Changelog

All builds are signed with the same key and install over each other.

## Build 5 · `Seed3-64bit-5.apk` (latest)

- **Fixed:** the game crashed when saving, from both **System → Save** and the save icon on the field. The save encryption key came from a seeded `SHA1PRNG`, which modern Android no longer makes repeatable, so every save used a different key. The key is now the MD5 of the device UUID. Saves go to a fresh table, because saves from earlier builds can't be decrypted.

## Build 4 · `Seed3-64bit-4.apk`

- **Added:** a private network-market server inside the app on `127.0.0.1:10217`. It handles storage, sales, search, rankings and notices, with automatic login.
- **Removed:** the remaining OpenFeint calls, such as score uploads and the dashboard, which would crash once the game was running.

## Build 3 · `Seed3-64bit-3.apk`

- **Changed:** buying a point pack now succeeds without payment and adds the pack's points. The balance is not infinite. The Colosseum and Warp Gate unlocks succeed the same way.

## Build 2 · `Seed3-64bit-2.apk`

- **Fixed:** a crash right after loading. `callNativeGetSkills` filled a `short[]` with `SetByteArrayRegion`. One byte in `libndk_kingdoms.so` now makes it use `SetShortArrayRegion`.

## Build 1 · `Seed3-64bit-1.apk`

- The original 32-bit game runs on 64-bit-only phones through the ZettaBridge translation layer, as its own app.
- **Switched off** the steps that modern Android blocks: the phone-number read, the cheat-app scan and its blocklist download, OpenFeint, Android Market billing, the SponsorPay offer wall, and `restartPackage` on exit.
