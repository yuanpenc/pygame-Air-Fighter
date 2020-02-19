# pygame-Air-Fighter（飞机大战）

### [Demo video](https://youtu.be/obxlqFeqS0w)

## Project description

My project is about a game. The name of the game is Air Fighter. I will use pygame to build a 1D game like Raiden Fighters or 1942. The whole game is about an excellent fighter plane want to go to the destination but must break through the enemies encircling. We should control the plane to attack the enemies as well as not to be attacked by the enemies’ planes.

Competitive Analysis
Similarities: There are many similar games on the internet, as I mention above, 1942 and Raiden Fighter. We all build on 1D surface. Player will control hero plane to defeat the boss to get to the destination. All these games have tuns of enemies to attack and bullets to avoid, which is the fun that this pattern of game bring to us.
Differences: The unique feature of my Air Fighter compared with these old plane fighter games is different mode, money system, various weapons (attacking mode), and smarter enemies (from easy to hard).

## code for the game
note: picture, gif, music mention in the code should be placed in the same folder with python code below.

```
import pygame
import sys
import math

#images of planes,sounds, bullets and background are download from 
# https://blog.csdn.net/mbl114/article/details/78074688
#boss image downloaded from
#https://scratch.mit.edu/studios/551638/
#skill circle image downloaded from
#https://picsart.com/i/sticker-neon-round-blue-freetoedit-circle-frame-border-287072299003211

#boss from google
class Boss(pygame.sprite.Sprite):
    def __init__(self,mainWindow,pos):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.pos=pos
        self.image=pygame.image.load("pic.png").convert()
        self.rect=self.image.get_rect()
        self.imageResize=pygame.transform.scale(self.image,\
                                (self.rect.width,self.rect.height))
        self.image=self.imageResize
        black=(0,0,0)
        self.image.set_colorkey(black)
        self.rect=self.image.get_rect()
        self.rect.centerx,self.rect.centery=self.pos
        self.initHp()
        # create its bullets 
        self.bulletList=[]
        self.interval=100
    
    def initHp(self):
        self.fullHp=100
        self.curHp=100
        self.hpPercent=self.curHp/self.fullHp
        self.bossDie=False
    
    def beAttacked(self):
        if self.curHp>=1:
            self.curHp-=1
            self.hpPercent=self.curHp/self.fullHp
        else:
            self.bossDie=True


    def move(self):
        if self.rect.centery<self.mainWindow.height//4:
            self.rect.centery+=1
        if self.rect.centery>=self.mainWindow.height//4:
            if len(self.bulletList)==0:
                Bx=self.rect.centerx
                By=self.rect.centery+self.rect.height//2
                self.bulletList.append(Bullet_boss(self.mainWindow,Bx,By,angle=-math.pi/2))
            else:
                previousBullet=self.bulletList[-1]
                Bx=self.rect.centerx
                By=self.rect.centery+self.rect.height//2
                Px=previousBullet.rect.centerx
                Py=previousBullet.rect.centery
                distance=self.mainWindow.getDistance(Bx,By,Px,Py)
                if distance>self.interval:
                    self.bulletList.append(Bullet_boss(self.mainWindow,Bx,By,angle=-math.pi/2))
    
    def drawBloodBar(self):
        #blood bar frame
        left=self.rect.left
        top=self.rect.top-20
        width=self.rect.width
        height=10
        dark=(51,25,0)
        red=(255,0,0)
        pygame.draw.rect(self.mainWindow.myScene,dark,(left,top,width,height),2)
        
        #blood bar
        height_b=height-1
        top_b=top+1
        left_b=left+1
        width_b=(width-1)*self.hpPercent
        yellow=(225,225,102)
        pygame.draw.rect(self.mainWindow.myScene,yellow,(left_b,top_b,width_b,height_b))
        

    def draw(self):
        self.mainWindow.myScene.blit(self.image,self.rect)
        self.drawBloodBar()
        for bullets in self.bulletList:
            bullets.drawBullets()
    
class Boss_2(Boss):
    def __init__(self,mainWindow,pos):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.pos=pos
        self.image=pygame.image.load("boss2.png").convert()
        self.rect=self.image.get_rect()
        self.imageResize=pygame.transform.scale(self.image,\
                                (self.rect.width,self.rect.height-80))
        self.image=self.imageResize
        black=(0,0,0)
        self.image.set_colorkey(black)
        self.rect=self.image.get_rect()
        self.rect.centerx,self.rect.centery=self.pos
        self.initHp()
        #create its bullets
        self.bulletList=[]
        self.interval=100
        self.bulletList_split=[]
        

    @staticmethod
    def getAngle(hx,hy,bx,by):
        if bx==hx:#verticle
            return -math.pi/2
        slope=(hy-by)/(hx-bx)
        angle=math.atan(slope)
        if hy<by:
            if slope<0:
                return angle
            elif slope>0:
                return angle-math.pi/2
        else:#hy>=by
            if slope>0:
                return angle
            elif slope==0:
                if hx>bx:return angle
                else: return math.pi
            else: #slope<0
                return angle+math.pi/2
                

    def initHp(self):
        self.fullHp=100
        self.curHp=100
        self.hpPercent=self.curHp/self.fullHp
        self.bossDie=False

    def move(self):
        if self.rect.centery<self.mainWindow.height//4:
            self.rect.centery+=1
        else:

            if len(self.bulletList)==0:
                Bx=self.rect.centerx
                By=self.rect.centery
                self.bulletList.append(Bullet_boss_angle(self.mainWindow,Bx,By,-math.pi/2))
            else:
                previousBullet=self.bulletList[-1]
                Bx=self.rect.centerx
                By=self.rect.centery
                Px=previousBullet.rect.centerx
                Py=previousBullet.rect.centery
                distance=self.mainWindow.getDistance(Bx,By,Px,Py)
                if distance>self.interval:
                    self.bulletList.append(Bullet_boss_angle(self.mainWindow,Bx,By,-math.pi/2))

            #split the bullet
            for bullet in self.bulletList:
                if bullet.rect.centery in [250,330,410,500,580,660]:
                    self.bulletList_split.append(Bullet_boss_split(self.mainWindow,bullet.rect.centerx,\
                                                bullet.rect.centery,0))
                    self.bulletList_split.append(Bullet_boss_split(self.mainWindow,bullet.rect.centerx,\
                                                bullet.rect.centery,math.pi))

    
    def drawBloodBar(self):
        #blood bar frame
        left=self.rect.left
        top=self.rect.top+10
        width=self.rect.width
        height=10
        dark=(51,25,0)
        red=(255,0,0)
        pygame.draw.rect(self.mainWindow.myScene,red,(left,top,width,height),2)
        
        #blood bar
        height_b=height-1
        top_b=top+1
        left_b=left+1
        width_b=(width-1)*self.hpPercent
        yellow=(225,225,102)
        pygame.draw.rect(self.mainWindow.myScene,yellow,(left_b,top_b,width_b,height_b))

    def draw(self):
        self.mainWindow.myScene.blit(self.image,self.rect)
        self.drawBloodBar()
        for bullets in self.bulletList:
            bullets.drawBullets()
        for bullet_split in self.bulletList_split:
            bullet_split.drawBullets()
    
        

class Enemy_Yellow(pygame.sprite.Sprite):
    def __init__(self,mainWindow,pos):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.pos=pos
        self.image=pygame.image.load("Picture2.png").convert()
        self.rect=self.image.get_rect()
        self.rect.centery=self.pos[1]
        self.rect.centerx=self.pos[0]
        self.margin=300
    
    def move(self):
        self.rect.centery+=1
        if self.rect.top>self.mainWindow.height:
            self.rect.top=self.pos[1]-300
    
    @staticmethod
    #copy from https://blog.csdn.net/hailler1119/article/details/89195558
    def pascal(n):
        triangle = [[0 for i in range(j + 1)] for j in range(n)]
        for i in range(0, n):
            triangle[i][0] = 1
            triangle[i][i] = 1
        for row in range(1, n):
            for col in range(1, row):
                triangle[row][col] = triangle[row - 1][col - 1] + triangle[row - 1][col]
        return triangle

    #copy from https://blog.csdn.net/hailler1119/article/details/89195558
    def bezier(self,controlPoint,t):
        # sum of (a * (1 - t)^b * t^c * Pn)
        #a is some rows of Pascal triangle
        n=len(controlPoint)
        x=0
        y=0
        triangle_a=self.pascal(n)
        for v in range(0, n):
            c = v
            b = n - 1 - c
            a = triangle_a[n - 1][v]
            x += a * math.pow((1 - t), b) * math.pow(t, c) * controlPoint[v][0]
            y += a * math.pow((1 - t), b) * math.pow(t, c) * controlPoint[v][1]
        return x,y

    def draw(self):
        self.mainWindow.myScene.blit(self.image,self.rect)

class Enemy_Yellow_Circle(Enemy_Yellow):
    # this kind of plane can fly up and down or circle around
    def __init__(self,mainWindow,controlPoint,rate,startPos):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.image=pygame.image.load("Picture2.png").convert()
        self.rect=self.image.get_rect()
        self.imageResize=pygame.transform.scale(self.image,\
                                (self.rect.width-30,self.rect.height-20))
        self.image=self.imageResize
        self.rect=self.image.get_rect()
        self.rect.centerx,self.rect.centery=startPos
        self.controlPoint=controlPoint
        self.path=[]
        #get a path to make plane to show up in different time
        self.rate=rate
        self.pathOutOfScreen=[(self.rect.centerx,self.rect.centery)]
        self.pathOutOfScreen.append(self.controlPoint[0])
        for j in range (0,self.rate+1):
            time=j/self.rate
            point=self.bezier(self.pathOutOfScreen,time)
            self.path.append(point)
        self.interval=500
        pathInScreen=self.getPath()
        self.path+=pathInScreen
        self.moveCount=0
        self.reverse=True

    @staticmethod
    def cutList(L):# help us divide list into smaller ones by adjacent same coordinate
        indexL=[]
        for index in range (len(L)-2):
            if L[index]==L[index+1]:
                indexL.append(index)
        result=[]
        preIndex=0
        for index in indexL:
            temp_L=L[preIndex:index+1]
            result.append(temp_L)
            preIndex=index+1
        result.append(L[preIndex:])
        return result
    
    def getPath(self):
        pathPoint=self.cutList(self.controlPoint)
        path=[]
        listNum=len(pathPoint)
        intervalforEach=self.interval//listNum
        for pointList in pathPoint:
            
            temp_path=[]
            for i in range (0,intervalforEach+1):
                t=i/intervalforEach
                point=self.bezier(pointList,t)
                temp_path.append(point)
            path+=temp_path
        self.interval=intervalforEach*listNum # for movecount
        return path
    
    def move(self):
        # use moveCount to make sure plane only move once when move is called
        if self.reverse and (self.moveCount<self.interval+self.rate):
            self.rect.centerx=self.path[self.moveCount][0]
            self.rect.centery=self.path[self.moveCount][1]
            self.moveCount+=1
        if not self.reverse and (self.moveCount>=0):
            self.rect.centerx=self.path[self.moveCount][0]
            self.rect.centery=self.path[self.moveCount][1]
            self.moveCount-=1
        
        if self.moveCount>=self.interval+self.rate:
            self.reverse=False
        elif self.moveCount<0:
            self.reverse=True
        
class Enemy_Blue_level2(Enemy_Yellow):
    def __init__(self,mainWindow,startPos):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.image=pygame.image.load("Pictureblue.png").convert()
        self.rect=self.image.get_rect()
        self.image=pygame.transform.scale(self.image,\
                                (self.rect.width-30,self.rect.height-20))
        self.rect=self.image.get_rect()
        self.rect.centerx,self.rect.centery=startPos
        self.speed=1
    
    def move(self):
        dx, dy = self.rect.x - self.mainWindow.hero.rect.x, self.rect.y - self.mainWindow.hero.rect.y
        dist = math.hypot(dx, dy)
        dx, dy = dx/dist*1.5, dy/dist*1.5
        self.rect.x -= dx * self.speed
        self.rect.y -= dy * self.speed
    



    
class Enemy_Blue(Enemy_Yellow):
    def __init__(self,mainWindow,controlPoint,rate,startPos):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.image=pygame.image.load("Pictureblue.png").convert()
        self.rect=self.image.get_rect()
        self.rect.centerx,self.rect.centery=startPos
        self.controlPoint=controlPoint
        self.path=[]
        #get a path to make plane to show up in different time
        self.rate=rate
        self.pathOutOfScreen=[(self.rect.centerx,self.rect.centery)]
        self.pathOutOfScreen.append(self.controlPoint[0])
        for j in range (0,self.rate+1):
            time=j/self.rate
            point=self.bezier(self.pathOutOfScreen,time)
            self.path.append(point)
        # path on the screen
        self.interval=300
        for i in range (0,self.interval+1):
            t=i/self.interval
            point=self.bezier(self.controlPoint,t)
            self.path.append(point)
        self.moveCount=0 #this is the index of the path points
        
    def move(self):
        # use moveCount to make sure plane only move once when move is called
        if self.moveCount<self.interval+self.rate:
            self.rect.centerx=self.path[self.moveCount][0]
            self.rect.centery=self.path[self.moveCount][1]
            self.moveCount+=1
        if self.moveCount>=self.interval+self.rate:
            self.moveCount=0


class Bullet(pygame.sprite.Sprite):
    def __init__(self,mainWindow,x,y,angle=math.pi/2):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.x=x
        self.y=y
        #angle is designed for adjusting bullet shooting angle
        self.angle=angle
        self.image=pygame.image.load('Picture1.png').convert()
        #make image background transparent
        black=(0,0,0)
        self.image.set_colorkey(black)
        self.rect=self.image.get_rect()
        self.rect.centerx=self.x
        self.rect.centery=self.y
        self.speed=3

    def moveBullets(self):
        self.rect.centery-=self.speed*math.sin(self.angle)
        self.rect.centerx+=self.speed*math.cos(self.angle)
        if self.rect.bottom<0 or self.rect.top>self.mainWindow.height or \
            self.rect.left>self.mainWindow.width or self.rect.right<0:
            self.mainWindow.hero.bulletsList.remove(self)

    def drawBullets(self):
        self.mainWindow.myScene.blit(self.image,self.rect)
    
class Bullet_missile(Bullet):
    def __init__(self,mainWindow,x,y,angle=math.pi/2):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.x=x
        self.y=y
        self.angle=angle
        self.image=pygame.image.load('missile.png').convert()
        #make image background transparent
        black=(0,0,0)
        self.image.set_colorkey(black)
        self.rect=self.image.get_rect()
        self.rect.centerx=self.x
        self.rect.centery=self.y
        self.speed=3

class Bullet_boss(Bullet):
    def __init__(self,mainWindow,x,y,angle=math.pi/2):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.x=x
        self.y=y
        self.angle=angle
        self.image=pygame.image.load('bossbullet.png').convert()
        #make image background transparent
        black=(0,0,0)
        self.image.set_colorkey(black)
        self.rect=self.image.get_rect()
        self.rect.centerx=self.x
        self.rect.centery=self.y
        self.speed=1
    
    def moveBullets(self):
        self.rect.centery-=self.speed*math.sin(self.angle)
        self.rect.centerx+=self.speed*math.cos(self.angle)
        if self.rect.bottom<0 or self.rect.top>self.mainWindow.height or \
            self.rect.left>self.mainWindow.width or self.rect.right<0:
            self.mainWindow.boss_level_1.bulletList.remove(self)
    
class Bullet_boss_angle(Bullet_boss):
    def moveBullets(self):
        self.rect.centery-=math.floor((self.speed*math.sin(self.angle)))
        self.rect.centerx+=((self.speed*math.cos(self.angle)))
        if self.rect.bottom<0 or self.rect.top>self.mainWindow.height or \
            self.rect.left>self.mainWindow.width or self.rect.right<0:
            self.mainWindow.boss_level_2.bulletList.remove(self)

class Bullet_boss_split(Bullet_boss):
    def __init__(self,mainWindow,x,y,angle=math.pi/2):
        pygame.sprite.Sprite.__init__(self)
        self.mainWindow=mainWindow
        self.x=x
        self.y=y
        self.angle=angle
        self.image=pygame.image.load('bossbullet.png').convert()
        self.image=pygame.transform.rotate(self.image, 90-self.angle/math.pi*180)
        #make image background transparent
        black=(0,0,0)
        self.image.set_colorkey(black)
        self.rect=self.image.get_rect()
        self.rect.centerx=self.x
        self.rect.centery=self.y
        self.speed=1

    def moveBullets(self):
        self.rect.centery-=math.floor((self.speed*math.sin(self.angle)))
        self.rect.centerx+=((self.speed*math.cos(self.angle)))
        if self.rect.bottom<0 or self.rect.top>self.mainWindow.height or \
            self.rect.left>self.mainWindow.width or self.rect.right<0:
            self.mainWindow.boss_level_2.bulletList_split.remove(self)



class  PlayerPlane(pygame.sprite.Sprite):
    def __init__(self,mainWindow,missile=False):
        pygame.sprite.Sprite.__init__(self)
        #load the plane image
        self.image=pygame.image.load("Picture3.png").convert()
        self.mainWindow=mainWindow
        #get plane rectangle
        self.rect=self.image.get_rect()
        #initial the position
        self.rect.centerx=self.mainWindow.width//2
        marginFromTheBottomScreen=50
        self.rect.centery=self.mainWindow.height-marginFromTheBottomScreen
        #define the move speed
        self.speed=2
        #create its bullets 
        self.missile=missile
        self.bulletsList=pygame.sprite.Group()
        self.previousTime=None
        self.currentTime=None
        self.missilePreTime=0
        self.missileCurTime=0
        self.shootInterval=250
        self.missileInterval=800

    def move(self,key_press):
        if key_press[pygame.K_UP] and self.rect.y>0:
            self.rect.centery-=self.speed
        if key_press[pygame.K_RIGHT] and self.rect.right<self.mainWindow.width:
            self.rect.centerx+=self.speed
        if (key_press[pygame.K_DOWN] and \
                self.rect.bottom<self.mainWindow.height):
            self.rect.centery+=self.speed
        if key_press[pygame.K_LEFT] and self.rect.left>0:
            self.rect.centerx-=self.speed
        elif key_press[pygame.K_SPACE]:
            self.mainWindow.skill.visiable=True
            self.mainWindow.invincible=True
        if key_press[pygame.K_f]:
            if len(self.bulletsList)==0:
                #no bullets shot, create the first one in front of the plane
                Bx=self.rect.centerx
                By=self.rect.centery-self.rect.height//2
                self.bulletsList.add(Bullet(self.mainWindow,Bx,By))
                self.previousTime=self.mainWindow.gameTime
            else:# more than one bullets
                #only when previous bullets have some distances, next one can be 
                #generated
                self.currentTime=self.mainWindow.gameTime
                if self.currentTime-self.previousTime>self.shootInterval:
                    self.previousTime=self.currentTime
                    By=self.rect.centery-self.rect.height//2
                    Bx=self.rect.centerx
                    self.bulletsList.add(Bullet(self.mainWindow,Bx,By))
                #can be use after buy new weapon in the store
            if self.missile:
                self.missileCurTime=self.mainWindow.gameTime
                if self.missileCurTime-self.missilePreTime>self.missileInterval:
                    self.missilePreTime=self.missileCurTime
                    By=self.rect.centery-self.rect.height//2-30
                    Bx=self.rect.centerx
                    self.bulletsList.add(Bullet_missile(self.mainWindow,Bx,By))
            
    
    def draw(self):
        #draw the hero plane
        self.mainWindow.myScene.blit(self.image,self.rect)
        #draw the bullets if plane attacks
        for bullets in self.bulletsList:
            bullets.drawBullets()
        


class BackGround():
    def __init__(self,mainWindow):
        #load the background image
        self.image = pygame.image.load("img_bg_grassland.jpg").convert()
        self.imageResize=pygame.transform.scale(self.image,\
                                (mainWindow.width,mainWindow.height))
        self.mainWindow=mainWindow
        #the origin coordinates on the map(screen)
        self.y1=0
        #two picture scrolling one by one consist of entire map
        self.y2=-self.mainWindow.height
        self.speed=0.3
    
    def mapScroll(self):
        self.y1+=self.speed
        self.y2+=self.speed
        #when first picture get to the very bottom, wrap around to the beginning
        if self.y1>=self.mainWindow.height:
            self.y1=0
        #similarly, wrap around to the where picture 2 begin
        if self.y2>=0:
            self.y2=-self.mainWindow.height

    def draw(self):
        self.mainWindow.myScene.blit(self.imageResize,(0,self.y1))
        self.mainWindow.myScene.blit(self.imageResize,(0,self.y2))
    
class BackGround_2(BackGround):
    def __init__(self,mainWindow):
        super().__init__(mainWindow)
        self.image = pygame.image.load("img_bg_lava.jpg").convert()
        self.imageResize=pygame.transform.scale(self.image,\
                                (mainWindow.width,mainWindow.height))

class HeroBlood():
    def __init__(self,someWindow,pos):
        self.someWindow=someWindow
        self.image = pygame.image.load("picture4.jpg").convert()
        self.rect=self.image.get_rect()
        self.image=pygame.transform.scale(self.image,\
                                (self.rect.width//2,self.rect.height//2))
        white=(255,255,255)
        self.image.set_colorkey(white)
        self.rect=self.image.get_rect()
        self.rect.centerx,self.rect.centery=pos
    
    def draw(self):
        self.someWindow.myScene.blit(self.image,self.rect)

class Skill():
    def __init__(self,hero):
        self.hero=hero
        self.image=pygame.image.load("skill.png")
        self.image=pygame.transform.scale(self.image,\
                                (hero.rect.width+10,hero.rect.height+40))
        self.rect=self.image.get_rect()
        self.visiable=False
        self.count=0
    
    def move(self):
        self.rect.centerx=self.hero.rect.centerx
        self.rect.centery=self.hero.rect.centery
        self.count+=1
        if self.count>500:
            self.visiable=False
            self.count=0
            self.hero.mainWindow.invincible=False

    def draw(self):
        if self.visiable:
            self.hero.mainWindow.myScene.blit(self.image,self.rect)
    
#bomb code refers to https://blog.csdn.net/mbl114/article/details/78075947
class Bomb():
    def __init__(self,mainWindow):
        self.mainWindow=mainWindow
        # load bomb images
        self.imageList=[pygame.image.load("bomb-" + str(v) + ".png") for v in range(1, 6)]
        # set up bomb interval
        self.interval=10
        self.interval_index=0
        self.index=0
        #set up initial position
        self.left=0
        self.top=0
        self.visible=False

    def set_pos(self,x,y):
        self.left=x
        self.top=y
    
    def move(self):
        if not self.visible:
            return 
        self.interval_index+=1
        if self.interval_index<self.interval:
            return
        self.index+=1
        if self.index>=len(self.imageList):
            self.index=0
            self.visible=False

    def draw(self):
        if self.visible:
            self.mainWindow.myScene.blit(self.imageList[self.index],(self.left,self.top))



class SecondWindow():
    def __init__(self,mainWindow):
        self.mainWindow=mainWindow
        self.crimson=(220,20,60)
        self.black=(0,0,0)
        self.gold=(255,215,0)
        self.darkGreen=(0,100,0)
        self.run=True
        self.gameTime=None
        #buile the window with assigned width and height
        self.width=512
        self.height=650
        self.myScene = pygame.display.set_mode([self.width,self.height])
        pygame.display.set_caption("Air Fighter")
        #build the background
        self.backGround=BackGround_2(self)
        #create hero
        self.hero=PlayerPlane(self)
        self.invincible=self.mainWindow.invincible
        #create the boss
        self.init_boss()
        #create enemy plane blue level2
        self.enemy_blue_level2_group=pygame.sprite.Group()
        self.enemy_blue_level2_group.add(Enemy_Blue_level2(self,(-50,-50)))
        #inherit hero blood
        self.heroBloodList=self.mainWindow.heroBloodList
        # inherit score
        self.font=self.mainWindow.font
        self.score=8
        self.score_text=self.font.render(f"Your score: {self.score}",1,self.black)
        center=(20,self.height-20)
        self.score_textpos = self.score_text.get_rect(center=center)
        #bomb animation
        self.bombList=[Bomb(self) for _ in range(4)]
        #enemy plane yellow curve
        self.init_enemy_yellow_curve()
        #hero's skill circle
        self.skill=Skill(self.hero)

    
    def init_boss(self):
        marginOutOfScreen=400
        startPos=(self.width//2,self.height//4-marginOutOfScreen)
        self.boss_level_2=Boss_2(self,startPos)
    
    def init_enemy_yellow_curve(self):
        self.enemy_yGroup_curve=pygame.sprite.Group()
        r=130
        sx,sy=400,0 #the top point on the circle which is used to draw the arc
        controlPoints=[(600,sy),(sx,sy),\
                    (sx,sy),(sx-r,sy),(sx-r,sy+r),\
                    (sx-r,sy+r),(sx-r,sy+r*2),(sx,sy+2*r),\
                    (sx,sy+2*r),(sx+r/1.5,sy+2*r),(sx+r/1.5,sy+2*r+r/1.5),\
                    (sx+r/1.5,sy+2*r+r/1.5),(sx+r/1.5,sy+2*r+r/1.5*2),(sx,sy+2*r+r/1.5*2),\
                    (sx,sy+2*r+r/1.5*2),(sx-100,sy+2*r+r/1.5*2),\
                    (sx-100,sy+2*r+r/1.5*2),(sx-200,sy+2*r+r/1.5*2),\
                    (sx-200,sy+2*r+r/1.5*2),(sx-200-r/2,sy+2*r+r/1.5*2),(sx-200-r/2,sy+2*r+r/1.5*2-r/2),\
                    (sx-200-r/2,sy+2*r+r/1.5*2-r/2),(sx-200-r/2,sy+2*r+r/1.5*2-r),(sx-200,sy+2*r+r/1.5*2-r),\
                    (sx-200,sy+2*r+r/1.5*2-r),(sx-200,sy+2*r+r/1.5*2-r)]
        rate=70
        startPos=(800,sy)
        for i in range (5):
            rate+=i*20
            self.enemy_yGroup_curve.add(Enemy_Yellow_Circle(self,\
                                        controlPoints,rate,startPos))
    
    @staticmethod
    def getDistance(x1,y1,x2,y2):
        return ((x1-x2)**2+(y1-y2)**2)**0.5


    def updateScore(self):
        if self.score==8:
            self.score=self.mainWindow.score
        self.score+=1
        self.score_text=self.font.render(f"Your score: {self.score}",1,self.black)


    def key_control(self):
        key_press = pygame.key.get_pressed()
        if (key_press[pygame.K_UP] or key_press[pygame.K_DOWN] or \
            key_press[pygame.K_RIGHT] or key_press[pygame.K_LEFT] or \
            key_press[pygame.K_f]):
            self.hero.move(key_press)
    
    def map_control(self):
        self.backGround.mapScroll()
    
    def bullets_control(self):
        for bullet in self.hero.bulletsList:
            bullet.moveBullets()
        for bullet_boss_level2 in self.boss_level_2.bulletList:
            bullet_boss_level2.moveBullets()
        for bullet_boss_level2_split in self.boss_level_2.bulletList_split:
            bullet_boss_level2_split.moveBullets()

    def enemy_blue_level2_control(self):
        if self.gameTime>0:
            for e_b in self.enemy_blue_level2_group:
                e_b.move()
    
    def enemy_yellow_control(self):
        for e_y in self.enemy_yGroup_curve:
            e_y.move()

    def boss_level_2_control(self):
        if self.gameTime>40000:
            self.boss_level_2.move()
    
    def hero_skill_control(self):
        self.skill.move()
    
    def check_Invincible(self):
        #this part designed for test and demo
        if not self.invincible:
            self.heroBloodList.pop(0)

    def check_Collision(self):
        #hero and enemies
        for e_b in self.enemy_blue_level2_group:
            result=pygame.sprite.collide_rect_ratio(0.7)(self.hero,e_b)
            if result: 
                    # activate bomb animation
                    for bomb in self.bombList:
                        if not bomb.visible:
                            bomb.set_pos(e_b.rect.left,e_b.rect.top)
                            bomb.visible=True
                            break
                    self.enemy_blue_level2_group.remove(e_b)
                    self.updateScore()
                    self.check_Invincible()
        
        for e_y_c in self.enemy_yGroup_curve:
            result=pygame.sprite.collide_rect_ratio(0.7)(self.hero,e_y_c)
            if result: 
                for bomb in self.bombList:
                    if not bomb.visible:
                        bomb.set_pos(e_y_c.rect.left,e_y_c.rect.top)
                        bomb.visible=True
                        break
                self.enemy_yGroup_curve.remove(e_y_c)
                self.check_Invincible()
        
        #bullet and heroplane
        
        for bossBullet in self.boss_level_2.bulletList:
            result=pygame.sprite.collide_rect_ratio(0.8)(bossBullet,self.hero)
            if result:
                self.boss_level_2.bulletList.remove(bossBullet)
                self.check_Invincible()
        for bossBullet_split in self.boss_level_2.bulletList_split:
            result=pygame.sprite.collide_rect_ratio(0.8)(bossBullet_split,self.hero)
            if result:
                self.boss_level_2.bulletList_split.remove(bossBullet_split)
                self.check_Invincible()

        #bullets and all enemies
        for bullet in self.hero.bulletsList:
            for e_b in self.enemy_blue_level2_group:
                result=pygame.sprite.collide_rect_ratio(0.7)(bullet,e_b)
                if result: 
                    # activate bomb animation
                    for bomb in self.bombList:
                        if not bomb.visible:
                            bomb.set_pos(e_b.rect.left,e_b.rect.top)
                            bomb.visible=True
                            break
                    self.enemy_blue_level2_group.remove(e_b)
                    self.updateScore()
        
            for e_y_c in self.enemy_yGroup_curve:
                    result=pygame.sprite.collide_rect_ratio(0.8)(bullet,e_y_c)
                    if result:
                        for bomb in self.bombList:
                            if not bomb.visible:
                                bomb.set_pos(e_y_c.rect.left,e_y_c.rect.top)
                                bomb.visible=True
                                break
                        self.enemy_yGroup_curve.remove(e_y_c)
                        self.hero.bulletsList.remove(bullet)
                        self.updateScore()
            
            if pygame.sprite.collide_mask(bullet,self.boss_level_2):
                self.hero.bulletsList.remove(bullet)
                self.boss_level_2.beAttacked()
                if self.boss_level_2.bossDie:
                    self.score+=20
                    self.gameOver()
        
        #check for game over
        if len(self.heroBloodList)==0:
            self.gameOver()
    
    def bomb_level_2_control(self):
        for bomb in self.bombList:
            if bomb.visible:
                bomb.move()


    def redrawAll(self):
        #draw the map(background)
        self.backGround.draw()
        #draw the herp plane
        self.hero.draw()
        #draw the boss
        self.boss_level_2.draw()
        #draw enemy
        for e_b in self.enemy_blue_level2_group:
            e_b.draw()
        for e_y_c in self.enemy_yGroup_curve:
            e_y_c.draw()

        #draw the heroblood
        for blood in self.heroBloodList:
            blood.draw()
        #draw the hero's skill
        if self.skill.visiable:
            self.skill.draw()
        #draw the score
        self.myScene.blit(self.score_text,self.score_textpos)

        #draw bomb animation
        for bomb in self.bombList:
            if bomb.visible:
                bomb.draw()
        

    def runGame(self):
        #set up a way to quit the game
        pygame.init()
        while self.run:
            pygame.time.delay(10)
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.run=False
        #######game controler 
            #time
            self.gameTime=pygame.time.get_ticks()
            #print(self.gameTime)
            #check for keypressed
            self.key_control()
            #map itself timerfire
            self.map_control()
            # bullets timerfire
            self.bullets_control()
            #boss timerfire
            self.boss_level_2_control()
            #enemy timerfire
            self.enemy_blue_level2_control()
            self.enemy_yellow_control()
            #bomb animation timerfire
            self.bomb_level_2_control()
            #update the skill circle
            self.hero_skill_control()


        #######game checking
            self.check_Collision()
            
        #######game viewer
            self.redrawAll()
            
            #refresh the game 
            pygame.display.update()
        #quit the game
        pygame.quit()
    
    def gameOver(self):
        pygame.init()
        run=True
        self.myScene.fill(self.crimson)

        font=pygame.font.SysFont('msyh',56)
        text=font.render("GameOver",1,self.black)
        center=(self.width//2,self.height//3-30)
        textpos = text.get_rect(center=center)
        
        text_score=font.render(f"Your score:{self.score}",1,self.black)
        center_score=(self.width//2,self.height//2-30)
        text_s_pos=text_score.get_rect(center=center_score)
        while run:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    run=False
            self.myScene.blit(text, textpos)
            self.myScene.blit(text_score,text_s_pos)
            result=self.botton(self.width//2-50,self.height//3*2,100,50,self.gold,\
                            self.darkGreen,'Restart')
            if result:
                self.mainWindow.__init__()
                self.mainWindow.introGame()
            pygame.display.update()
        pygame.quit()

    def botton(self,left,top,width,height,color1,color2,text):
        
        #draw botton and check for click
        mouse=pygame.mouse.get_pos()
        click=pygame.mouse.get_pressed()
        pygame.draw.rect(self.myScene,color1,(left,top,width,height))
        if left<mouse[0]<left+width and top<mouse[1]<top+height:
            pygame.draw.rect(self.myScene,color2,(left,top,width,height))
            if click[0]==1:
                return True
        #draw text
        font=pygame.font.Font(None,25)
        text=font.render(text,1,self.black)
        center=(left+width/2,top+height/2)
        textpos = text.get_rect(center=center)
        self.myScene.blit(text, textpos)

class MainWindow():
    def __init__(self):
        pygame.init()
        self.invincible=False #for test
        self.run=True
        self.gameTime=None
        self.blue=(153,204,255)
        self.lightblue=(204,229,255)
        self.white=(255,255,255)
        self.black=(0,0,0)
        self.darkGreen=(0,100,0)
        self.lime=(0,255,0)
        self.gold=(255,215,0)
        self.crimson=(220,20,60)
        #buile the window with assigned width and height
        self.width=512
        self.height=650
        self.myScene = pygame.display.set_mode([self.width,self.height])
        #set up the title 
        pygame.display.set_caption("Air Fighter")
        #create score and money
        self.score=0
        self.money=0
        self.font=pygame.font.SysFont('msyh',20)
        self.score_text=self.font.render(f"Your score: {self.score}",1,self.black)
        center=(20,self.height-20)
        self.score_textpos = self.score_text.get_rect(center=center)
        #build the background
        self.backGround=BackGround(self)
        #create hero
        self.hero=PlayerPlane(self)
        #hero's blood
        self.init_heroBlood()
        #hero's skill
        self.skill=Skill(self.hero)
        ###create enemy
        self.init_enemy_blue()
        self.init_enemy_yellow_1()
        self.init_enemy_yellow_circle()

        #create the boss
        self.init_boss()
        #create the second level
        self.secondGame=SecondWindow(self)

        #create bomb animation
        #4 elements are to make sure multiple bomb animation existing simultaneously
        self.bombList=[Bomb(self) for _ in range(4)]

        #create music
        self.init_music()
    
    def init_music(self):
        file=r'bgm.mp3'
        track = pygame.mixer.music.load(file)
        pygame.mixer.music.play()

        
    
    def init_heroBlood(self):
        self.heroBlood=3
        self.heroBloodList=[]
        bloodX=self.width//3*2+30
        interval=50
        bloodY=self.height-20
        for i in range (3):
            pos=(bloodX+i*interval,bloodY)
            self.heroBloodList.append(HeroBlood(self,pos))

    def init_boss(self):
        marginOutOfScreen=400
        startPos=(self.width//2,self.height//4-marginOutOfScreen)
        self.boss_level_1=Boss(self,startPos)

    
    def init_enemy_yellow_1(self):
        #vertical fly pattern
        self.enemy_yGroup_1=pygame.sprite.Group()
        x,interval=self.width//3,60
        for pos in ((x,-interval),(x,-2*interval),\
                        (x,-3*interval),(x,-4*interval)):
            self.enemy_yellow=Enemy_Yellow(self,pos)
            self.enemy_yGroup_1.add(self.enemy_yellow)

        #diamond pattern
        interval=80
        startP=-500 #this parameter refers to the starting position of the plane
        posSet={(self.width//2-interval,0+startP),(self.width//2,interval+startP),\
                (self.width//2+interval,0+startP),(self.width//2,-interval+startP)}
        for pos in posSet:
            self.enemy_yGroup_1.add(Enemy_Yellow(self,pos))
    
    def init_enemy_yellow_circle(self):
        self.enemy_yGroup_circle=pygame.sprite.Group()
        r=150
        sx,sy=250,200 #the top point on the circle which is used to draw the arc
        controlPoints=[(0,sy),(sx,sy),\
                    (sx,sy),(sx+r,sy),(sx+r,sy+r),\
                    (sx+r,sy+r),(sx+r,sy+2*r),(sx,sy+2*r),\
                    (sx,sy+2*r),(sx-r,sy+2*r),(sx-r,sy+r),\
                    (sx-r,sy+r),(sx-r,sy),(sx,sy),\
                    (sx,sy),(self.width+100,sy)]
        rate=50
        startPos=(-100,sy)
        self.enemy_yGroup_circle.add(Enemy_Yellow_Circle(self,\
                                        controlPoints,rate,startPos))
        

    
    def init_enemy_blue(self):
        #hyperbola fly pattern
        controlPoints=[(-30,-10),(330,90),(100,450),(500,300),(-100,600)]
        self.enemy_bGroup=pygame.sprite.Group()
        rate=200  #control the time(speed) plane move out of the screen
        startPos=(-70,-70) #start position on the outside screen
        for i in range (5):
            rate+=i*20
            self.enemy_bGroup.add(Enemy_Blue(self,controlPoints,rate,startPos))
        
        
        #pattern S curve
        rate = 500
        startPos=(-70,-70)
        controlPoints=[(-30,-10),(330,90),(100,300),(30,400),(600,500)]
        for i in range (5):
            rate+=i*20
            self.enemy_bGroup.add(Enemy_Blue(self,controlPoints,rate,startPos))
        

    @staticmethod
    def getDistance(x1,y1,x2,y2):
        return ((x1-x2)**2+(y1-y2)**2)**0.5

    
    #key press control
    def key_control(self):
        key_press = pygame.key.get_pressed()
        if (key_press[pygame.K_UP] or key_press[pygame.K_DOWN] or \
            key_press[pygame.K_RIGHT] or key_press[pygame.K_LEFT] or \
            key_press[pygame.K_f]):
            self.hero.move(key_press)

    def hero_skill_control(self):
        self.skill.move()
    
    def map_control(self):
        self.backGround.mapScroll()
    
    def bullets_control(self):
        for bullet in self.hero.bulletsList:
            bullet.moveBullets()
        for bullet_boss_level1 in self.boss_level_1.bulletList:
            bullet_boss_level1.moveBullets()
    
    def enemy_yellow_control(self):
        for e_y_1 in self.enemy_yGroup_1:
            e_y_1.move()
        
    
    def enemy_blue_control(self):
        #make plane start according to the time
        if self.gameTime>20000:
            for e_b in self.enemy_bGroup:
                e_b.move()
            
            for e_y_c in self.enemy_yGroup_circle:
                e_y_c.move()

    def boss_level_1_control(self):
        if self.gameTime>28000:
            self.boss_level_1.move()

    def bomb_level_1_control(self):
        for bomb in self.bombList:
            if bomb.visible:
                bomb.move()

    def updateScore(self):
        self.money+=1
        self.score+=1
        self.score_text=self.font.render(f"Your score: {self.score}",1,self.black)

    def check_Invincible(self):
        #this part designed for test and demo
        if not self.invincible:
            self.heroBloodList.pop(0)

    def check_Collision(self):
        #hero and enemies
        for e_y in self.enemy_yGroup_1:
            result=pygame.sprite.collide_rect_ratio(0.7)(self.hero,e_y)
            if result: 
                # activate bomb animation
                for bomb in self.bombList:
                    if not bomb.visible:
                        bomb.set_pos(e_y.rect.left,e_y.rect.top)
                        bomb.visible=True
                        break
                self.enemy_yGroup_1.remove(e_y)
                self.check_Invincible()
                

        for e_b in self.enemy_bGroup:
            result=pygame.sprite.collide_rect_ratio(0.7)(self.hero,e_b)
            if result: 
                for bomb in self.bombList:
                    if not bomb.visible:
                        bomb.set_pos(e_b.rect.left,e_b.rect.top)
                        bomb.visible=True
                        break
                self.enemy_bGroup.remove(e_b)
                self.check_Invincible()
        
        for e_y_c in self.enemy_yGroup_circle:
            result=pygame.sprite.collide_rect_ratio(0.7)(self.hero,e_y_c)
            if result: 
                for bomb in self.bombList:
                    if not bomb.visible:
                        bomb.set_pos(e_y_c.rect.left,e_y_c.rect.top)
                        bomb.visible=True
                        break
                self.enemy_yGroup_circle.remove(e_y_c)
                self.check_Invincible()
                
        
        #bullets and all enemies
        for bullet in self.hero.bulletsList:
            for e_y in self.enemy_yGroup_1:
                result=pygame.sprite.collide_rect_ratio(0.8)(bullet,e_y)
                if result:
                    for bomb in self.bombList:
                        if not bomb.visible:
                            bomb.set_pos(e_y.rect.left,e_y.rect.top)
                            bomb.visible=True
                            break
                    self.enemy_yGroup_1.remove(e_y)
                    self.hero.bulletsList.remove(bullet)
                    self.updateScore()
            
            for e_b in self.enemy_bGroup:
                result=pygame.sprite.collide_rect_ratio(0.8)(bullet,e_b)
                if result:
                    for bomb in self.bombList:
                        if not bomb.visible:
                            bomb.set_pos(e_b.rect.left,e_b.rect.top)
                            bomb.visible=True
                            break
                    self.enemy_bGroup.remove(e_b)
                    self.hero.bulletsList.remove(bullet)
                    self.updateScore()
            
            for e_y_c in self.enemy_yGroup_circle:
                result=pygame.sprite.collide_rect_ratio(0.8)(bullet,e_y_c)
                if result:
                    for bomb in self.bombList:
                        if not bomb.visible:
                            bomb.set_pos(e_y_c.rect.left,e_y_c.rect.top)
                            bomb.visible=True
                            break
                    self.enemy_yGroup_circle.remove(e_y_c)
                    self.hero.bulletsList.remove(bullet)
                    self.updateScore()
            
            if pygame.sprite.collide_mask(bullet,self.boss_level_1):
                self.hero.bulletsList.remove(bullet)
                self.boss_level_1.beAttacked()
                if self.boss_level_1.bossDie:
                    self.storeGame()
            
        #bullet and player
        for bossBullet in self.boss_level_1.bulletList:
            result=pygame.sprite.collide_rect_ratio(0.8)(bossBullet,self.hero)
            if result:
                self.boss_level_1.bulletList.remove(bossBullet)
                self.check_Invincible()


        
            
        
        #check for game over
        if len(self.heroBloodList)==0:
            self.secondGame.gameOver()

        

        

    def redrawAll(self):
        #draw the map(background)
        self.backGround.draw()
        #draw the heroblood
        for blood in self.heroBloodList:
            blood.draw()
        #draw the herp plane
        self.hero.draw()
        
        #draw the enemy plane
        for e_b in self.enemy_bGroup:
            e_b.draw()
        
        for e_y_1 in self.enemy_yGroup_1:
            e_y_1.draw()
        
        for e_y_c in self.enemy_yGroup_circle:
            e_y_c.draw()

        #draw the boss
        self.boss_level_1.draw()

        #draw the money
        self.myScene.blit(self.score_text,self.score_textpos)
        
        #draw bomb animation
        for bomb in self.bombList:
            if bomb.visible:
                bomb.draw()
        
        #draw skill circle 
        if self.skill.visiable:
            self.skill.draw()
        
        

    def runGame(self):
        #set up a way to quit the game
        pygame.init()
        while self.run:
            pygame.time.delay(10)
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.run=False
        #######game controler 
            #time
            self.gameTime=pygame.time.get_ticks()
            #print(self.gameTime)
            #check for keypressed
            self.key_control()
            #map itself timerfire
            self.map_control()
            # bullets timerfire
            self.bullets_control()
            # enemy plane timerfire
            self.enemy_yellow_control()
            self.enemy_blue_control()
            #boss plane timerfire
            self.boss_level_1_control()
            # bomb animation timerfire
            self.bomb_level_1_control()
            #update the skill circle
            self.hero_skill_control()

        #######game checking
            self.check_Collision() 
            
        #######game viewer
            self.redrawAll()
            
            #refresh the game 
            pygame.display.update()
        #quit the game
        pygame.quit()
    

    #refer code from "https://pythonprogramming.net/pygame-start-menu-tutorial/" 
    #game begin interface
    def introGame(self):
        pygame.init()
        #initial result
        result=False
        run=True
        #cover picture comes from 
        #https://www.beastsofwar.com/board-games/hobbity-new-game-303-squadron-spiel-19/
        cover_image=pygame.image.load("cover.jpg").convert()
        cover_rect=cover_image.get_rect()
        cover_image=pygame.transform.scale(cover_image,\
                                (self.width,self.height))
        #cover_rect=cover_image.get_rect()
        self.myScene.blit(cover_image,(0,0))

        font=pygame.font.SysFont('msyh',56)
        text=font.render("Air Fighter",1,self.lightblue)
        center=(self.width//2,self.height//2-30)
        textpos = text.get_rect(center=center)
        while run:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    run=False
            self.myScene.blit(text, textpos)
            result=self.botton(self.width//2-50,self.height//4*3+10,100,50,self.lightblue,\
                            self.blue,'Start')
            if result:
                self.runGame()
            pygame.display.update()
        pygame.quit()
    
    #"https://pythonprogramming.net/pygame-start-menu-tutorial/"
    def botton(self,left,top,width,height,color1,color2,text):
        
        #draw botton and check for click
        mouse=pygame.mouse.get_pos()
        click=pygame.mouse.get_pressed()
        pygame.draw.rect(self.myScene,color1,(left,top,width,height))
        if left<mouse[0]<left+width and top<mouse[1]<top+height:
            pygame.draw.rect(self.myScene,color2,(left,top,width,height))
            if click[0]==1:
                return True
        #draw text
        font=pygame.font.Font(None,25)
        text=font.render(text,1,self.black)
        center=(left+width/2,top+height/2)
        textpos = text.get_rect(center=center)
        self.myScene.blit(text, textpos)
    
    def storeGame(self):
        pygame.init()
        self.myScene.fill(self.white)
        run=True
        #set up merchant picture
        Mimage=pygame.image.load("buy.png").convert()
        Mrect=Mimage.get_rect()
        Mimage=pygame.transform.scale(Mimage,\
                                (Mrect.width//2+50,Mrect.height//2+50))
        Mrect=Mimage.get_rect()#update the rect data
        self.myScene.blit(Mimage,(self.width//2-Mrect.width//2,0))
        #set up text
        font=pygame.font.SysFont('msyh',25)
        text=font.render(f"Your money : {self.money}",2,self.gold)
        pos=(self.width//2,Mrect.height)
        textpos = text.get_rect(center=pos)
        self.myScene.blit(text, textpos)
        #weapon
        w1_image=pygame.image.load("missil.jpg").convert()
        w1_rect=w1_image.get_rect()
        w1_image=pygame.transform.scale(w1_image,\
                                (w1_rect.width//3,w1_rect.height//3))
        w1_rect=w1_image.get_rect()
        w1_rect.centerx,w1_rect.centery=self.width//4,Mrect.height+100
        self.myScene.blit(w1_image,w1_rect)

        w2_image=pygame.image.load("speedup.jpg").convert()
        w2_rect=w1_image.get_rect()
        w2_image=pygame.transform.scale(w2_image,\
                                (w2_rect.width,w2_rect.height))
        w2_rect=w2_image.get_rect()
        w2_rect.centerx,w2_rect.centery=self.width//4*3,Mrect.height+100
        self.myScene.blit(w2_image,w2_rect)

        #update score in level 2
        



        while run:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    run=False
            result1=self.botton(w1_rect.centerx-60,w1_rect.centery+100,\
                                120,30,self.gold,self.darkGreen,' Missile($2) ')
            result2=self.botton(w2_rect.centerx-60,w2_rect.centery+100,\
                                120,30,self.gold,self.darkGreen,' Speed($5) ')
            if result1 and self.money>=2:
                self.secondGame.hero.missile=True
                self.secondGame.runGame()
            if result2 and self.money>=5:
                self.secondGame.hero.speed=3
                self.secondGame.runGame()
            pygame.display.update()
        pygame.quit()


    


if __name__=='__main__':
    mainGame=MainWindow()
    #mainGame.secondGame.runGame()
    mainGame.introGame()
    
    

```
