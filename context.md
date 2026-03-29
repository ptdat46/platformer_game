Sunnyland Platformer — Giải thích Logic Toàn bộ Dự án
Tổng quan
Đây là một game platformer 2D side-scrolling xây dựng hoàn toàn bằng vanilla JavaScript + HTML5 Canvas, không dùng engine hay framework nào. Game lấy cảm hứng từ bộ asset Sunnyland (nghệ thuật pixel, chủ đề rừng).

1. Cấu trúc thư mục

sunnyland-platformer/
├── index.html                    ← Điểm khởi đầu, tạo canvas
├── js/
│   ├── index.js                  ← Game loop, render, khởi tạo game
│   ├── eventListeners.js         ← Xử lý input bàn phím
│   └── utils.js                  ← Load ảnh + hàm va chạm AABB
├── classes/
│   ├── Player.js                 ← Nhân vật chính (vật lý, animation)
│   ├── Oposum.js                 ← Kẻ thù đi bộ trên mặt đất
│   ├── Eagle.js                  ← Kẻ thù bay trên trời
│   ├── Sprite.js                 ← Sprite động (gem, hiệu ứng nổ)
│   ├── Heart.js                  ← Trái tim HUD (máu)
│   ├── CollisionBlock.js         ← Tile rắn không đi qua
│   └── Platform.js               ← Tile nửa-vừa-đi-qua (one-way)
└── data/
    ├── collisions.js             ← Bản đồ tile rắn
    ├── l_Gems.js                  ← Vị trí ngọc trong map
    ├── l_Back_Tiles.js            ← Layer nền sau
    ├── l_Front_Tiles.js           ← Layer nền trước
    ├── l_New_Layer_8.js           ← Layer giữa
    ├── l_Decorations.js           ← Trang trí
    ├── l_Grass.js, l_Trees.js...  ← Các layer trang trí khác
    └── l_New_Layer_1/2.js         ← Layer ocean + bramble
2. Canvas & Hiển thị
Thuộc tính	Giá trị
Độ phân giải logic	1024 × 576 px
Device pixel ratio	dpr = 2
Canvas thực tế	2048 × 1152 px (3×)
Rendering style	pixelated (giữ pixel rõ ràng)
Điểm đặc biệt: Canvas được scale lên 3 lần để hiển thị sắc nét trên màn hình Retina/HD.

3. Game Loop — Vòng lặp game

startRendering()       → Render tất cả tile tĩnh ra offscreen canvas (1 LẦN)
       ↓
animate()              → Gọi mỗi frame (60 lần/giây)
       ↓
1. Tính deltaTime      → Thời gian giữa 2 frame
2. player.handleInput()→ Đọc phím người chơi
3. player.update()     → Vật lý + va chạm tile
4. Cập nhật Oposum/Eagle + va chạm player↔enemy
5. Cập nhật Gem + va chạm player↔gem
6. Cập nhật hiệu ứng (nổ enemy, feedback ngọc)
7. Cập nhật Camera
8. Render: clear → background → entities → UI
9. requestAnimationFrame(animate) → loop tiếp
Camera:

Horizontal scroll: khi player.x > 330 → camera跟踪 player
Vertical scroll: lên khi player.y < 100, xuống khi player.y > 220
Giới hạn: không lộn ra ngoài bản đồ (MAX_CAMERA_X = 1264, MAX_CAMERA_Y = 448)
4. Tất cả Entities & Classes
🧑 Player (Player.js)
Nhân vật chính 32×32 px với hitbox 20×23 px.

Animation states:

State	Hàng	Frames
idle	y=0	4
run	y=32	6
jump	y=160	1
fall	y=160, x=33	1
roll	y=288	4
Vật lý:

X_VELOCITY = 200 px/s — tốc độ di chuyển
JUMP_POWER = 250 px/s — lực nhảy
GRAVITY = 580 px/s² — trọng lực
Controls:

Phím	Hành động
W	Nhảy
A	Trái
D	Phải
Space	Lăn/dash (khi đang đứng trên đất)
Cơ chế Roll:

Spacebar khi đứng trên đất → lăn với tốc độ 300 px/s (1.5× bình thường)
Khi lăn mà đâm vào Oposum từ bên hông → tiêu diệt Oposum
Kết thúc roll khi animation frame === 3
🦔 Oposum (Oposum.js)
Kẻ thù đi bộ trên mặt đất:

Kích thước: 36×28 px (hitbox 30×19 px)
Tốc độ: -20 px/s (bắt đầu đi trái)
Sau khi đi 100 px → quay đầu
Có gravity (rơi xuống)
🦅 Eagle (Eagle.js)
Kẻ thù bay trên trời:

Kích thước: 40×41 px
Di chuyển ngang như Oposum
Bug: Gravity bị comment → Eagle không rơi, chỉ bay ngang
Bug: Eagle đọc phím của player → di chuyển theo hướng player bấm
💎 Sprite (Sprite.js)
Class generic cho các sprite có animation:

Gem — 5 frame xoay, thu thập được
Enemy death — 6 frame hiệu ứng nổ
Item feedback — 5 frame popup khi nhặt đồ
Mỗi frame cách nhau 0.1s. Sprite tạm thời tự xóa sau 1 vòng animation.

❤️ Heart (Heart.js)
Trái tim HUD:

Frame 0 = đầy máu (bình thường)
Frame 1 = trống rỗng (đã mất)
Vẽ trực tiếp lên canvas (không bị ảnh hưởng bởi camera)
🧱 CollisionBlock (CollisionBlock.js)
Tile rắn 16×16 px. Player và enemy không thể đi qua. Tạo từ collisions.js (symbol === 1).

🪵 Platform (Platform.js)
Tile nửa-vừa-đi-qua (symbol === 2):

Chỉ dày 4 px
Player có thể nhảy từ dưới lên để đi qua
Player đáp xuống được khi nhảy từ trên

// Va chạm chỉ xảy ra khi player đến từ TRÊN
player.y + player.height <= this.y &&  // đang ở trên
player.y + player.height + velocity * dt >= this.y  // sẽ chạm
5. Hệ thống Va chạm (AABB)
Thuật toán AABB (Axis-Aligned Bounding Box):


checkCollisions(obj1, obj2)
├── So sánh overlap ngang (ox) và dọc (oy)
├── overlap nhỏ hơn → đó là hướng va chạm chính
└── Trả về: 'left' | 'right' | 'top' | 'bottom'
Xử lý va chạm ngang: Đẩy player sang bên ngoài block, reset velocity ngang

Xử lý va chạm dọc:

Đi lên → dừng ở đáy block
Đi xuống → đáp lên trên block, đánh dấu isOnGround = true
6. Hệ thống Máu (Health)
3 trái tim hiển thị góc trên-trái (x=10, 33, 56)
Khi va chạm với enemy (nếu không trong trạng thái bất tử):
1 trái tim bị trống
Bật 1.5s invincibility (nhân vật mờ 50%)
Trong thời gian invincibility → không nhận sát thương
Khi 0 trái tim → gọi init() → reset toàn bộ game (không có màn hình game over)
7. Hệ thống Điểm (Gems)
Đếm số gem đã thu thập
Khi thu thập gem → hiệu ứng feedback popup + animation
Khi thu thập hết tất cả gem → console.log('YOU WIN!') (không có màn hình thắng thật sự)
8. Bản đồ (Tile Maps)
Kích thước thế giới: 143 cột × 64 hàng × 16 px = 2288 × 1024 px
Dữ liệu bản đồ là mảng 2D các số nguyên (tile IDs)
Mỗi layer được render ra offscreen canvas 1 lần khi khởi động → mỗi frame chỉ drawImage() canvas đó
Thứ tự render (back → front):


1. Ocean background (nước)
2. Bramble background (bụi gai)
3. Static tile layers (cây, cỏ, nền...)
4. Player, enemies, gems, effects
5. UI (trái tim, icon ngọc + số)
9. Các Trạng thái Game
Trạng thái	Xử lý
Playing	Game loop bình thường
Chết (0 máu)	init() → reset im lặng
Thắng	console.log('YOU WIN!') → game tiếp tục chạy
Menu/Pause	Không tồn tại
⚠️ Game không có menu, không có pause, không có màn hình thắng/thua — chỉ reset im lặng khi chết.

10. Các Bug / Vấn đề đáng chú ý
#	Bug	Ảnh hưởng
1	Eagle gravity bị comment out	Eagle không rơi, chỉ bay ngang vĩnh viễn
2	Eagle đọc keys của player	Eagle di chuyển theo hướng player bấm phím
3	Không có màn hình thắng/thua	Chỉ console.log, game chạy tiếp
4	Canvas scale bằng dpr + 1	Scale 3× thay vì 2× (vẫn hoạt động đúng)
5	Vị trí hearts hardcoded	Không tự động tính theo số lượng
Sơ đồ luồng dữ liệu tổng quát

Bạn bấm phím (WASD/Space)
       ↓
eventListeners.js ghi vào keys{}
       ↓
Player.handleInput() đọc keys{}
       ↓
Player.update() áp dụng vật lý
       ↓
checkCollisions() vs CollisionBlocks → đẩy ra khỏi tường
       ↓
checkCollisions() vs Enemies → sát thương hoặc tiêu diệt
       ↓
checkCollisions() vs Gems → thu thập + cập nhật điểm
       ↓
Camera theo dõi Player
       ↓
Render: background → entities → UI
       ↓
requestAnimationFrame → loop tiếp

Chi tiết hệ thống hiển thị Tile Map
1. Cấu trúc dữ liệu bản đồ (Tile Map Data)
Mỗi file data/*.js là một mảng 2D

// Mỗi hàng = 1 dòng tile (trục Y)
const collisions = [
  [0, 0, 0, 0, ..., 1, 0, 0, ...],  // row 0 (y = 0)
  [0, 0, 0, 0, ..., 1, 0, 0, ...],  // row 1 (y = 1)
  ...
]
// Mỗi phần tử = 1 cột tile (trục X)
Giá trị	Ý nghĩa
0	Ô trống (không vẽ gì)
1	Tile rắn (CollisionBlock) — trong collisions.js
2	Platform (one-way) — trong collisions.js
18	Gem thu thập — trong l_Gems.js
Số khác	Tile trang trí — trong các file layer khác
Kích thước: 143 cột × ~64 hàng × 16 px/tile = 2288 × 1024 px thế giới game

2. Hệ thống Tileset — Cách tile ID được ánh xạ ra ảnh
Tileset là gì?
Một file ảnh .png chứa nhiều tile nhỏ xếp cạnh nhau:


┌────┬────┬────┬────┬────┐
│ 1  │ 2  │ 3  │ 4  │ 5  │  ...
├────┼────┼────┼────┼────┤
│ 6  │ 7  │ 8  │ 9  │ 10 │  ...
└────┴────┴────┴────┴────┘
Công thức tính vị trí tile trong ảnh

// index.js - hàm renderLayer()
const srcX = ((symbol - 1) % (tilesetImage.width / tileSize)) * tileSize
const srcY =   Math.floor((symbol - 1) / (tilesetImage.width / tileSize)) * tileSize
Giải thích từng bước:


symbol = tile ID trong mảng 2D (bắt đầu từ 1)

Bước 1: symbol - 1
  → Chuyển sang chỉ số 0-based (tile 1 → index 0, tile 2 → index 1)

Bước 2: Chia cho số tile trên 1 hàng
  tilesPerRow = tilesetImage.width / tileSize
  → 128px / 16px = 8 tiles/hàng

Bước 3: srcX = index % tilesPerRow * tileSize
  → Vị trí cột (px) trong ảnh gốc

Bước 4: srcY = floor(index / tilesPerRow) * tileSize
  → Vị trí hàng (px) trong ảnh gốc
Ví dụ cụ thể

Giả sử tileset rộng 128px, mỗi tile 16px → 8 tiles/hàng

symbol = 9
  → index = 9 - 1 = 8
  → srcX = (8 % 8) * 16 = 0
  → srcY = floor(8 / 8) * 16 = 16
  → Lấy tile ở hàng 2, cột 1 (ô thứ 9 trong tileset)

symbol = 15
  → index = 14
  → srcX = (14 % 8) * 16 = 6 * 16 = 96
  → srcY = floor(14 / 8) * 16 = 1 * 16 = 16
  → Lấy tile ở hàng 2, cột 7 (ô thứ 15)
Vẽ tile ra canvas

context.drawImage(
  tilesetImage,       // Ảnh nguồn
  srcX, srcY,         // Tọa độ cắt trong ảnh gốc
  tileSize, tileSize, // Kích thước cắt
  x * 16, y * 16,     // Vị trí vẽ trên canvas (world coords)
  16, 16              // Kích thước vẽ trên canvas
)
Mỗi tile được cắt ra từ ảnh tileset và dán vào đúng vị trí trên canvas theo tọa độ thế giới.

3. Tất cả các Layer và thứ tự render
Thứ tự nạp script trong index.html

<!-- Layer nền xa nhất (nạp TRƯỚC) -->
<script src="./data/l_New_Layer_1.js"></script>  <!-- ocean/water -->
<script src="./data/l_New_Layer_2.js"></script>  <!-- bramble/foreground -->
<script src="./data/l_New_Layer_8.js"></script>  <!-- midground tiles -->
<script src="./data/l_Back_Tiles.js"></script>   <!-- nền chính -->
<script src="./data/l_Decorations.js"></script>  <!-- trang trí -->
<script src="./data/l_Front_Tiles.js"></script>  <!-- tile phía trước -->
<script src="./data/l_Shrooms.js"></script>      <!-- nấm -->
<script src="./data/l_Collisions.js"></script>   <!-- decorative coll -->
<script src="./data/l_Grass.js"></script>         <!-- cỏ -->
<script src="./data/l_Trees.js"></script>        <!-- cây -->

<!-- Objects -->
<script src="./data/l_Gems.js"></script>
<script src="./data/collisions.js"></script>     <!-- tile rắn -->

<!-- Logic -->
<script src="./js/utils.js"></script>
Thứ tự vẽ lên canvas (back → front)

┌─────────────────────────────────────┐
│  Layer 1: l_New_Layer_1 (ocean)    │  ← Xa nhất, decorations.png
│  Layer 2: l_New_Layer_2 (bramble)  │    decorations.png
│  Layer 3: layersData (8 layers)   │  ← tileset.png + decorations.png
│  Layer 4: Player, Enemies, Gems    │  ← Entity động
│  Layer 5: UI (Hearts, Gem count)   │  ← Gần nhất, fixed coords
└─────────────────────────────────────┘
Chi tiết từng layer
Layer	Tileset	Kích thước tile	Tile IDs thường gặp
l_New_Layer_1	decorations.png	16px	981–1109 (ocean/water)
l_New_Layer_2	decorations.png	16px	bramble (bụi gai)
l_New_Layer_8	tileset.png	16px	nền giữa
l_Back_Tiles	tileset.png	16px	1, 3, 5, 9, 10, 11, 47, 49, 143, 306, 310, 314...
l_Decorations	decorations.png	16px	cây, hoa, đá...
l_Front_Tiles	tileset.png	16px	tile phía trước nền
l_Shrooms	decorations.png	16px	nấm trang trí
l_Collisions	decorations.png	16px	decorative collision
l_Grass	tileset.png	16px	cỏ
l_Trees	decorations.png	16px	cây lớn
4. Kỹ thuật Pre-rendering (Tối ưu hóa)
Vấn đề
Vẽ tất cả tile mỗi frame = rất tốn performance (143×64 = ~9000 tiles × nhiều layers).

Giải pháp: Render một lần, dùng mãi mãi

// index.js
async function renderStaticLayers(layersData) {
  // 1. Tạo 1 offscreen canvas
  const offscreenCanvas = document.createElement('canvas')
  offscreenCanvas.width  = canvas.width   // 2048
  offscreenCanvas.height = canvas.height // 1152
  const offscreenContext = offscreenCanvas.getContext('2d')

  // 2. Vẽ tất cả layer vào offscreen canvas (CHỈ 1 LẦN)
  for (const [layerName, tilesData] of Object.entries(layersData)) {
    const tilesetImage = await loadImage(tilesetInfo.imageUrl)
    renderLayer(tilesData, tilesetImage, tileSize, offscreenContext)
  }

  // 3. Trả về canvas đã vẽ sẵn
  return offscreenCanvas
}

// 4. Mỗi frame: chỉ cần drawImage cái canvas đó
c.drawImage(oceanBackgroundCanvas, 0, 0)  // Nhanh!
c.drawImage(backgroundCanvas, 0, 0)        // Nhanh!
Sơ đồ pre-rendering

Khởi động game (1 lần):
  Lặp qua tất cả layer
    Lặp qua tất cả tile trong layer
      Cắt tile từ tileset image
      Dán vào offscreen canvas
    → Kết quả: 3 offscreen canvas (ocean, bramble, background)
                    ↓
Mỗi frame (60 lần/giây):
  c.drawImage(oceanBackgroundCanvas)    ← Chỉ 1 lệnh!
  c.drawImage(brambleBackgroundCanvas)  ← Chỉ 1 lệnh!
  c.drawImage(backgroundCanvas)         ← Chỉ 1 lệnh!
  draw Player, enemies, gems             ← Chỉ entity động
Tiết kiệm: Thay vì vẽ ~9000 tiles mỗi frame, chỉ vẽ 3 canvas pre-rendered + các entity động. Rất nhiều tileset 2D game dùng kỹ thuật này.

5. Collision Blocks vs Platforms
collisions.js — Xây dựng từ tile map

collisions.forEach((row, y) => {
  row.forEach((symbol, x) => {
    if (symbol === 1) {
      // Tile rắn → CollisionBlock
      collisionBlocks.push(new CollisionBlock({
        x: x * 16,  // pixel X
        y: y * 16,  // pixel Y
        size: 16,
      }))
    } else if (symbol === 2) {
      // Platform một chiều → Platform
      platforms.push(new Platform({
        x: x * 16,
        y: y * 16 + 16,   // ← offset +16! đáy platform trùng đỉnh tile
        width: 16,
        height: 4,         // ← chỉ cao 4px
      }))
    }
  })
})
Sự khác biệt CollisionBlock vs Platform

CollisionBlock (symbol = 1):
┌──────────────────────────────────┐
│  16×16 px — RẮN TOÀN BỘ         │
│  ████████████████████████████   │
│  ████████████████████████████   │
│  ████ Player BỊ CHẶN █████████  │ ← Không đi qua được
│  ████████████████████████████   │
└──────────────────────────────────┘

Platform (symbol = 2):
┌──────────────────────────────────┐
│  ████ ĐÁY PLATFORM ████████     │ ← Chỉ 4px cao
│       (y = tileY + 16)          │
│                                  │
│  ↓ Player có thể NHẢY LÊN       │
│  ↑ Player có thể ĐI QUA         │ ← Từ dưới lên
└──────────────────────────────────┘
Logic va chạm Platform

// Platform.js
checkCollision(player, deltaTime) {
  return (
    // Player ĐANG Ở TRÊN platform
    player.y + player.height <= this.y &&
    // Và sẽ CHẠM platform trong frame này
    player.y + player.height + player.velocity.y * deltaTime >= this.y &&
    // Và overlap ngang
    player.x + player.width > this.x &&
    player.x < this.x + this.width
  )
}
Điều kiện quan trọng: player.y + player.height <= this.y — Player phải đang bên trên mới được đáp. Nếu đang bên trong platform (từ bên hông) → không có gì xảy ra.

6. Vị trí gem trong l_Gems.js

// init() xử lý gem layer
if (symbol === 18) {
  gems.push(new Sprite({
    x: x * 16,
    y: y * 16,
    ...
  }))
}
Cấu trúc l_Gems.js

Mỗi số 18 = 1 viên gem tại vị trí (x*16, y*16)

Row 4: ...0,0,0,18,18,18,0,0,0...     ← 3 gem liền nhau (vị trí x=18,19,20)
Row 9: ...18,0,18...                  ← 2 gem riêng
Row 13: ...18,0,0,18,0,18...           ← gem cách nhau
Row 15: ...0,0,0,0,0,0,18,18,18,18,18  ← 5 gem liền (vị trí x=60-64)
7. Tổng hợp luồng hiển thị tile map

index.html load
    │
    ├─ Nạp tất cả data/*.js (mảng 2D tile IDs)
    │
startRendering()
    │
    ├─ renderStaticLayers(oceanLayerData)
    │     → Duyệt l_New_Layer_1 (tile IDs ~981-1109)
    │     → Mỗi tile: cắt từ decorations.png → dán vào offscreen canvas
    │     → oceanBackgroundCanvas ✓
    │
    ├─ renderStaticLayers(brambleLayerData)
    │     → Duyệt l_New_Layer_2
    │     → brambleBackgroundCanvas ✓
    │
    └─ renderStaticLayers(layersData)
          → Duyệt 8 layers (l_New_Layer_8 → l_Trees)
          → Mỗi tile: cắt từ tileset.png / decorations.png → dán
          → backgroundCanvas ✓

animate() [mỗi frame]
    │
    c.save()
    c.scale(dpr+1, dpr+1)          ← Scale canvas lên 3×
    c.translate(-camera.x, camera.y) ← Camera offset
    │
    c.drawImage(oceanBackgroundCanvas, 0, 0)   ← Layer nền ocean
    c.drawImage(brambleBackgroundCanvas, 0, 0) ← Layer bramble
    c.drawImage(backgroundCanvas, 0, 0)        ← 8 layers còn lại
    │
    player.draw(c)                               ← Entity động
    oposums.forEach(o => o.draw(c))
    eagles.forEach(e => e.draw(c))
    gems.forEach(g => g.draw(c))
    sprites.forEach(s => s.draw(c))
    │
    c.restore()
    │
    c.save()
    c.scale(dpr+1, dpr+1)
    hearts.forEach(h => h.draw(c))   ← UI (fixed coords, không camera)
    gemUI.draw(c)
    c.fillText(gemCount, 33, 46)
    c.restore()
    │
    requestAnimationFrame(animate)
8. Camera và Tile Map
Camera chỉ ảnh hưởng đến các entity động + phần lớn tile. Lưu ý:


c.translate(-camera.x, camera.y)  // ← Canvas dịch ngược hướng camera
Các layer độc lập với camera:

UI (hearts, gemUI, fillText) → được restore() rồi save() lại, không bị translate
Kích thước camera giới hạn:


VIEW_WIDTH  = 1024
VIEW_HEIGHT = 576
WORLD_WIDTH  = 143 * 16 = 2288
WORLD_HEIGHT = 64  * 16 = 1024

MAX_CAMERA_X = 2288 - 1024 = 1264  ← Camera không lộn sang phải
MAX_CAMERA_Y = 1024 - 576  = 448   ← Camera không lộn xuống dưới
Camera chỉ cuộn trong phạm vi bản đồ, không bao giờ lộn ra ngoài.