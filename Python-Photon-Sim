import pygame
import sys
import random
import math
import time

# ---------- Config ----------
WIDTH, HEIGHT = 800, 600
FPS = 60
PHOTON_SPEED = 1000  # pixels/sec
PHOTON_DECAY = 5.0   # seconds
PORTAL_SIZE = (300, 200)
PORTAL_BORDER = 4
JOYSTICK_RADIUS = 50
PHOTON_SUBSAMPLE = 2  # take every Nth pixel for performance
# ----------------------------

pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Ultimate Pixel-Accurate Photon Portal")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 18)

# Scene
boxes = []
player_pos = [WIDTH//2, HEIGHT//2]
photons = []
history_frames = []

# Portal
portal_rect = pygame.Rect(WIDTH//2 - PORTAL_SIZE[0]//2, 50, PORTAL_SIZE[0], PORTAL_SIZE[1])
portal_slider_rect = pygame.Rect(50, HEIGHT-60, WIDTH-100, 20)
portal_slider_value = 0.0

# Joystick
joystick_center = (100, HEIGHT-150)
joystick_pos = joystick_center
joystick_active = False
move_vector = [0,0]

# Photon visibility toggle
show_photons = True
toggle_rect = pygame.Rect(10, 10, 120, 30)

# Colors
BG_COLOR = (15, 15, 25)
BOX_COLOR = (80, 180, 220)
PLAYER_COLOR = (220, 180, 50)
PHOTON_COLOR = (255, 255, 120)
JOYSTICK_BG = (50,50,50)
JOYSTICK_FG = (200,200,200)
SLIDER_COLOR = (200,200,200)
TOGGLE_BG = (60,60,60)
TOGGLE_TEXT = (255,255,255)

# ---------- Helpers ----------
def spawn_box(x, y):
    boxes.append(pygame.Rect(x-20, y-20, 40, 40))

def remove_box(x, y):
    for b in boxes:
        if b.collidepoint(x, y):
            boxes.remove(b)
            break

def emit_photons():
    global photons
    objects = [{"type":"player","shape":"circle","pos":tuple(player_pos),"radius":10}] + \
              [{"type":"box","shape":"rect","rect":b} for b in boxes]

    for obj in objects:
        if obj["shape"]=="rect":
            rect = obj["rect"]
            for x in range(rect.left, rect.right, PHOTON_SUBSAMPLE):
                for y in range(rect.top, rect.bottom, PHOTON_SUBSAMPLE):
                    angle = random.uniform(0,2*math.pi)
                    vel = [PHOTON_SPEED*math.cos(angle), PHOTON_SPEED*math.sin(angle)]
                    photons.append({
                        "pos":[x,y],
                        "vel":vel,
                        "color":BOX_COLOR,
                        "type":"box",
                        "hit_history":[],
                        "time_alive":0
                    })
        elif obj["shape"]=="circle":
            cx,cy = obj["pos"]
            r = obj["radius"]
            for dx in range(-r,r+1,PHOTON_SUBSAMPLE):
                for dy in range(-r,r+1,PHOTON_SUBSAMPLE):
                    if dx*dx + dy*dy <= r*r:
                        angle = random.uniform(0,2*math.pi)
                        vel = [PHOTON_SPEED*math.cos(angle), PHOTON_SPEED*math.sin(angle)]
                        photons.append({
                            "pos":[cx+dx, cy+dy],
                            "vel":vel,
                            "color":PLAYER_COLOR,
                            "type":"player",
                            "hit_history":[],
                            "time_alive":0
                        })

def update_photons(dt):
    global photons
    new_photons=[]
    for p in photons:
        p["pos"][0] += p["vel"][0]*dt
        p["pos"][1] += p["vel"][1]*dt
        p["time_alive"] += dt

        # collision recording
        hits=[]
        # player
        dx = player_pos[0]-p["pos"][0]
        dy = player_pos[1]-p["pos"][1]
        if dx*dx + dy*dy <= 10*10:
            hits.append({"type":"player","pos":tuple(player_pos)})
        for b in boxes:
            if b.collidepoint(p["pos"]):
                hits.append({"type":"box","rect":b.copy()})
        if hits:
            p["hit_history"].extend(hits)

        if 0<=p["pos"][0]<=WIDTH and 0<=p["pos"][1]<=HEIGHT and p["time_alive"]<=PHOTON_DECAY:
            new_photons.append(p)
    photons=new_photons

def render_scene():
    scene = pygame.Surface((WIDTH, HEIGHT))
    scene.fill(BG_COLOR)
    for b in boxes:
        pygame.draw.rect(scene, BOX_COLOR, b)
    pygame.draw.circle(scene, PLAYER_COLOR, (int(player_pos[0]),int(player_pos[1])), 10)
    if show_photons:
        for p in photons:
            pygame.draw.circle(scene, PHOTON_COLOR, (int(p["pos"][0]),int(p["pos"][1])), 2)
    return scene

def capture_history(scene):
    history_frames.append({"scene":scene.copy(),"photons":[dict(p) for p in photons]})
    if len(history_frames) > int(FPS*PHOTON_DECAY*2):
        history_frames.pop(0)

def draw_portal():
    pygame.draw.rect(screen,(200,200,200),portal_rect.inflate(PORTAL_BORDER*2,PORTAL_BORDER*2),PORTAL_BORDER)
    if not history_frames:
        return
    idx = int((len(history_frames)-1)*(1-portal_slider_value))
    idx = max(0, min(len(history_frames)-1, idx))
    frame = history_frames[idx]
    portal_surf = pygame.Surface(PORTAL_SIZE)
    portal_surf.fill(BG_COLOR)
    for p in frame["photons"]:
        for obj in p["hit_history"]:
            if obj["type"]=="player":
                pygame.draw.circle(portal_surf, PLAYER_COLOR, (int(obj["pos"][0]-portal_rect.left), int(obj["pos"][1]-portal_rect.top)),10)
            elif obj["type"]=="box":
                rect = obj["rect"].copy()
                rect.topleft = (rect.left-portal_rect.left, rect.top-portal_rect.top)
                pygame.draw.rect(portal_surf, BOX_COLOR, rect)
    screen.blit(portal_surf, portal_rect.topleft)

def draw_toggle():
    pygame.draw.rect(screen, TOGGLE_BG, toggle_rect)
    txt = "Photons: ON" if show_photons else "Photons: OFF"
    txt_surf = font.render(txt, True, TOGGLE_TEXT)
    screen.blit(txt_surf, (toggle_rect.left+5, toggle_rect.top+5))

# ---------- Main Loop ----------
running=True
last_tap_time=0
tap_threshold=0.25
while running:
    dt = clock.tick(FPS)/1000.0
    for event in pygame.event.get():
        if event.type==pygame.QUIT:
            running=False
        elif event.type==pygame.MOUSEBUTTONDOWN:
            mx,my = event.pos
            if math.hypot(mx-joystick_center[0],my-joystick_center[1])<=JOYSTICK_RADIUS:
                joystick_active=True
            elif portal_slider_rect.collidepoint(mx,my):
                portal_slider_value = (mx-portal_slider_rect.left)/portal_slider_rect.width
            elif toggle_rect.collidepoint(mx,my):
                show_photons = not show_photons
            else:
                now=time.time()
                if now-last_tap_time < tap_threshold:
                    remove_box(mx,my)
                else:
                    spawn_box(mx,my)
                last_tap_time=now
        elif event.type==pygame.MOUSEBUTTONUP:
            joystick_active=False
            joystick_pos = joystick_center
            move_vector = [0,0]
        elif event.type==pygame.MOUSEMOTION:
            mx,my = event.pos
            if joystick_active:
                dx = mx - joystick_center[0]
                dy = my - joystick_center[1]
                dist = math.hypot(dx,dy)
                if dist>JOYSTICK_RADIUS:
                    dx,dy = dx/dist*JOYSTICK_RADIUS, dy/dist*JOYSTICK_RADIUS
                joystick_pos = (joystick_center[0]+dx, joystick_center[1]+dy)
                move_vector = [dx/JOYSTICK_RADIUS*200, dy/JOYSTICK_RADIUS*200]
            if pygame.mouse.get_pressed()[0]:
                if portal_slider_rect.collidepoint(mx,my):
                    portal_slider_value = (mx-portal_slider_rect.left)/portal_slider_rect.width

    # Move player
    player_pos[0] += move_vector[0]*dt
    player_pos[1] += move_vector[1]*dt
    player_pos[0] = max(0, min(WIDTH, player_pos[0]))
    player_pos[1] = max(0, min(HEIGHT, player_pos[1]))

    emit_photons()
    update_photons(dt)
    scene = render_scene()
    capture_history(scene)
    screen.blit(scene,(0,0))
    draw_portal()
    draw_toggle()

    # Draw joystick
    pygame.draw.circle(screen, JOYSTICK_BG, joystick_center, JOYSTICK_RADIUS)
    pygame.draw.circle(screen, JOYSTICK_FG, joystick_pos, 20)

    # Draw slider
    pygame.draw.rect(screen,(80,80,80),portal_slider_rect)
    handle_x = portal_slider_rect.left + portal_slider_value*portal_slider_rect.width
    pygame.draw.rect(screen, SLIDER_COLOR, (handle_x-8, portal_slider_rect.top-4, 16, portal_slider_rect.height+8))

    # Info text
    info = "Joystick: move | Tap: spawn | Double tap: remove | Slider: rewind | Toggle: photons"
    txt_surf = font.render(info,True,(255,255,255))
    screen.blit(txt_surf,(6,HEIGHT-26))

    pygame.display.flip()

pygame.quit()
sys.exit()
