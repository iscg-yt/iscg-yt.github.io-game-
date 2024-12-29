import pygame
import sys

# 初始化 Pygame
pygame.init()

# 視窗設定
screen_width, screen_height = 1080, 2400
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("自訂圖片與全域拖曳")

# 顏色設定
WHITE = (255, 255, 255)

# 載入自訂圖片
player_image = pygame.image.load("IMG_20240607_182905_800.jpg")  # 替換為你的圖片路徑
player_image = pygame.transform.scale(player_image, (50, 50))  # 縮放圖片為 50x50 大小
player_size = 50  # 圖片大小
player_x = screen_width // 2  # 初始 X 座標
player_y = screen_height // 2  # 初始 Y 座標

# 拖曳狀態
dragging = False

# 遊戲主循環
clock = pygame.time.Clock()

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        # 滑鼠按下：無論點擊哪裡都啟動拖曳
        if event.type == pygame.MOUSEBUTTONDOWN:
            dragging = True  # 開始拖曳
        # 滑鼠移動：拖曳中時更新方塊位置
        if event.type == pygame.MOUSEMOTION and dragging:
            mouse_x, mouse_y = event.pos
            player_x = mouse_x - player_size // 2  # 更新方塊 X 座標
            player_y = mouse_y - player_size // 2  # 更新方塊 Y 座標
        # 滑鼠釋放：結束拖曳
        if event.type == pygame.MOUSEBUTTONUP:
            dragging = False  # 停止拖曳
    # 邊界檢查
    player_x = max(0, min(screen_width - player_size, player_x))
    player_y = max(0, min(screen_height - player_size, player_y))
    # 畫面更新
    screen.fill(WHITE)  # 清空畫面
    screen.blit(player_image, (player_x, player_y))  # 繪製自訂圖片
    pygame.display.flip()  # 更新畫面
    # 控制遊戲速度
    clock.tick(30)
