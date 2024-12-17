# dance-to-forget-website
import pygame
import sys
import random
import time
from pygame.locals import *
# Initialize Pygame
pygame.init()
# Game Constants
SCREEN_WIDTH, SCREEN_HEIGHT = 800, 600
FPS = 60
NIGHT_DURATION = 300  # 5 minutes per night
TOTAL_NIGHTS = 30
# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
BLUE = (100, 100, 255)
# Fonts
pygame.font.init()
FONT = pygame.font.SysFont("Arial", 30)
BIG_FONT = pygame.font.SysFont("Arial", 60, bold=True)
# Screen Setup
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Dance to Forget")
# Load Sounds (add your sound files in the same directory)
scream_sound = pygame.mixer.Sound("scream.wav")
ambient_music = pygame.mixer.Sound("ambient.wav")
ending_music = pygame.mixer.Sound("ending.wav")

# Load Sprites
girl_sprite = pygame.image.load("girl.png")
girl_sprite = pygame.transform.scale(girl_sprite, (100, 100))
monster_sprite = pygame.image.load("monster.png")
monster_sprite = pygame.transform.scale(monster_sprite, (80, 80))

# Game Variables
player_position = [SCREEN_WIDTH // 2, SCREEN_HEIGHT - 100]
monsters = []
health = 100
current_night = 1
game_over = False
ending_choice = None

# Functions
def show_text(text, size, position, color):
    font = pygame.font.SysFont("Arial", size)
    text_surface = font.render(text, True, color)
    screen.blit(text_surface, position)

def draw_health_bar():
    pygame.draw.rect(screen, RED, (20, 20, health * 2, 20))
    pygame.draw.rect(screen, WHITE, (20, 20, 200, 20), 2)

def spawn_monsters(night):
    monsters.clear()
    for _ in range(night * 3):
        x = random.randint(50, SCREEN_WIDTH - 50)
        y = random.randint(-800, -50)
        monsters.append([x, y])

def monster_logic():
    global health, game_over
    for monster in monsters:
        monster[1] += 2 + current_night * 0.3
        if monster[1] > SCREEN_HEIGHT:
            monster[1] = random.randint(-500, -50)
            monster[0] = random.randint(50, SCREEN_WIDTH - 50)

        screen.blit(monster_sprite, (monster[0], monster[1]))
        # Collision check
        if abs(player_position[0] - monster[0]) < 50 and abs(player_position[1] - monster[1]) < 50:
            health -= 5
            if health <= 0:
                game_over = True
def ending_screen():
    global ending_choice
    screen.fill(BLACK)
    show_text("STOP DANCING (S) or KEEP DANCING (K)?", 40, (100, SCREEN_HEIGHT // 2), WHITE)
    pygame.display.update()
    while True:
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            if event.type == KEYDOWN:
                if event.key == K_s:
                    ending_choice = "stop"
                    return
                elif event.key == K_k:
                    ending_choice = "keep"
                    return
def main():
    global health, current_night, game_over
    clock = pygame.time.Clock()
    start_time = time.time()
    pygame.mixer.Sound.play(ambient_music, loops=-1)
    spawn_monsters(current_night)
    while current_night <= TOTAL_NIGHTS:
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
        # Player Movement
        keys = pygame.key.get_pressed()
        if keys[K_LEFT] and player_position[0] > 50:
            player_position[0] -= 5
        if keys[K_RIGHT] and player_position[0] < SCREEN_WIDTH - 50:
            player_position[0] += 5
        # Update Screen
        screen.fill(BLACK)
        draw_health_bar()
        monster_logic()
        screen.blit(girl_sprite, (player_position[0], player_position[1]))
        show_text(f"Night: {current_night}/30", 30, (600, 20), WHITE)
        # Night Progress
        elapsed_time = time.time() - start_time
        if elapsed_time > NIGHT_DURATION:
            current_night += 1
            spawn_monsters(current_night)
            health = min(100, health + 20)
            start_time = time.time()
        if game_over:
            show_text("GAME OVER", 50, (SCREEN_WIDTH // 3, SCREEN_HEIGHT // 2), RED)
            pygame.display.update()
            pygame.time.wait(3000)
            pygame.quit()
            sys.exit()
        pygame.display.update()
        clock.tick(FPS)
    ending_screen()
    if ending_choice == "stop":
        show_text("The girl rests peacefully. GOOD ENDING.", 40, (100, SCREEN_HEIGHT // 2), BLUE)
    else:
        show_text("The girl danced until her death. BAD ENDING.", 40, (100, SCREEN_HEIGHT // 2), RED)
    pygame.display.update()
    pygame.time.wait(5000)
    pygame.quit()
if __name__ == "__main__":
    main()
    girl.png: The little girl sprite.
monster.png: Monster sprite.
scream.wav: Monster screech sound.
ambient.wav: Creepy background music.
ending.wav: Ending music.