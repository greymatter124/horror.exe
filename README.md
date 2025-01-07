import pygame
import random

pygame.init()

width = 800
height = 600
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("Virus Escape")

# Colors
black = (0, 0, 0)
white = (255, 255, 255)
red = (255, 0, 0)
gray = (100, 100, 100)

# "Desktop" elements
icons = [
    pygame.Rect(50, 50, 64, 64),
    pygame.Rect(150, 50, 64, 64),
    pygame.Rect(50, 150, 64, 64),
]

# Virus properties
virus_rect = pygame.Rect(width // 2, height // 2, 32, 32)
virus_speed = 5

# Game state
scare_timer = 0
scare_interval = random.randint(5000, 10000)  # 5-10 seconds
game_over = False
glitching = False  # Use a boolean for glitch state
glitch_duration = 1000  # 1 second glitch duration
glitch_start_time = 0

# Load image
try:
    virus_image = pygame.image.load("virus.png").convert_alpha()
    virus_image = pygame.transform.scale(virus_image, (virus_rect.width, virus_rect.height))
except pygame.error:
    print("Error loading image. Make sure 'virus.png' is in the same directory.")
    virus_image = None

clock = pygame.time.Clock() #Create a clock object

running = True
while running:
    dt = clock.tick(60) / 1000 #dt is delta time
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    if not game_over:
        # Virus movement (using delta time for smoother movement)
        virus_rect.x += random.randint(-virus_speed, virus_speed) * dt * 60
        virus_rect.y += random.randint(-virus_speed, virus_speed) * dt * 60

        virus_rect.clamp_ip(screen.get_rect())

        scare_timer += dt * 1000 #Multiply by 1000 to convert to miliseconds

        if scare_timer >= scare_interval:
            game_over = True
            glitching = True
            glitch_start_time = pygame.time.get_ticks()
            scare_timer = 0
            scare_interval = random.randint(5000, 10000)

        screen.fill(gray)

        for icon in icons:
            pygame.draw.rect(screen, white, icon)

        if virus_image:
            screen.blit(virus_image, virus_rect)
        else:
            pygame.draw.rect(screen, red, virus_rect)

        if glitching:
            for _ in range(10):
                x1 = random.randint(0, width)
                y1 = random.randint(0, height)
                x2 = random.randint(0, width)
                y2 = random.randint(0, height)
                pygame.draw.line(screen, red, (x1, y1), (x2, y2), 3)
            
            if pygame.time.get_ticks() - glitch_start_time >= glitch_duration:
                glitching = False

    else:
        screen.fill(black)
        font = pygame.font.Font(None, 72)
        text = font.render("SYSTEM ERROR", True, red)
        text_rect = text.get_rect(center=(width // 2, height // 2))
        screen.blit(text, text_rect)

    pygame.display.flip()

pygame.quit()
