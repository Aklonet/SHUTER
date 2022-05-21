#from pygame import *
from random import randint
from time import time as timer

font.init()
font2 = font.Font(None, 60)
win = font2.render('Вы перешли на светлую сторону!', True, (0, 250, 0))
lose = font2.render('Вы перешли на темную сторану!', True,(250, 0, 0))

# нам нужны такие картинки:
img_back = "galaxy.jpg" # фон игры 
img_hero = "rocket.png" # герой
img_enemy = "ufo.png" # Враг 
img_bullet = "bullet.png" # Патрон
img_ast = "asteroid.png"

num_fire = 0
rel_time = False
score = 0
max_lost = 15
lost = 0
goal = 15

# класс-родитель для других спрайтов
class GameSprite(sprite.Sprite):
  # конструктор класса
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        # Вызываем конструктор класса (Sprite):
        sprite.Sprite.__init__(self)

        # каждый спрайт должен хранить свойство image - изображение
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.speed = player_speed

        # каждый спрайт должен хранить свойство rect - прямоугольник, в который он вписан
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
 
  # метод, отрисовывающий героя на окне
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Asteroid(GameSprite):
    def update(self):
        global lost
        self.rect.y += self.speed
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            lost = lost + 0

# класс главного игрока
class Player(GameSprite):
    # метод для управления спрайтом стрелками клавиатуры
    def update(self):
        keys = key.get_pressed()
        if keys[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_RIGHT] and self.rect.x < win_width - 80:
            self.rect.x += self.speed
        # метод "выстрел" (используем место игрока, чтобы создать там пулю)
    def fire(self):
        bullet = Bullet(img_bullet, self.rect.centerx, self.rect.top, 15, 20, -15)
        bullets.add(bullet)

lost = 0

class Enemy(GameSprite):
    def update(self):
        global lost
        self.rect.y += self.speed
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            lost = lost + 1

class Bullet(GameSprite):
    def update(self):
        self.rect.y += self.speed
        if self.rect.y < 0:
            self.kill()

# Создаем окошко
win_width = 1200
win_height = 600
display.set_caption("звёздные воёны на минималках")
window = display.set_mode((win_width, win_height))
background = transform.scale(image.load(img_back), (win_width, win_height))

# создаем спрайты
ship = Player(img_hero, 5, win_height - 100, 80, 100, 10)
monsters = sprite.Group()
for i in range(1, 6):
    monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
    monsters.add(monster)

asteroids = sprite.Group()
for i in range(1, 3):
    asteroid = Asteroid(img_ast, randint(30, win_width - 30), -40, 80, 50, randint(1, 7))
    asteroids.add(asteroid)

bullets = sprite.Group()
# переменная "игра закончилась": как только там True, в основном цикле перестают работать спрайты
finish = False
# Основной цикл игры:
run = True # флаг сбрасывается кнопкой закрытия окна
while run:
    # событие нажатия на кнопку Закрыть
    for e in event.get():
        if e.type == QUIT:
            run = False
        #keys = key.get_pressed()
        #if keys[K_LCTRL] and finish == True:
            #finish = False
        elif e.type == KEYDOWN:
            if e.key == K_SPACE:

                if num_fire < 5 and rel_time == False:
                    num_fire = num_fire + 1
                    ship.fire()
                if num_fire >= 5 and rel_time == False:
                    last_time = timer()
                    rel_time = True

    if not finish:
        # обновляем фон
        window.blit(background,(0,0))

        text = font2.render("Счёт: " + str(score), 1, (255, 255, 255))
        window.blit(text, (15, 50))

        text_lose = font2.render("Пропущено: " + str(lost), 1, (255, 255, 255))
        window.blit(text_lose, (10, 100))

        if rel_time == True:
            now_time = timer()
            if now_time - last_time < 3:
                reload = font2.render('Перезарядка!', 1, (150, 60, 0))
                window.blit(reload, (10, 150))
            else:
                num_fire = 0 
                rel_time = False

        # производим движения спрайтов
        ship.update()
        monsters.update()
        bullets.update()
        asteroids.update()

        # обновляем их в новом местоположении при каждой итерации цикла
        ship.reset()
        monsters.draw(window)
        bullets.draw(window)
        asteroids.draw(window)

        collides = sprite_list = sprite.groupcollide(monsters, bullets, True, True)
        for c in collides:
            score = score+1

            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)

        if sprite.spritecollide(ship, monsters, False) or lost >= max_lost:
            finish = True
            window.blit(lose, (450, 300))
        
        if sprite.spritecollide(ship, asteroids, False):
            finish = True
            window.blit(lose, (450, 300))

        if score >= goal:
            finish = True
            window.blit(win, (450, 300))

        display.update()
    # цикл срабатывает каждую 0.05 секунд
    time.delay(20)
