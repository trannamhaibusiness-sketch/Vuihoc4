SPEC.md — VUI HỌC (PWA GAME GIA ĐÌNH)

Phiên bản: 3.3

Ngày cập nhật: 2026-09-26

Trạng thái: Phase 3.3 hoàn thành — có SGK Toán 4



1\. 🎯 MỤC TIÊU

Xây dựng PWA game học tập cho trẻ lớp 4 trong phạm vi gia đình (\~100 user), gồm:



Nhiều môn (Toán, Toán TA, Tiếng Anh, Tin học)



Phân cấp Môn → Loại hình → (Chương) → Thi



2 track hiện có:



Violympic lớp 4 (500 câu)



SGK Toán 4 (13 chương × 100 câu = 1.300 câu)



Thi trắc nghiệm có timer



3 cơ chế thưởng: streak 5, daily 30+, daily login



Vòng quay may mắn + kho quà + đổi quà tự động



Lưu trữ qua Google Sheets



Cache localStorage để chơi nhanh, offline fallback



2\. 👥 NGƯỜI DÙNG

Role	Mô tả	Quyền

Student	Học sinh	Chơi, xem điểm, đổi quà

Admin	Người quản lý	Quản lý user, thêm câu hỏi

\~\~Parent\~\~	\~\~Phụ huynh\~\~	ĐÃ BỎ Ở v3.2

3\. 🏗️ KIẾN TRÚC

text

┌─────────────────────────────┐

│   PWA (GitHub Pages)        │

│  - index.html (gộp CSS+JS)  │

│  - manifest.json + sw.js    │

│  - icon.svg + 2 PNG         │

│  - localStorage cache 24h   │

└──────────────┬──────────────┘

&#x20;              │ HTTPS

&#x20;              ▼

┌─────────────────────────────┐

│  Google Apps Script (API)   │

│  - SECRET\_KEY               │

│  - 15+ actions              │

└──────────────┬──────────────┘

&#x20;              │

&#x20;              ▼

┌─────────────────────────────┐

│    Google Sheets (DB)       │

│  - 8 tabs                   │

└─────────────────────────────┘

Mục	Giá trị

API URL	https://script.google.com/macros/s/AKfycbztO88kMMXwKGgBkLfxZ-HiyCUxbXZYc1A-lfnOjJKV3DDXMFPEUpKOVdycJKoNyvrL/exec

API KEY	vuihoc\_family\_2026

Sheet name	VuiHoc\_DB

Timezone	Asia/Ho\_Chi\_Minh

4\. 🗄️ DATA MODEL

Sheet Users

Cột	Kiểu	Ghi chú

id	string	u001, p001, a001

username	string	Duy nhất

password	string	Plain

role	string	student / parent / admin

display\_name	string	

parent\_id	string	

total\_score	number	

created\_at	datetime	

last\_login\_date	date	YYYY-MM-DD

streak\_count	number	0-5

streak\_score	number	Tổng điểm 5 vòng

today\_rounds	number	Số vòng hôm nay

today\_date	date	Ngày của today\_rounds

Sheet Subjects

id	name	description	active

toan	Toán	Toán lớp 4	TRUE

toan\_en	Toán TA	Math in English	FALSE

en	Tiếng Anh	English	FALSE

tin	Tin học	Informatics	FALSE

Sheet Tracks

id	subject\_id	name	active	order

vio4	toan	Violympic lớp 4	TRUE	1

sgk4	toan	Toán lớp 4 (SGK)	TRUE	2

nc4	toan	Toán nâng cao 4	FALSE	3

Sheet Questions — CẬP NHẬT v3.3

Cột	Mô tả

id	q0001 (violympic) / sgk\_chuong\_X\_0001 (sgk)

subject\_id	toan, toan\_en, en, tin

track\_id	vio4, sgk4, nc4

question	Nội dung

option\_a-d	4 đáp án

correct	A/B/C/D

difficulty	1-3

game\_type	violympic / sgk / luyentap / thi

chapter\_id	chuong\_1 → chuong\_13 (null nếu violympic)

Quy tắc quan trọng:



game\_type trống → mặc định violympic (backward compatible với 500 câu cũ)



Violympic: chapter\_id = null



SGK: chapter\_id = chuong\_1 → chuong\_13



Số lượng hiện tại:



500 câu Violympic



1.300 câu SGK (13 chương × 100)



Sheet Results

| id | user\_id | subject\_id | track\_id | score | total\_questions | round\_number | played\_at |



Sheet Rewards

id	user\_id	value	status	earned\_at	approved\_at	source

rw\_login\_xxx	u001	2000	in\_warehouse	...		daily\_login

spin\_streak\_xxx	u001	0	pending\_spin\_streak	...		streak\_5

spin\_daily\_xxx	u001	0	pending\_spin\_daily	...		daily\_30

Status: pending\_spin\_streak / pending\_spin\_daily / in\_warehouse / redeemed

Source: daily\_login / streak\_5 / daily\_30 / manual



Sheet ExchangeRequests

id	user\_id	reward\_ids	total\_value	status	created\_at	approved\_at	approved\_by

ex\_xxx	u001	id1,id2	8000	completed	...	...	auto

Sheet Config

key	value

time\_per\_round	600

questions\_per\_round	10

streak\_count\_required	5

streak\_min\_score	45

daily\_rounds\_for\_bonus	30

daily\_login\_bonus	2000

streak\_rewards	1000:50,2000:25,3000:15,5000:10

daily\_rewards	1000:40,2000:30,5000:20,10000:7,20000:3

5\. 🎮 LUỒNG HOẠT ĐỘNG

5.1. Login

text

\[Login] → \[Vào Home NGAY] → \[Load status ngầm ở background]

5.2. Chơi game — Violympic

text

\[Home] → "Chơi ngay" → \[Chọn môn] → \[Chọn track: Violympic] → \[Thi luôn]

5.3. Chơi game — SGK (v3.3)

text

\[Home] → "Chơi ngay" → \[Chọn môn] → \[Chọn track: SGK Toán 4]

&#x20;    → \[Chọn chương] (13 chương + nút "Ôn tập tổng hợp")

&#x20;    → \[Thi 10 câu của chương đó]

5.4. Vòng quay + Kho quà

text

\[Thi xong] → \[Check streak 5 + daily 30+] → \[Popup "Chúc mừng + Quay ngay"]

&#x20;    → \[Vòng quay 4s] → \[Nhận voucher] → \[Lưu kho]

&#x20;    → \[Kho quà] → \[Đổi tất cả thành tiền] → \[Phụ huynh trả tiền mặt]

6\. ⏱️ QUY TẮC THI

Quy tắc	Giá trị

Số câu / vòng	10

Thời gian	600s (10 phút)

Format	Trắc nghiệm 4 đáp án A/B/C/D

Trộn câu	Có

Trộn đáp án	Có

Hết giờ	Tự động nộp

Feedback	Ngay khi chọn (xanh/đỏ + âm thanh)

Auto next	Sau 1.5s

Nút Nộp	Luôn hiển thị, có popup xác nhận

Nav back/home	Ẩn khi đang thi

Double-submit	Chặn (flag submitted)

7\. 🎁 CƠ CHẾ THƯỞNG

7.1. Streak 5 (cùng môn)

Chơi 5 vòng liên tiếp cùng 1 môn



Tổng điểm 5 vòng ≥ 45



→ 1 lượt quay streak: 1k/2k/3k/5k (50/25/15/10)



Sau khi quay → reset streak về 0



Đổi môn → reset streak



7.2. Daily 30+

Trong 1 ngày chơi ≥ 30 vòng (bất kỳ môn)



Mỗi mốc 30 (30/60/90…) → 1 lượt quay daily



Daily: 1k/2k/5k/10k/20k (40/30/20/7/3)



Reset theo ngày



7.3. Daily login (v3.2)

Điều kiện: đã chơi ≥ 1 bài trong ngày



Login lần đầu trong ngày → +2k vào kho



1 lần/ngày (không nhận nhiều)



Không bù ngày quên



Claim sau khi chơi bài đầu tiên



7.4. Popup thông báo

"Chúc mừng \[Tên]! Bé được 1 lượt quay may mắn 🎉"



Nút: "Quay ngay" / "Để sau"



8\. 🔄 ĐỔI QUÀ (v3.2 — bỏ phụ huynh)

Bước	Chi tiết

1	Học sinh vào Kho quà

2	Xem danh sách voucher (tổng giá trị)

3	Bấm "Đổi tất cả thành tiền"

4	Popup xác nhận

5	API exchangeRewards → tự động duyệt

6	Status rewards → redeemed

7	Ghi lịch sử vào ExchangeRequests

8	Kho trống, có lịch sử

9	Phụ huynh trả tiền mặt cho bé

9\. 🎨 UI/UX

9.1. Màu sắc

Tên	Hex

brand	#6366f1 (indigo)

sunny	#FCD34D (vàng)

coral	#FB7185 (hồng)

mint	#34D399 (xanh lá)

sky	#38BDF8 (xanh dương)

9.2. Font

Baloo 2 — tiêu đề



Nunito — nội dung



9.3. Nút điều hướng

⬅️ Quay lại — về màn hình trước



🏠 Trang chủ — về Home



Ẩn cả 2 khi đang thi



9.4. Màn hình (v3.3)

\#	Màn hình	Nav?

1	Login	Không

2	Home học sinh	Không

3	Chọn môn	Có

4	Chọn loại hình (track)	Có

5	Chọn chương (SGK)	Có

6	Thi	KHÔNG

7	Kết quả	Có

8	Chọn loại quay	Có

9	Vòng quay	Có

10	Kho quà	Có

9.5. Điều hướng Back khi ở màn kết quả

Violympic: Back → về chọn track



SGK: Back → về chọn chương



10\. 📚 DANH SÁCH 13 CHƯƠNG SGK TOÁN 4

\#	ID	Tên chương	Icon

1	chuong\_1	Ôn tập và bổ sung	📘

2	chuong\_2	Góc và đơn vị đo góc	📐

3	chuong\_3	Số có nhiều chữ số	🔢

4	chuong\_4	Đơn vị đo đại lượng	⚖️

5	chuong\_5	Phép cộng và phép trừ	➕

6	chuong\_6	Đường thẳng vuông góc, song song	📏

7	chuong\_7	Ôn tập học kì 1	📝

8	chuong\_8	Phép nhân và phép chia	✖️

9	chuong\_9	Thống kê, xác suất	📊

10	chuong\_10	Phân số	½

11	chuong\_11	Cộng trừ phân số	➕

12	chuong\_12	Nhân chia phân số	✖️

13	chuong\_13	Ôn tập cuối năm	🏆

Mỗi chương: 100 câu hỏi (tổng 1.300 câu).



Nút "Ôn tập tổng hợp": lấy random 10 câu từ toàn bộ 13 chương.



11\. 🛠️ TECH STACK

Layer	Công nghệ

Frontend	HTML + Vanilla JS + TailwindCSS (CDN)

PWA	manifest.json + Service Worker

Cache	localStorage (questions 24h, status 30s)

Font	Baloo 2 + Nunito (Google Fonts)

Backend	Google Apps Script

DB	Google Sheets

Deploy	GitHub Pages

12\. 📁 CẤU TRÚC FILE

text

Game Vui học/

├── index.html        (HTML + inline CSS + inline JS + PATCHES)

├── manifest.json

├── sw.js

├── icon.svg

├── icon-192.png

├── icon-512.png

└── gen-sgk.html      (Tool sinh câu hỏi SGK — dùng offline)

Lưu ý:



index.html gộp cả CSS + JS + patches ở cuối file



Mỗi patch có comment // PATCH vX.Y



13\. 🚀 TỐI ƯU TỐC ĐỘ

Vấn đề	Giải pháp

API chậm 5-20s	Cache câu hỏi 24h, status 30s

Login đợi lâu	Vào Home ngay, load status ngầm

Nộp bài đợi 20s	Hiện kết quả ngay, save API ngầm

Double-claim daily	Khóa localStorage + server check

Double-submit	Flag submitted

Cache keys:



vuihoc\_questions\_{subject}\_{track} — TTL 24h (Violympic)



vuihoc\_questions\_{subject}\_{track}\_{chapter} — TTL 24h (SGK)



vuihoc\_status\_{user\_id} — TTL 30s



vuihoc\_daily\_lock\_{user\_id}\_{date} — vĩnh viễn trong ngày



vuihoc\_scores — offline scores



14\. 📡 API ENDPOINTS

Action	Method	Params	Trả về

ping	GET	—	{pong, time}

login	GET	username, password	{user}

getSubjects	GET	—	\[subjects]

getTracks	GET	subject	\[tracks]

getQuestions	GET	subject, track, chapter, game\_type, limit	\[questions]

saveResult	POST	user\_id, subject\_id, track\_id, score...	{id, streakRewardTriggered, ...}

getUserStatus	GET	user\_id	{total\_score, streak, today\_rounds, ...}

spinReward	POST	user\_id, spin\_type	{value, reward\_id}

claimDailyLogin	POST	user\_id	{bonus, alreadyClaimed, reason}

getUserRewards	GET	user\_id	\[rewards]

exchangeRewards	POST	user\_id, reward\_ids	{ok, exchange\_id, total\_value}

getExchangeHistory	GET	user\_id	\[history]

getConfig	GET	—	{config}

getLeaderboard	GET	limit	\[top]

Quy tắc getQuestions:



game\_type trống → mặc định violympic



Nếu chapter = 'mixed' → lấy random từ toàn bộ SGK



Nếu có chapter = 'chuong\_X' → lọc theo chương



15\. 🔧 TOOL SINH CÂU HỎI

File gen-sgk.html

Chức năng: Sinh 13 file CSV cho 13 chương SGK



Chạy offline: Double-click mở bằng Chrome



Cách dùng: Chọn chương → bấm "Sinh CSV" → tải file



Số câu mặc định: 100/chương



Output: File CSV khớp cột với Sheet Questions



Lưu ý quan trọng khi sinh CSV

Prefix ' cho phân số: Tránh Sheets parse 1/2 thành ngày



javascript

if (/^\\d+\\/\\d+$/.test(str)) str = "'" + str;

Đúng thứ tự cột:



text

id | subject\_id | track\_id | question | option\_a | option\_b | option\_c | option\_d | correct | difficulty | game\_type | chapter\_id

Xóa header trước khi import (hoặc giữ nếu import thay thế)



Cách import CSV vào Sheets

Mở tab Questions



Ctrl+End → xuống hàng cuối



File → Import → Upload → chọn CSV



Import location: Append to current sheet



Separator: Comma



Bấm Import data



16\. 🗺️ ROADMAP

Phase	Nội dung	Trạng thái

1	PWA offline + thi cơ bản	✅

2	Google Sheets backend	✅

2.5	Cấu trúc Môn → Loại hình + Nav	✅

2.6	500 câu hỏi Violympic	✅

3	Vòng quay + 3 cơ chế thưởng	✅

3.1	Tối ưu tốc độ (cache)	✅

3.2	Daily bonus + Đổi quà tự động	✅

3.3	13 chương SGK + UI chọn chương	✅

4	\~\~Phụ huynh duyệt\~\~	❌ BỎ

5	Đa môn (Toán TA, TA, Tin học)	⏳

6	Admin panel	⏳

7	Deploy PWA GitHub Pages	⏳

17\. ⚠️ RỦI RO \& GIẢI PHÁP

Rủi ro	Giải pháp

API chậm	Cache localStorage + vào Home ngay

Copy file dài bị cắt	Chia tin nhắn, ghép liền

Mất mạng	FALLBACK\_QUESTIONS + localStorage

Trùng câu	Random + shuffle mỗi lần

PWA không chạy file://	Deploy GitHub Pages (HTTPS)

Double-claim	Khóa localStorage + server check

Sheets parse phân số thành ngày	Prefix ' trong CSV

Thứ tự cột CSV sai	Đảm bảo khớp Sheet Questions

Import đè lên data cũ	Dùng "Append to current sheet"

18\. 🔑 THÔNG TIN QUAN TRỌNG

Mục	Giá trị

User demo	minh/1234, lan/1234, bome/1234, admin/1234

Ngôn ngữ	100% Tiếng Việt

Timezone	Asia/Ho\_Chi\_Minh (GMT+7)

Múi giờ tính daily	Theo ngày VN

19\. 📌 QUY ƯỚC CODE

Tất cả trong 1 file index.html (trừ PWA files)



Dùng window.xxx thay vì ES6 module



Patch cuối file thay vì sửa giữa (dễ cập nhật)



Comment // PATCH vX.Y để dễ nhận biết



Không dùng framework



Override functions bằng cách khai báo lại — hàm sau sẽ ghi đè hàm trước



20\. ✅ TRẠNG THÁI HIỆN TẠI

Đã hoàn thành:



✅ Login + Home



✅ Chọn môn → Loại hình



✅ Chọn chương (SGK)



✅ Thi 10 câu / 10 phút



✅ Feedback đúng/sai tức thì



✅ 500 câu Violympic + 1.300 câu SGK



✅ Ôn tập tổng hợp



✅ Streak 5 + Daily 30+ + Daily login



✅ Vòng quay streak + daily



✅ Kho quà + Đổi quà tự động



✅ Lịch sử đổi quà



✅ Cache 24h + tối ưu tốc độ



✅ PWA files (manifest, sw, icons)



✅ Tool sinh câu hỏi SGK



Chưa làm:



⏳ Đa môn (Toán TA, TA, Tin học)



⏳ Admin panel



⏳ Test PWA trên GitHub Pages



⏳ Test kỹ nội dung 13 chương SGK



21\. 🎯 NEXT STEP

Chờ quyết định:



A: Test kỹ 13 chương SGK (chơi thử, báo lỗi)



B: Deploy PWA GitHub Pages



C: Phase 5 — Đa môn



D: Phase 6 — Admin panel

