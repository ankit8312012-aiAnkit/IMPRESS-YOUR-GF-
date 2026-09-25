import pygame
import math
import random

pygame.init()

# =========================================================
# WINDOW
# =========================================================

WIDTH = 800
HEIGHT = 600

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("3D Hex Heart")

clock = pygame.time.Clock()

# =========================================================
# COLORS
# =========================================================

BLACK = (2, 2, 8)

HEART_COLORS = [
    (255, 20, 80),
    (255, 40, 100),
    (220, 10, 70),
    (255, 80, 130),
    (180, 0, 60),
]

PARTICLE_COLORS = [
    (255, 40, 100),
    (255, 100, 160),
    (180, 30, 255),
    (100, 50, 255),
    (255, 255, 255),
]

# =========================================================
# OUTER 3D PARTICLE SPHERE
# =========================================================

particles = []

PARTICLE_COUNT = 900

for i in range(PARTICLE_COUNT):

    phi = random.uniform(0, math.pi * 2)
    theta = math.acos(random.uniform(-1, 1))

    radius = random.uniform(260, 290)

    x = radius * math.sin(theta) * math.cos(phi)
    y = radius * math.sin(theta) * math.sin(phi)
    z = radius * math.cos(theta)

    particles.append({
        "x": x,
        "y": y,
        "z": z,
        "size": random.uniform(1, 2.8),
        "color": random.choice(PARTICLE_COLORS)
    })

# =========================================================
# HEXAGONAL HEART
# =========================================================
#
# We create a 3D heart from many small triangular faces.
# The faces create the faceted / hexagonal appearance.
# =========================================================

heart_vertices = []
heart_faces = []

GRID = 13

# Create heart surface points
for iy in range(-GRID, GRID + 1):

    y = iy / GRID

    for ix in range(-GRID, GRID + 1):

        x = ix / GRID

        # Heart equation
        hx = x * 1.35
        hy = y * 1.35

        value = (
            (hx * hx + hy * hy - 1) ** 3
            - hx * hx * hy ** 3
        )

        if value <= 0:

            # Front/back thickness
            depth = math.sqrt(
                max(0, 1 - abs(x) * 0.35)
            )

            z = depth * 0.55

            # Rounded 3D surface
            front_z = z * (
                0.8 +
                0.2 * math.cos(y * math.pi / 2)
            )

            # Scale
            px = x * 150
            py = -y * 150
            pz = front_z * 70

            heart_vertices.append(
                (px, py, pz)
            )

# =========================================================
# MAKE FACET TRIANGLES
# =========================================================

# Instead of a perfectly smooth heart,
# triangles overlap to produce a low-poly look.

for i in range(len(heart_vertices) - 2):

    a = i
    b = (i + 1) % len(heart_vertices)
    c = (i + 2) % len(heart_vertices)

    heart_faces.append(
        (a, b, c)
    )

# =========================================================
# EXTRA HEXAGONAL FACETS
# =========================================================

hex_faces = []

for i in range(0, len(heart_vertices) - 6, 6):

    ids = [
        i,
        i + 1,
        i + 2,
        i + 3,
        i + 4,
        i + 5
    ]

    if max(ids) < len(heart_vertices):
        hex_faces.append(ids)

# =========================================================
# 3D PROJECTION
# =========================================================

def project(x, y, z, camera):

    perspective = camera / (camera + z)

    sx = WIDTH / 2 + x * perspective
    sy = HEIGHT / 2 + y * perspective

    return int(sx), int(sy), perspective


# =========================================================
# ROTATE POINT
# =========================================================

def rotate_point(x, y, z, ax, ay):

    # Y rotation
    cos_y = math.cos(ay)
    sin_y = math.sin(ay)

    rx = x * cos_y - z * sin_y
    rz = x * sin_y + z * cos_y

    # X rotation
    cos_x = math.cos(ax)
    sin_x = math.sin(ax)

    ry = y * cos_x - rz * sin_x
    rz2 = y * sin_x + rz * cos_x

    return rx, ry, rz2


# =========================================================
# HEART OUTLINE SHAPE
# =========================================================

def heart_2d(scale):

    points = []

    for i in range(100):

        t = math.pi * 2 * i / 100

        x = 16 * math.sin(t) ** 3

        y = (
            13 * math.cos(t)
            - 5 * math.cos(2 * t)
            - 2 * math.cos(3 * t)
            - math.cos(4 * t)
        )

        points.append(
            (
                WIDTH / 2 + x * scale,
                HEIGHT / 2 - y * scale
            )
        )

    return points


# =========================================================
# ANIMATION
# =========================================================

rotation = 0

zoom_time = 0

running = True

while running:

    # -----------------------------------------------------
    # EVENTS
    # -----------------------------------------------------

    for event in pygame.event.get():

        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:

            if event.key == pygame.K_ESCAPE:
                running = False

    # -----------------------------------------------------
    # CLEAR
    # -----------------------------------------------------

    screen.fill(BLACK)

    # -----------------------------------------------------
    # ROTATION
    # -----------------------------------------------------

    rotation += 0.008

    # -----------------------------------------------------
    # ZOOM
    # -----------------------------------------------------

    zoom_time += 0.012

    # smooth 0 -> 1 -> 0
    zoom = (math.sin(zoom_time) + 1) / 2

    # smoother cinematic curve
    zoom = zoom * zoom * (3 - 2 * zoom)

    camera = 700 - zoom * 430

    # Heart gets larger when camera approaches
    heart_scale = 0.72 + zoom * 0.55

    # -----------------------------------------------------
    # PARTICLES
    # -----------------------------------------------------

    objects = []

    for p in particles:

        x = p["x"]
        y = p["y"]
        z = p["z"]

        rx, ry, rz = rotate_point(
            x,
            y,
            z,
            rotation * 0.45,
            rotation
        )

        sx, sy, perspective = project(
            rx,
            ry,
            rz,
            camera
        )

        size = max(
            1,
            int(p["size"] * perspective)
        )

        objects.append(
            (
                rz,
                "particle",
                sx,
                sy,
                size,
                p["color"]
            )
        )

    # -----------------------------------------------------
    # HEART FACETS
    # -----------------------------------------------------

    transformed = []

    for v in heart_vertices:

        x, y, z = v

        x *= heart_scale
        y *= heart_scale
        z *= heart_scale

        rx, ry, rz = rotate_point(
            x,
            y,
            z,
            rotation * 0.25,
            rotation * 0.7
        )

        transformed.append(
            (rx, ry, rz)
        )

    # -----------------------------------------------------
    # TRIANGULAR FACETS
    # -----------------------------------------------------

    for face in heart_faces:

        if max(face) >= len(transformed):
            continue

        p1 = transformed[face[0]]
        p2 = transformed[face[1]]
        p3 = transformed[face[2]]

        a = project(*p1, camera)
        b = project(*p2, camera)
        c = project(*p3, camera)

        depth = (
            p1[2] +
            p2[2] +
            p3[2]
        ) / 3

        # Different reds make the surface faceted
        color = random.choice(HEART_COLORS)

        objects.append(
            (
                depth,
                "face",
                [
                    (a[0], a[1]),
                    (b[0], b[1]),
                    (c[0], c[1])
                ],
                color
            )
        )

    # -----------------------------------------------------
    # DRAW FROM BACK TO FRONT
    # -----------------------------------------------------

    objects.sort(
        key=lambda item: item[0]
    )

    for obj in objects:

        if obj[1] == "particle":

            _, _, x, y, size, color = obj

            if (
                -10 < x < WIDTH + 10
                and -10 < y < HEIGHT + 10
            ):

                pygame.draw.circle(
                    screen,
                    color,
                    (x, y),
                    size
                )

        else:

            _, _, points, color = obj

            if len(points) == 3:

                pygame.draw.polygon(
                    screen,
                    color,
                    points
                )

                # Dark facet border
                pygame.draw.line(
                    screen,
                    (80, 5, 35),
                    points[0],
                    points[1],
                    1
                )

                pygame.draw.line(
                    screen,
                    (80, 5, 35),
                    points[1],
                    points[2],
                    1
                )

                pygame.draw.line(
                    screen,
                    (80, 5, 35),
                    points[2],
                    points[0],
                    1
                )

    # =====================================================
    # BRIGHT HEART HIGHLIGHT
    # =====================================================

    outline = heart_2d(
        0.75 * heart_scale
    )

    # Only subtle outline
    pygame.draw.lines(
        screen,
        (255, 100, 160),
        True,
        outline,
        2
    )

    # =====================================================
    # DISPLAY
    # =====================================================

    pygame.display.flip()

    clock.tick(60)

pygame.quit()