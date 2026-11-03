# game_sudoku

Game Sudoku truyền thống.

## Thiết lập PvP thời gian thực

Trận đấu trực tuyến dùng Firebase Realtime Database và Firebase Anonymous Authentication. Mỗi người chơi cần kết nối Internet.

1. Tạo một dự án Firebase, thêm ứng dụng web, bật **Authentication → Sign-in method → Anonymous**, rồi tạo **Realtime Database**.
2. Trong `index.html`, dùng cấu hình ứng dụng web đúng với dự án Firebase, đặc biệt là `apiKey`, `projectId`, `appId` và URL `databaseURL` được hiển thị trong trang Realtime Database. Thêm tên miền đang chạy trò chơi vào **Authentication → Settings → Authorized domains**.
3. Trong **Authentication → Sign-in method**, bật **Anonymous**.
4. **Chỉ dùng tạm thời khi thử nghiệm:** Nếu cần mở quyền để kiểm tra kết nối PvP, dán các quy tắc sau vào **Realtime Database → Rules** rồi bấm **Publish**:

   ```json
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```

   Các quy tắc này cho phép bất kỳ ai đọc và ghi toàn bộ database, không yêu cầu đăng nhập và có thể bị lạm dụng. Chỉ sử dụng trong dự án thử nghiệm; sau khi kiểm tra, thay bằng các quy tắc giới hạn người chơi bên dưới.

   **Khuyến nghị:** Dùng các quy tắc giới hạn người chơi trong [`database.rules.json`](database.rules.json) cho PvP. Sao chép toàn bộ nội dung tệp đó vào **Realtime Database → Rules** rồi bấm **Publish**. Quy tắc cho phép tạo phòng mới và chỉ cho người chơi khác nhận phòng đang chờ bằng cách đặt `guestUid` của chính họ và chuyển trạng thái sang `active` trong cùng giao dịch. Nếu ứng dụng báo phòng đang chờ nhưng từ chối tham gia, hãy xuất bản lại chính xác tệp quy tắc này.

5. Serve the page over HTTPS (or localhost during development) and open it in two browser sessions. On the **Phòng đấu** tab, one player creates a room and shares its four-digit code; the other enters that code to join.

Both clients generate the same puzzle from the room seed and difficulty. Player progress is written to the room in Realtime Database; the first client to claim a completed board atomically sets the winner.

The Firebase web configuration is public client configuration, not a server secret. The client already initializes Firebase Authentication and automatically calls `signInAnonymously()` when no user session exists. With the recommended rules, anonymous authentication and the rules above limit writes to authenticated room participants, but a client-only game cannot verify puzzle solutions against cheating clients. Do not use client-reported results for prizes or other high-stakes ranking without server-side validation.
