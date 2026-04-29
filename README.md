import pygame
import random
from enum import Enum
from collections import deque

class Direction(Enum):
    UP = (0, -1)
    DOWN = (0, 1)
    LEFT = (-1, 0)
    RIGHT = (1, 0)

class SnakeGame:
    def __init__(self, width=800, height=600, grid_size=20):
        """Initialize the Snake game"""
        pygame.init()
        
        self.width = width
        self.height = height
        self.grid_size = grid_size
        self.grid_width = width // grid_size
        self.grid_height = height // grid_size
        
        self.screen = pygame.display.set_mode((width, height))
        pygame.display.set_caption("Snake Game")
        self.clock = pygame.time.Clock()
        self.fps = 10
        
        # Colors
        self.BLACK = (0, 0, 0)
        self.GREEN = (0, 255, 0)
        self.RED = (255, 0, 0)
        self.YELLOW = (255, 255, 0)
        self.GRAY = (50, 50, 50)
        
        self.reset_game()
    
    def reset_game(self):
        """Reset game state"""
        # Initialize snake in the middle
        start_x = self.grid_width // 2
        start_y = self.grid_height // 2
        self.snake = deque([
            (start_x, start_y),
            (start_x - 1, start_y),
            (start_x - 2, start_y)
        ])
        
        self.direction = Direction.RIGHT
        self.next_direction = Direction.RIGHT
        self.food = self.spawn_food()
        self.score = 0
        self.game_over = False
    
    def spawn_food(self):
        """Spawn food at a random location"""
        while True:
            x = random.randint(0, self.grid_width - 1)
            y = random.randint(0, self.grid_height - 1)
            if (x, y) not in self.snake:
                return (x, y)
    
    def handle_events(self):
        """Handle user input and window events"""
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return False
            
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP and self.direction != Direction.DOWN:
                    self.next_direction = Direction.UP
                elif event.key == pygame.K_DOWN and self.direction != Direction.UP:
                    self.next_direction = Direction.DOWN
                elif event.key == pygame.K_LEFT and self.direction != Direction.RIGHT:
                    self.next_direction = Direction.LEFT
                elif event.key == pygame.K_RIGHT and self.direction != Direction.LEFT:
                    self.next_direction = Direction.RIGHT
                elif event.key == pygame.K_SPACE and self.game_over:
                    self.reset_game()
        
        return True
    
    def update(self):
        """Update game state"""
        if self.game_over:
            return
        
        self.direction = self.next_direction
        
        # Calculate new head position
        head_x, head_y = self.snake[0]
        dx, dy = self.direction.value
        new_head = (head_x + dx, head_y + dy)
        
        # Check collision with walls
        if (new_head[0] < 0 or new_head[0] >= self.grid_width or
            new_head[1] < 0 or new_head[1] >= self.grid_height):
            self.game_over = True
            return
        
        # Check collision with itself
        if new_head in self.snake:
            self.game_over = True
            return
        
        # Add new head
        self.snake.appendleft(new_head)
        
        # Check if food is eaten
        if new_head == self.food:
            self.score += 10
            self.food = self.spawn_food()
        else:
            # Remove tail if no food eaten
            self.snake.pop()
    
    def draw(self):
        """Draw game elements"""
        self.screen.fill(self.BLACK)
        
        # Draw grid
        for x in range(0, self.width, self.grid_size):
            pygame.draw.line(self.screen, self.GRAY, (x, 0), (x, self.height), 1)
        for y in range(0, self.height, self.grid_size):
            pygame.draw.line(self.screen, self.GRAY, (0, y), (self.width, y), 1)
        
        # Draw snake
        for i, (x, y) in enumerate(self.snake):
            color = self.GREEN if i == 0 else (0, 200, 0)  # Head lighter
            pygame.draw.rect(
                self.screen,
                color,
                (x * self.grid_size + 1, y * self.grid_size + 1,
                 self.grid_size - 2, self.grid_size - 2)
            )
        
        # Draw food
        food_x, food_y = self.food
        pygame.draw.rect(
            self.screen,
            self.RED,
            (food_x * self.grid_size + 1, food_y * self.grid_size + 1,
             self.grid_size - 2, self.grid_size - 2)
        )
        
        # Draw score
        font = pygame.font.Font(None, 36)
        score_text = font.render(f"Score: {self.score}", True, self.YELLOW)
        self.screen.blit(score_text, (10, 10))
        
        # Draw game over message
        if self.game_over:
            font_large = pygame.font.Font(None, 72)
            game_over_text = font_large.render("GAME OVER", True, self.RED)
            
            font_small = pygame.font.Font(None, 36)
            restart_text = font_small.render("Press SPACE to restart", True, self.YELLOW)
            
            text_rect = game_over_text.get_rect(
                center=(self.width // 2, self.height // 2 - 40)
            )
            restart_rect = restart_text.get_rect(
                center=(self.width // 2, self.height // 2 + 40)
            )
            
            self.screen.blit(game_over_text, text_rect)
            self.screen.blit(restart_text, restart_rect)
        
        pygame.display.flip()
    
    def run(self):
        """Main game loop"""
        running = True
        while running:
            running = self.handle_events()
            self.update()
            self.draw()
            self.clock.tick(self.fps)
        
        pygame.quit()

if __name__ == "__main__":
    game = SnakeGame()
    game.run()

