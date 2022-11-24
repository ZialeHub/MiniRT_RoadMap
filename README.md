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

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/IntersectSphere.png)
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
&emsp;L'intersection du plan est probablement la plus simple. Nous allons utiliser le fait que le dot product de deux vecteurs perpendiculaires est égal à 0. ( Un point sur le plan a donc un vecteur du point du plan vers ses coordonnées qui correspond à un vecteur perpendiculaire à la normal du plan. )
```
double  denom;
double  t;

denom = ft_dot(plane->normal, ray->dir);
if (denom == 0)
  return (NULL);
t = ft_dot((plane->point - ray->origin), plane->normal) / denom;
```

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/IntersectPlane.png)
- Calculer la normal
&emsp;La normal d'une intersection sur un plan correspond à la normal du plan. Aucun calcul necessaire :D

### INTERSECTION CYLINDER
- Résoudre l'équation du second degré (t1, t2)
- Calculer les caps de bout de cylindre (t3, t4)
- Calculer l'intersection
- Calculer la normal

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/DiffuseCylinder.png)

Pour tout les calculs de lumiere, il est recommandé de mettre toutes les couleurs ( RGB ) sur des ratios entre 0 et 1. Cela simplifie beaucoup les calculs et évite de saturer les couleurs.

### LUMIERE AMBIENTE
- A quoi correspond la lumiere ambiente
&emsp;La lumiere ambiente est une lumiere hypothétique, elle est présente partout et à tout moment du rendu. Un object ne peut donc jamais être totalement dans le noir.
Pour calculer la lumiere ambiente dans miniRT, on utilise les propriétés données dans la scene.
```
t_color *color;

color = ft_clamp(ambient->color) * ambient->ratio;
```

### LUMIERE DIFFUSE
- A quoi correspond la lumiere diffuse
&emsp;La lumiere diffuse est la lumiere d'un spot sur un objet, elle permet d'amplifier le rendu 3d d'un objet grace à de legeres ombres en fonction de l'angle des rayons de lumiere. On va donc calculer l'angle créer entre le rayon allant de l'intersection vers la lumiere et la normal de l'intersection.
```
double  diffuse_ratio;
t_color *color;
t_vec3 *light_dir;

light_dir = light->point - intersection.point;
diffuse_ratio = ft_dot(intersection.dir, light_dir);
diffuse_ratio = max(diffuse_ratio, 0.0);
diffuse_ratio = diffuse_ratio * light->ratio;
color = ft_new_color(1, 1, 1) * diffuse_ratio;
```

&emsp;La couleur finale correspond à la somme de la lumiere ambient et de la lumiere diffuse, multiplié par la couleur de l'objet.
Evidemment, si vous avez ramené les couleurs sur un ratio entre 0 et 1, alors il vous faut multiplier le résultat par 255.0 .
### OMBRES (HARD)
&emsp;Dans le ray tracing il y a plusieurs types d'ombres, les ombres "hard" qui sont les ombres créées par la presence d'un objet entre la lumiere et le point d'intersection actuel. Et les ombres "soft", qui sont les ombres plus legeres pour rendre plus réalistes les ombres "hard" ( la penombre autour des ombres ).
Nous allons ici nous limiter aux ombres "hard" car nos point de lumieres sont par defaut limiter à un point fix.

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/softhardShadow.jpg)
- Calcul des ombres
```
t_ray *shadow_ray;

shadow_ray->origin = intersection->point;
shadow_ray->dir = light->point - intersection->point;
```
Il vous faut donc trouver l'intersection avec se ray, si il y a intersection alors l'objet est dans l'ombre et on va donc supprimer la lumiere diffuse du calcul de la couleur finale.

Pour aller plus loin, je laisse les personnes interessées faire leur recherches personnels :D

### REDIMENSIONNER / PIVOTER / TRANSLATER
- Redimensionner (sphere: diametre / cylindre: Diametre, longueur)
- Pivoter (camera, plan, cylindre)
- Translater (camera, light, sphere, plan, cylindre)

&emsp;La lumiere Pour les modifications des données des objets, nous avons utiliser des hooks de la mlx par simplicité, d'autres méthodes peuvent être utilisées comme une interface graphique ou un menu interactif fait avec la minilibx.
Par defaut, nous avons utilisé des valeurs de ± 1 pour les redimensionnement et les translations, mais des rotations de ±0.1 sur chaque axes.
Bien sur les valeurs peuvent être changées comme bon vous semble tant que le rendu reste correct et utilisable par l'utilisateur.


# BONUS
### LUMIERE SPECULAIRE => PHONG REFLECTION MODEL
- A quoi correspond la lumiere speculaire

&emsp;La lumiere speculaire est la reflexion de la lumiere sur un objet, ce qui produit un effet "brillant" sans modifier totalement les spécificités des materiaux.

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/ReflectSpecular.png)
- Calcul de la lumiere speculaire

&emsp; Pour calculer la lumiere speculaire nous allons devoir calculer l'angle de reflexion de la lumiere pour savoir si la partie reflechie est transmise vers la camera.

```
t_vec3  reflect;

reflect = ray.dir - (intersection.normal * (2.0 * ft_dot(light.dir, intersection.normal)));
```

Une fois le vecteur de reflexion calculé et normalizé, le ratio de cette lumiere correspond au dot entre le vecteur de reflexion et la direction du ray de la camera inversé, mis à la puissance de la force de reflexion choisi. ([1 - 1000] plus la force est haute et plus la reflexion sera précise et moins étalée.)
On peut ensuite calculer la couleur donnée par cette partie de la lumiere:

```
t_color color;

color = light->rgb * light->ratio * reflect_ratio * Ratio_Reflexion_matiere;
```

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
