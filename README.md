# game_sudoku

Game Sudoku truyền thống.

## Thiết lập PvP thời gian thực

Trận đấu trực tuyến dùng Firebase Realtime Database và Firebase Anonymous Authentication. Mỗi người chơi cần kết nối Internet.

1. Tạo một dự án Firebase, thêm ứng dụng web, bật **Authentication → Sign-in method → Anonymous**, rồi tạo **Realtime Database**.
2. Trong `/home/runner/work/game_sudoku/game_sudoku/index.html`, thay các giá trị `YOUR_API_KEY`, `YOUR_PROJECT_ID` và `YOUR_APP_ID` trong `FIREBASE_CONFIG` bằng thông tin ứng dụng web trên Firebase. Dùng đúng URL database trong trường `databaseURL`.
3. Dán các quy tắc sau vào trang **Realtime Database → Rules**:

```json
{
  "rules": {
    "sudokuRooms": {
      "$roomCode": {
        ".read": "auth != null",
        ".write": "auth != null && ((!data.exists() && newData.child('hostUid').val() === auth.uid) || (data.child('status').val() === 'waiting' && data.child('hostUid').val() === auth.uid && !newData.exists()) || (data.child('status').val() === 'waiting' && !data.child('guestUid').exists() && newData.child('guestUid').val() === auth.uid && newData.child('status').val() === 'active') || (data.child('status').val() === 'active' && (data.child('hostUid').val() === auth.uid || data.child('guestUid').val() === auth.uid) && newData.child('status').val() === 'finished' && newData.child('winnerUid').val() === auth.uid))",
        ".validate": "!data.exists() || (newData.child('hostUid').val() === data.child('hostUid').val() && newData.child('seed').val() === data.child('seed').val() && newData.child('difficulty').val() === data.child('difficulty').val() && newData.child('createdAt').val() === data.child('createdAt').val() && (newData.child('guestUid').val() === data.child('guestUid').val() || (data.child('status').val() === 'waiting' && !data.child('guestUid').exists() && newData.child('guestUid').val() === auth.uid)) && (newData.child('status').val() === data.child('status').val() || (data.child('status').val() === 'waiting' && newData.child('status').val() === 'active' && newData.child('guestUid').val() === auth.uid) || (data.child('status').val() === 'active' && newData.child('status').val() === 'finished' && newData.child('winnerUid').val() === auth.uid)) && (newData.child('winnerUid').val() === data.child('winnerUid').val() || (data.child('status').val() === 'active' && newData.child('status').val() === 'finished' && newData.child('winnerUid').val() === auth.uid)))",
        "players": {
          "$playerUid": {
            ".write": "auth != null && auth.uid === $playerUid && (auth.uid === root.child('sudokuRooms').child($roomCode).child('hostUid').val() || auth.uid === root.child('sudokuRooms').child($roomCode).child('guestUid').val())",
            ".validate": "newData.hasChildren(['name', 'progress', 'completed']) && newData.child('progress').isNumber() && newData.child('progress').val() >= 0 && newData.child('progress').val() <= 1 && newData.child('completed').isBoolean()"
          }
        }
      }
    }
  }
}
```

4. Serve the page over HTTPS (or localhost during development) and open it in two browser sessions. On the **Phòng đấu** tab, one player creates a room and shares its four-digit code; the other enters that code to join.

Both clients generate the same puzzle from the room seed and difficulty. Player progress is written to the room in Realtime Database; the first client to claim a completed board atomically sets the winner.

The Firebase web configuration is public client configuration, not a server secret. Anonymous authentication and the rules above limit writes to authenticated room participants, but a client-only game cannot verify puzzle solutions against cheating clients. Do not use client-reported results for prizes or other high-stakes ranking without server-side validation.
