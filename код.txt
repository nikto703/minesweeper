import pygame
import random
import sys

# Инициализация Pygame
pygame.init()

# Размеры окна
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
ROWS = 0
COLS = 0

# Цвета
BLACK = (0, 255, 0)
WHITE = (255, 255, 255)
GRAY = (128, 128, 128)
RED = (255, 0, 0)

# Размеры клетки
CELL_SIZE = 40

# Создание окна
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Сапер")

# Загрузка изображений и изменение их размеров
background_img = pygame.image.load("background.png")
background_img = pygame.transform.scale(background_img, (SCREEN_WIDTH, SCREEN_HEIGHT))
mine_img = pygame.image.load("mine.png")
mine_img = pygame.transform.scale(mine_img, (CELL_SIZE, CELL_SIZE))
win_img = pygame.image.load("win.png")
win_img = pygame.transform.scale(win_img, (300, 200))
lose_img = pygame.image.load("lose.png")
lose_img = pygame.transform.scale(lose_img, (300, 200))

# Шрифты
font = pygame.font.Font(None, 36)
small_font = pygame.font.Font(None, 24)

# Уровни сложности
LEVELS = {
    "Легкий": (10, 10, 10),
    "Средний": (20, 15, 40),
    "Сложный": (25, 20, 50)
}
def set_level(level):
    global ROWS, COLS, MINES
    if level == "easy":
        ROWS = 9
        COLS = 9
        MINES = 10
    elif level == "medium":
        ROWS = 16
        COLS = 16
        MINES = 40
    elif level == "hard":
        ROWS = 16
        COLS = 30
        MINES = 99

def draw_background():
    screen.blit(background_img, (0, 0))

def draw_window(x, y, text):
    text_surface = font.render(text, True, BLACK)
    text_rect = text_surface.get_rect(center=(x + 150, y + 50))
    screen.blit(text_surface, text_rect)

def draw_grid(grid, revealed):
    
    for row in range(len(grid)):
        for col in range(len(grid[0])):
            cell_x = col * CELL_SIZE
            cell_y = row * CELL_SIZE

            if revealed[row][col]:
                pygame.draw.rect(screen, (200, 200, 200), (cell_x, cell_y, CELL_SIZE, CELL_SIZE))
                pygame.draw.rect(screen, BLACK, (cell_x, cell_y, CELL_SIZE, CELL_SIZE), 1)

                if grid[row][col] == "M":
                    screen.blit(mine_img, (cell_x + 4, cell_y + 4))
                elif grid[row][col] != 0:
                    text = str(count_adjacent_mines(grid, row, col))
                    text_surface = small_font.render(text, True, BLACK)
                    text_rect = text_surface.get_rect(center=(cell_x + CELL_SIZE // 2, cell_y + CELL_SIZE // 2))
                    screen.blit(text_surface, text_rect)
            else:
                pygame.draw.rect(screen, GRAY, (cell_x, cell_y, CELL_SIZE, CELL_SIZE))
                pygame.draw.rect(screen, BLACK, (cell_x, cell_y, CELL_SIZE, CELL_SIZE), 1)

# Функция для подсчета количества мин рядом с клеткой
def count_adjacent_mines(grid, row, col):
    count = 0
    for dx in [-1, 0, 1]:
        for dy in [-1, 0, 1]:
            if dx == 0 and dy == 0:
                continue
            new_row = row + dy
            new_col = col + dx
            if 0 <= new_row < len(grid) and 0 <= new_col < len(grid[0]) and grid[new_row][new_col] == "M":
                count += 1
    return count

def count_neighboring_mines(grid, row, col):
    count = 0
    for i in range(max(0, row - 1), min(row + 2, len(grid))):
        for j in range(max(0, col - 1), min(col + 2, len(grid[0]))):
            if grid[i][j] == "M":
                count += 1
    return count


def reveal_cells(grid, revealed, row, col):
    if revealed[row][col]:
        return

    revealed[row][col] = True

    if grid[row][col] == 0:
        for dx in [-1, 0, 1]:
            for dy in [-1, 0, 1]:
                if dx == 0 and dy == 0:
                    continue
                new_row = row + dy
                new_col = col + dx
                if 0 <= new_row < len(grid) and 0 <= new_col < len(grid[0]):
                    reveal_cells(grid, revealed, new_row, new_col)

def check_win(grid, revealed):
    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] != "M" and not revealed[i][j]:
                return False
    return True

def game_over():
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
                
        draw_background()
        screen.blit(lose_img, (250, 200))
        pygame.display.flip()

def game_won():
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
                
        draw_background()
        screen.blit(win_img, (250, 200))
        pygame.display.flip()

def main():
    # Меню выбора уровня сложности
    selected_level = None
    while selected_level is None:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                mouse_pos = pygame.mouse.get_pos()
                if 250 <= mouse_pos[0] <= 550 and 250 + 50 <= mouse_pos[1] <= 300 +50:
                    selected_level = "Легкий"
                elif 250 <= mouse_pos[0] <= 550 and 325 +50 <= mouse_pos[1] <= 375 + 50:
                    selected_level = "Средний"
                elif 250 <= mouse_pos[0] <= 550+50 and 400 + 50<= mouse_pos[1] <= 450 + 50:
                    selected_level = "Сложный"
        
        draw_background()
        draw_window(250, 200, "Выберите уровень сложности:")
        draw_window(250, 275, "Легкий")
        draw_window(250, 350, "Средний")
        draw_window(250, 425, "Сложный")
        pygame.display.flip()
    
    rows, cols, num_mines = LEVELS[selected_level]

    grid = [[""] * cols for _ in range(rows)]
    revealed = [[False] * cols for _ in range(rows)]
    
    game_over_flag = False
    game_won_flag = False
    first_click = True
    
    while not game_over_flag and not game_won_flag:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                mouse_pos = pygame.mouse.get_pos()
                col = mouse_pos[0] // CELL_SIZE
                row = mouse_pos[1] // CELL_SIZE

                if row < 0 or row >= rows or col < 0 or col >= cols:
                    continue

                if first_click:
                    # Расстановка мин и открытие клеток после первого нажатия
                    mines_placed = 0
                    while mines_placed < num_mines:
                        rand_row = random.randint(0, rows - 1)
                        rand_col = random.randint(0, cols - 1)
                        if (rand_row != row or rand_col != col) and grid[rand_row][rand_col] != "M":
                            grid[rand_row][rand_col] = "M"
                            mines_placed += 1
                    
                    first_click = False

                if grid[row][col] == "M":
                    game_over_flag = True
                else:
                    reveal_cells(grid, revealed, row, col)
                    if check_win(grid, revealed):
                        game_won_flag = True

        
        draw_background()
        draw_grid(grid, revealed)
        
        for i in range(len(revealed)):
            for j in range(len(revealed[0])):
                if revealed[i][j]:
                    text = str(grid[i][j]) if grid[i][j] != 0 else ""
                    text_surface = small_font.render(text, True, BLACK)
                    text_rect = text_surface.get_rect(center=(j * CELL_SIZE + CELL_SIZE // 2,
                                                               i * CELL_SIZE + CELL_SIZE // 2))
                    screen.blit(text_surface, text_rect)
        
        pygame.display.flip()
        
    if game_over_flag:
        game_over()
    elif game_won_flag:
        game_won()

if __name__ == "__main__":
    main()
