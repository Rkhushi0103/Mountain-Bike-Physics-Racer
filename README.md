# Mountain-Bike-Physics-Racer
🚗 Physics Racer  Side-view physics car (like Hill Climb Racing)  Realistic physics (gravity, torque, suspension)  Animated terrain with collision  Extra tech  Parallax scrolling  Settings UI  Save player progress
#code section 
import pygame
import math
import json
import os

pygame.init()

# Screen setup
WIDTH, HEIGHT = 1000, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Bike Physics Racer")

clock = pygame.time.Clock()

# Colors
WHITE = (255, 255, 255)
GREEN = (50, 200, 50)
BLACK = (0, 0, 0)

# Load Background Image
bg_image = pygame.image.load("mountains.png")
bg_image = pygame.transform.scale(bg_image, (WIDTH, HEIGHT))

# Physics variables
gravity = 0.5
bike_x = 200
bike_y = 300
bike_vel_y = 0
wheel_angle = 0
speed = 0

# Terrain
terrain_points = []
for i in range(0, 5000, 100):
    height = HEIGHT - 150 + math.sin(i * 0.01) * 50
    terrain_points.append((i, height))

# Parallax background offset
bg_offset = 0

# Save system
save_file = "progress.json"

def save_progress(distance):
    with open(save_file, "w") as f:
        json.dump({"distance": distance}, f)

def load_progress():
    if os.path.exists(save_file):
        with open(save_file, "r") as f:
            return json.load(f)["distance"]
    return 0

distance_travelled = load_progress()

# UI
font = pygame.font.SysFont("Arial", 24)
show_settings = False

running = True
while running:
    clock.tick(60)

    # Draw scrolling background
    bg_offset -= speed * 0.3
    screen.blit(bg_image, (bg_offset % WIDTH - WIDTH, 0))
    screen.blit(bg_image, (bg_offset % WIDTH, 0))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            save_progress(distance_travelled)
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_s:
                show_settings = not show_settings

    keys = pygame.key.get_pressed()

    # Controls
    if keys[pygame.K_RIGHT]:
        speed += 0.2
    if keys[pygame.K_LEFT]:
        speed -= 0.2

    # Apply friction
    speed *= 0.98

    # Gravity
    bike_vel_y += gravity
    bike_y += bike_vel_y

    # Terrain collision
    terrain_height = HEIGHT - 150 + math.sin((bike_x + distance_travelled) * 0.01) * 50
    if bike_y > terrain_height - 30:
        bike_y = terrain_height - 30
        bike_vel_y = 0

    # Move forward
    distance_travelled += speed

    # Draw terrain
    shifted_points = []
    for x, y in terrain_points:
        shifted_points.append((x - distance_travelled, y))
    pygame.draw.lines(screen, GREEN, False, shifted_points, 5)

    # Draw bike body (triangle style)
    bike_body = [
        (bike_x + 20, bike_y - 20),
        (bike_x + 60, bike_y - 20),
        (bike_x + 40, bike_y - 50)
    ]
    pygame.draw.polygon(screen, BLACK, bike_body)

    # Rotate wheels
    wheel_angle += speed * 5
    wheel_radius = 18

    for wheel_x in [bike_x + 25, bike_x + 55]:
        pygame.draw.circle(screen, BLACK, (int(wheel_x), int(bike_y)), wheel_radius)

        # Wheel spoke animation
        line_x = wheel_x + math.cos(math.radians(wheel_angle)) * wheel_radius
        line_y = bike_y + math.sin(math.radians(wheel_angle)) * wheel_radius
        pygame.draw.line(screen, WHITE, (wheel_x, bike_y), (line_x, line_y), 3)

    # UI display
    distance_text = font.render(f"Distance: {int(distance_travelled)}", True, BLACK)
    screen.blit(distance_text, (20, 20))

    settings_text = font.render("Press S for Settings", True, BLACK)
    screen.blit(settings_text, (20, 50))

    if show_settings:
        pygame.draw.rect(screen, WHITE, (300, 150, 400, 300))
        pygame.draw.rect(screen, BLACK, (300, 150, 400, 300), 3)
        text = font.render("Settings Menu (Press S to close)", True, BLACK)
        screen.blit(text, (320, 200))

    pygame.display.flip()

pygame.quit()
