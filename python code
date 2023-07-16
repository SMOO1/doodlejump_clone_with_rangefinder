```python
import random
import pygame
import RPi.GPIO as GPIO
import time

pygame.init()

#Define window
win_size = (960, 780)
screen = pygame.display.set_mode(win_size)

#Game character properties
char_pos = [win_size[0] / 2, win_size[1] / 2]
char_vel = [0, 0]
char_size = 30
char_color = (255, 255, 255)
char_jump_height = 10
char_max_jump = False

#Platform properties
platform_width = 100
platform_height = 20
platform_color = (0, 255, 0)
platform_gap = 200
platform_min_height = 50
platform_max_height = 200
platforms = []

#Jump booster properties
jump_booster_size = 20
jump_booster_color = (255, 0, 255)
jump_booster_quantity = 1
jump_boosters = []

altitude = 0
altitude_font = pygame.font.Font(None, 30)
altitude_text = altitude_font.render("Altitude: " + str(altitude), True, (255, 255, 255))

#range text
range_font = pygame.font.Font(None, 30)
range_text = range_font.render("Range: ", True, (255, 255, 255))

#ultrasonic rangefinder pins
TRIG_PIN = 18
ECHO_PIN = 16
GPIO.setmode(GPIO.BOARD)
GPIO.setup(TRIG_PIN, GPIO.OUT)
GPIO.setup(ECHO_PIN, GPIO.IN)

#start platform
start_platform = pygame.Rect(char_pos[0] - platform_width / 2, char_pos[1] + char_size / 2, platform_width,
                             platform_height)
platforms.append(start_platform)

# Generate initial platformas and jump boosters
for i in range(6):
    platform_x = random.randint(0, win_size[0] - platform_width)
    platform_y = i * platform_gap + random.randint(platform_min_height, platform_max_height)
    platforms.append(pygame.Rect(platform_x, platform_y, platform_width, platform_height))

for _ in range(jump_booster_quantity):
    booster_x = random.randint(0, win_size[0] - jump_booster_size)
    booster_y = random.randint(platform_min_height, platform_max_height)
    jump_boosters.append(pygame.Rect(booster_x, booster_y, jump_booster_size, jump_booster_size))

max_altitude = char_pos[1]

original_platform_pos = start_platform.bottom
platform_generate_threshold = 100
gravity = 0.07
jump_velocity = -6

#game loop
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            quit()

    #distance from rangefinder
    GPIO.output(TRIG_PIN, True)
    time.sleep(0.0001)
    GPIO.output(TRIG_PIN, False)

    while GPIO.input(ECHO_PIN) == 0:
        pulse_start = time.time()

    while GPIO.input(ECHO_PIN) == 1:
        pulse_end = time.time()

    pulse_duration = pulse_end - pulse_start
    distance = round(pulse_duration * 34300 / 2, 2)  # speed of sound is 34300 cm/s

        # change character position based on the rangefinder vlaue
    if distance < win_size[0]:
        if distance <= 25:
            char_vel[0] = 1  #move left
        else:
            char_vel[0] = -1  #move right
    else:
        char_vel[0] = 0



    #character position updating

    char_pos[1] += char_vel[1]
    char_pos[0] += char_vel[0] * 5
    #updating the og platform position
    camera_offset = char_pos[1] - win_size[1] / 2
    original_platform_pos -= char_vel[1]
    char_pos[1] -= camera_offset
    for platform in platforms:
        platform.y -= camera_offset
    for booster in jump_boosters:
        booster.y -= camera_offset

    #gravity effect
    char_vel[1] += gravity

    #jump condition
    collision = False
    for platform in platforms:
        if char_vel[1] > 0 and char_pos[1] + char_size / 2 < platform.bottom and char_pos[
            1] + char_size / 2 + char_vel[1] >= platform.top and char_pos[0] + char_size / 2 > platform.left and char_pos[
            0] - char_size / 2 < platform.right:
            char_pos[1] = platform.top - char_size / 2
            char_vel[1] = -6
            collision = True
            break

    if not collision:
        char_max_jump = False

    if len(platforms) > 0:
        platform_generate_threshold = char_pos[1] - original_platform_pos + platform_gap

    #altitude acalculation (based off of the og platform position)
    altitude = int(original_platform_pos - char_pos[1])

    # create new platforms/jump boosters if the player's altitude is greater than the threshold
    if altitude >= platform_generate_threshold:
        platform_x = random.randint(0, win_size[0] - platform_width)
        platform_y = platforms[0].top - platform_gap - random.randint(platform_min_height, platform_max_height)
        platforms.insert(0, pygame.Rect(platform_x, platform_y, platform_width, platform_height))

        #generate more boosters
        for _ in range(jump_booster_quantity):
            booster_x = random.randint(0, win_size[0] - jump_booster_size)
            booster_y = platforms[0].top - platform_gap - random.randint(platform_min_height, platform_max_height)
            jump_boosters.append(pygame.Rect(booster_x, booster_y, jump_booster_size, jump_booster_size))

    #delete off screen platforms
    platforms = [platform for platform in platforms if platform.top < win_size[1]]
    jump_boosters = [booster for booster in jump_boosters if booster.top < win_size[1]]

    #check for collisions with jump boosters
    for booster in jump_boosters:
        if char_pos[1] + char_size / 2 > booster.top and char_pos[1] - char_size / 2 < booster.bottom and char_pos[
            0] + char_size / 2 > booster.left and char_pos[0] - char_size / 2 < booster.right:
            char_vel[1] = -13


    screen.fill((0, 0, 0))

    #draw all game elements
    for platform in platforms:
        pygame.draw.rect(screen, platform_color, platform)

    for booster in jump_boosters:
        pygame.draw.rect(screen, jump_booster_color, booster)

    pygame.draw.rect(screen, char_color,
                     pygame.Rect(char_pos[0] - char_size / 2, char_pos[1] - char_size / 2, char_size, char_size))

    altitude_text = altitude_font.render("Altitude: " + str(altitude), True, (255, 255, 255))
    screen.blit(altitude_text, (win_size[0] - altitude_text.get_width() - 10, 10))
    range_text = range_font.render("Range: " + str(distance), True, (255, 255, 255))
    screen.blit(range_text, (10, 10))
    pygame.display.update()

    pygame.time.wait(10)
```
