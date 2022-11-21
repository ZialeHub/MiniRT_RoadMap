# MiniRT_RoadMap(WIP)
*Read this in other languages: [English](README.en.md)**(WIP)**, [Français](README.md)**(WIP)**.*
### GENERER LES RAYS
- Génerer le ray en fonction de la camera

&emsp;Premierement, le ray tracing, comme son nom l'indique consiste à tracer des rays d'une camera dans une direction données. Dans miniRT, votre camera est désignée par un point ( x, y, z ), un vecteur normalizé ( Vx, Vy, Vz ) et un FOV ( 0-180 ). Nous devons donc tracer des rays partant de son point, dans la direction du vecteur, tout en limitant le FOV.

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/Camera.jpg)
&emsp;Le FOV correspond à l'angle horizontal de la vision de la camera, mais nous devons prendre en compte la taille de votre fenêtre pour connaitre le nombre de pixel.
```
t_ray *ray;
double  tmp;

ray = ft_init_ray();
ray->tmax = INFINITY;
ray->tmin = 0.0;
ray->point = camera->point;
ray->dir.x = pixel_x + 0.5 - WIN_WIDTH * 0.5;
ray->dir.y = pixel_y + 0.5 - WIN_HEIGHT * 0.5;
tmp = 2.0 * tan((camera->fov * M_PI * 0.5) / 180.0);
if (WIN_WIDTH < WIN_HEIGHT)
  ray->dir.z = WIN_HEIGHT / tmp;
else
  ray->dir.z = WIN_HEIGHT / tmp;
ft_normalize(ray->dir);
```
- Pivoter le ray en fonction du pixel visé
```
double  phi;
double  theta;

phi = atan(camera->dir.y, camera->dir.x);
theta = acos(camera->dir.z);
camera->dir.x = cos(theta) * camera->dir.x + sin(theta) * camera->dir.z;
camera->dir.z = cos(theta) * camera->dir.z - sin(theta) * camera->dir.x;
camera->dir.x = cos(theta) * camera->dir.x - sin(theta) * camera->dir.y;
camera->dir.y = sin(theta) * camera->dir.x + cos(theta) * camera->dir.y;
```

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/RotateCamera.svg)

### INTERSECTION SPHERE
- Resoudre l'équation du second degré
&emsp;L'intersection avec la sphere est la plus classique pour un petit ray tracer, il faut résoudre l'équation du second degré.
```
a = ft_dot(ray->dir, ray->dir);
b = 2.0 * (ft_dot(ray->dir, (ray->origin - sphere->center));
c = ft_dot((ray->origin - sphere->center), (ray->origin - sphere->center)) - (sphere->radius * sphere->radius);

delta = b * b - 4.0 * a * c;
```
Si le delta est inferieur à 0 alors il n'y a pas d'intersection, sinon la distance (aussi appelé temps "t") est donné par le minimum positif entre:
```
t1 = (-b - sqrt(delta)) / (2.0 * a);
t2 = (-b + sqrt(delta)) / (2.0 * a);
```
Dans le cas où l'on veut récupérer l'object le plus proche touché par le rayon on peut stocker un "t" dans le ray. 
Si le min(t1, t2) est inférieur à 0 alors l'intersection est derriere la camera.
Sinon on compare min(t1, t2) avec ray->t pour le mettre à jour si le résultat est inferieur.
Le point d'intersection est donc défini comme suit:
```
intersection->origin = ray->origin + (t * ray->dir);
```

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/IntersectSphere.jpg)
- Calculer la normal
&emsp;Le calcul de la normal d'une sphere correspond au vecteur normalizé allant du centre de la sphere vers le point d'intersection.
```
intersection.dir = intersection.origin - sphere->center;
```
Pour une bonne gestion des intersections dans le cas où la camera est à l'interieur de l'objet, il faut vérifier que l'angle entre la normal au point d'intersection et la direction du rayon soit strictement superieur à 0.
Si c'est le cas alors, la camera est à l'interieur de l'objet et il faut donc inverser la normal.
```
if (ft_dot(ray->dir, intersection->dir) > 0.0)
  intersection.dir = -intersection.dir;
```
### INTERSECTION PLAN
- Vérifier l'intersection
- Calculer la normal
### INTERSECTION CYLINDER
- Résoudre l'équation du second degré (t1, t2)
- Calculer les caps de bout de cylindre (t3, t4)
- Calculer l'intersection
- Calculer la normal

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/DiffuseCylinder.png)
### LUMIERE AMBIENTE
- A quoi correspond la lumiere ambiente
- Simple calcul
### LUMIERE DIFFUSE
- A quoi correspond la lumiere diffuse
- Calcule de la lumiere diffuse
### OMBRES (HARD)
- Creation des rayons de lumière
- Calcul des ombres
### REDIMENSIONNER / PIVOTER / TRANSLATER
- Redimensionner (sphere: diametre / cylindre: Diametre, longueur)
- Pivoter (camera, plan, cylindre)
- Translater (camera, light, sphere, plan, cylindre)


# BONUS
### LUMIERE SPECULAIRE => PHONG REFLECTION MODEL
- A quoi correspond la lumiere speculaire
- Calcul de la lumiere speculaire

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/EverythingPhongLight.png)

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/PhongShadow.png)
### CHECKERBOARD
- Checkerboard sphere
- Checkerboard plan
- Checkerboard cylindre
- Checkerboard cone

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/Checkerboard.png)
### MULTIPLE-SPOT DE LUMIERE + LUMIERE COLORE
- Calculs d'addition des lumieres
- Calculs des mélanges de couleurs

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/MultipleSpot.png)
### INTERSECTION CONE
- Résoudre l'équation du second degré (t1, t2)
- Calculs des caps de haut et bas (t3, t4)
- Calcul de l'intersection
- Calcul de la normal

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/DiffuseConePlane.png)
### BUMB MAP
- A quoi correspond une normal map / Bump map
- Initialisation d'une bump map
- Exemple de calcul

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/BumpMap.png)
