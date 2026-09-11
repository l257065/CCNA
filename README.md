# CCNA 200-301 練習（加密）

兩份練習頁，各自獨立、各有各的密碼：

| 入口 | 題數 | 網址 |
|---|---|---|
| 巨匠題庫 | 1194 | [pcschool/pcschool-practice.html](pcschool/pcschool-practice.html) |
| ExamTopics | 1395 | [examtopics/examtopics-practice.html](examtopics/examtopics-practice.html) |

👉 https://l257065.github.io/CCNA/

## 原始碼不在這裡

題庫明文、抓取與翻譯工具、471 張重繪圖的原始檔，全部在**另一個私有 repo**（`CCNA_practice`）。

**為什麼要拆**：練習頁是單檔自足的——整份題庫連正解、解析、翻譯、社群留言都內嵌在
HTML 裡。放上 GitHub Pages 就等於開放下載，而 **GitHub Pages 在個人帳號上沒有任何
存取控制**：

> 官方〈Changing the visibility of your GitHub Pages site〉：存取控制需要
> GitHub Enterprise Cloud ＋組織持有的 repo，而且「除非用 Enterprise Managed Users，
> **即使 repo 是 private 或 internal，Pages 網站預設仍然公開在網際網路上**」。

⚠️ 所以**把 repo 轉成私有是沒有用的**，網站照樣公開。唯一可行的是把內容本身加密，
而那又要求**明文不能待在同一個公開 repo 裡**——否則點進 repo 就繞過密碼了。

## 加密方式

**頁面**：`PBKDF2-HMAC-SHA256`（60 萬次迭代、16 bytes 隨機 salt）導出 256-bit 金鑰
→ `AES-256-GCM`（12 bytes 隨機 IV）加密整份 HTML → 瀏覽器用 WebCrypto 解密後
`document.write` 出來。

**圖片**：1760 張拓撲圖與示意圖也各自用 `AES-256-GCM` 加密成 `<網站>/a/<雜湊>.bin`，
檔名是路徑的 HMAC，看不出是哪一題的圖。頁面解鎖後，內嵌的解密器在瀏覽器端抓 `.bin`、
解密、換成 blob URL，每次只解目前這題用到的圖。圖片金鑰寫在頁面明文裡，跟整份頁面
一起被密碼加密，沒密碼拿不到。

全部走 Node 與瀏覽器內建的原語，沒有任何相依套件。產生方式是私有 repo 的
`node tools/deploy_web.js`，一次加密兩份頁面與所有圖片，寫完會把每一張都解回來
跟來源逐位元比對。

兩份用不同的 sessionStorage 鑰匙（`ccna_pcschool` / `ccna_examtopics`）。**這點與 Azure 那組不同**：
Azure 兩份共用同一個密碼所以共用鑰匙，這裡兩份密碼不同，若還共用鑰匙，從 A 解鎖後
點進 B 會拿 A 的密碼白跑一次 60 萬次 PBKDF2（手機上要卡好幾秒）才失敗。

## ⚠️ 這套加密擋得住什麼、擋不住什麼

**擋得住**：沒有密碼的人拿到的是密文——**頁面與圖片都是**。這是真的擋得住（不像
「JS 跳個輸入框」那種一看原始碼就破）。也擋掉了搜尋引擎與掃描機器人（另有 `noindex,nofollow`）。

**擋不住**：

1. **密文會落在對方電腦上、可以離線暴力破解。** 目前用的是短密碼，實際效果是擋掉隨手點進來的人，不是擋想拿題庫的人。

2. **已經解鎖的人**可以另存圖片，或從開發者工具拿到解密後的內容。

3. **2026-09-08 至 09-11 之間，這 1760 張圖曾以明文放在這個 repo**，可能已被快取或下載。
   加密是止血，不是收回。

## 內容來源

ExamTopics 那份的題目原文與社群留言來自 examtopics.com，站方 ToS 不歡迎轉載，
所以這個 repo 才把內容加密、並標 `noindex`。這是止血措施，不是授權。
