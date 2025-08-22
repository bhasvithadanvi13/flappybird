# flappybird
import pygame
import random
import sys
import os

# Initialize pygame
pygame.init()

# Game constants
SCREEN_WIDTH = 400
SCREEN_HEIGHT = 600
GRAVITY = 0.25
FLAP_STRENGTH = -7
PIPE_SPEED = 3
PIPE_GAP = 150
PIPE_FREQUENCY = 1500  # milliseconds
FLOOR_HEIGHT = 100

# Colors
SKY_BLUE = (113, 197, 207)
WHITE = (255, 255, 255)
GREEN = (0, 128, 0)
BROWN = (139, 69, 19)
BLACK = (0, 0, 0)
YELLOW = (255, 255, 0)
ORANGE = (255, 165, 0)

# Set up the display
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()

# Font
font = pygame.font.SysFont('Arial', 30, bold=True)
large_font = pygame.font.SysFont('Arial', 50, bold=True)

class Bird:
    def __init__(self):
        self.x = 100
        self.y = SCREEN_HEIGHT // 2
        self.velocity = 0
        self.width = 40
        self.height = 30
        self.alive = True
        self.rotation = 0
        
    def draw(self):
        # Create a surface for the bird
        bird_surface = pygame.Surface((self.width, self.height), pygame.SRCALPHA)
        
        # Draw bird body (circle)
        pygame.draw.circle(bird_surface, YELLOW, (self.width//2, self.height//2), self.height//2)
        
        # Draw bird eye
        pygame.draw.circle(bird_surface, BLACK, (self.width//2 + 8, self.height//2 - 5), 4)
        
        # Draw bird beak
        pygame.draw.polygon(bird_surface, ORANGE, [(self.width - 5, self.height//2), 
                                                 (self.width + 10, self.height//2 - 5),
                                                 (self.width + 10, self.height//2 + 5)])
        
        # Rotate the bird based on velocity
        self.rotation = max(-30, min(self.velocity * 2, 70))
        rotated_bird = pygame.transform.rotate(bird_surface, -self.rotation)
        
        # Draw the rotated bird
        screen.blit(rotated_bird, (self.x - rotated_bird.get_width()//2, self.y - rotated_bird.get_height()//2))
    
    def update(self):
        # Apply gravity
        self.velocity += GRAVITY
        self.y += self.velocity
        
        # Prevent bird from going off the top of the screen
        if self.y < 0:
            self.y = 0
            self.velocity = 0
    
    def flap(self):
        self.velocity = FLAP_STRENGTH
    
    def get_mask(self):
        return pygame.Rect(self.x - self.width//2, self.y - self.height//2, self.width, self.height)

class Pipe:
    def __init__(self):
        self.x = SCREEN_WIDTH
        self.height = random.randint(100, 400)
        self.top_pipe = pygame.Rect(self.x, 0, 60, self.height)
        self.bottom_pipe = pygame.Rect(self.x, self.height + PIPE_GAP, 60, SCREEN_HEIGHT)
        self.passed = False
    
    def draw(self):
        # Draw top pipe
        pygame.draw.rect(screen, GREEN, self.top_pipe)
        pygame.draw.rect(screen, (0, 100, 0), (self.top_pipe.x - 2, self.top_pipe.y - 20, 64, 20))
        
        # Draw bottom pipe
        pygame.draw.rect(screen, GREEN, self.bottom_pipe)
        pygame.draw.rect(screen, (0, 100, 0), (self.bottom_pipe.x - 2, self.bottom_pipe.y, 64, 20))
    
    def update(self):
        self.x -= PIPE_SPEED
        self.top_pipe.x = self.x
        self.bottom_pipe.x = self.x
    
    def collide(self, bird_rect):
        return self.top_pipe.colliderect(bird_rect) or self.bottom_pipe.colliderect(bird_rect)

class Game:
    def __init__(self):
        self.bird = Bird()
        self.pipes = []
        self.score = 0
        self.last_pipe = pygame.time.get_ticks()
        self.game_state = "start"  # start, playing, game_over
        self.floor_scroll = 0
        self.floor_speed = 2
        
    def draw_floor(self):
        # Draw the floor
        pygame.draw.rect(screen, BROWN, (0, SCREEN_HEIGHT - FLOOR_HEIGHT, SCREEN_WIDTH, FLOOR_HEIGHT))
        
        # Draw floor pattern
        for i in range(20):
            pygame.draw.rect(screen, (160, 82, 45), 
                            (i * 40 - self.floor_scroll % 40, SCREEN_HEIGHT - FLOOR_HEIGHT, 20, 20))
    
    def update_floor(self):
        self.floor_scroll += self.floor_speed
    
    def draw_background(self):
        # Draw sky
        screen.fill(SKY_BLUE)
        
        # Draw clouds
        for i in range(5):
            x = (i * 100 + self.floor_scroll // 3) % (SCREEN_WIDTH + 100) - 50
            y = 100 + (i * 30) % 200
            pygame.draw.circle(screen, WHITE, (x, y), 20)
            pygame.draw.circle(screen, WHITE, (x + 15, y - 10), 15)
            pygame.draw.circle(screen, WHITE, (x + 15, y + 10), 15)
    
    def draw_start_screen(self):
        self.draw_background()
        
        # Draw title
        title = large_font.render("FLAPPY BIRD", True, WHITE)
        screen.blit(title, (SCREEN_WIDTH//2 - title.get_width()//2, 150))
        
        # Draw instructions
        instruction = font.render("Press SPACE to start", True, WHITE)
        screen.blit(instruction, (SCREEN_WIDTH//2 - instruction.get_width()//2, 250))
        
        # Draw bird
        self.bird.draw()
        
        self.draw_floor()
    
    def draw_game_over(self):
        # Draw game over text
        game_over_text = large_font.render("GAME OVER", True, WHITE)
        screen.blit(game_over_text, (SCREEN_WIDTH//2 - game_over_text.get_width()//2, 150))
        
        # Draw score
        score_text = font.render(f"Score: {self.score}", True, WHITE)
        screen.blit(score_text, (SCREEN_WIDTH//2 - score_text.get_width()//2, 220))
        
        # Draw restart instruction
        restart_text = font.render("Press SPACE to restart", True, WHITE)
        screen.blit(restart_text, (SCREEN_WIDTH//2 - restart_text.get_width()//2, 280))
    
    def draw_score(self):
        score_text = font.render(str(self.score), True, WHITE)
        screen.blit(score_text, (SCREEN_WIDTH//2 - score_text.get_width()//2, 50))
    
    def update(self):
        if self.game_state == "playing":
            # Update bird
            self.bird.update()
            
            # Update floor
            self.update_floor()
            
            # Generate new pipes
            current_time = pygame.time.get_ticks()
            if current_time - self.last_pipe > PIPE_FREQUENCY:
                self.pipes.append(Pipe())
                self.last_pipe = current_time
            
            # Update pipes and check for collisions
            for pipe in self.pipes:
                pipe.update()
                
                # Check if bird passed the pipe
                if not pipe.passed and pipe.x < self.bird.x:
                    pipe.passed = True
                    self.score += 1
                
                # Check for collisions
                if pipe.collide(self.bird.get_mask()):
                    self.bird.alive = False
                    self.game_state = "game_over"
            
            # Remove pipes that are off screen
            self.pipes = [pipe for pipe in self.pipes if pipe.x > -60]
            
            # Check if bird hit the ground or ceiling
            if self.bird.y >= SCREEN_HEIGHT - FLOOR_HEIGHT or self.bird.y <= 0:
                self.bird.alive = False
                self.game_state = "game_over"
    
    def draw(self):
        self.draw_background()
        
        # Draw pipes
        for pipe in self.pipes:
            pipe.draw()
        
        # Draw bird
        self.bird.draw()
        
        # Draw floor
        self.draw_floor()
        
        # Draw score
        if self.game_state == "playing":
            self.draw_score()
        
        # Draw game over screen
        if self.game_state == "game_over":
            self.draw_game_over()
        
        # Draw start screen
        if self.game_state == "start":
            self.draw_start_screen()
    
    def reset(self):
        self.bird = Bird()
        self.pipes = []
        self.score = 0
        self.last_pipe = pygame.time.get_ticks()
        self.game_state = "start"
        self.floor_scroll = 0

# Create game instance
game = Game()

# Game loop
running = True
while running:
    # Handle events
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                if game.game_state == "start":
                    game.game_state = "playing"
                elif game.game_state == "playing":
                    game.bird.flap()
                elif game.game_state == "game_over":
                    game.reset()
    
    # Update game state
    game.update()
    
    # Draw everything
    game.draw()
    
    # Update display
    pygame.display.update()
    
    # Cap the frame rate
    clock.tick(60)

pygame.quit()
sys.exit()
