# MiniRT_RoadMap(WIP)
*Read this in other languages: [English](README.en.md)**(WIP)**, [Français](README.md)**(WIP)**.*
### GENERER LES RAYS
- Générer le ray en fonction de la camera

&emsp;Premièrement, le ray tracing, comme son nom l'indique consiste à tracer des rays d'une camera dans une direction donnée. Dans miniRT, votre camera est désignée par un point ( x, y, z ), un vecteur normalisé ( Vx, Vy, Vz ) et un FOV ( 0-180 ). Nous devons donc tracer des rays partant de son point, dans la direction du vecteur, tout en limitant le FOV.

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/Camera.jpg)
&emsp;Le FOV correspond à l'angle horizontal de la vision de la camera, mais nous devons prendre en compte la taille de votre fenêtre pour connaître le nombre de pixels.
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

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/RotateCamera.svg)

### INTERSECTION SPHERE
- Résoudre l'équation du second degré
&emsp;L'intersection avec la sphère est la plus classique pour un petit ray tracé, il faut résoudre l'équation du second degré.
```
a = ft_dot(ray->dir, ray->dir);
b = 2.0 * (ft_dot(ray->dir, (ray->origin - sphere->center));
c = ft_dot((ray->origin - sphere->center), (ray->origin - sphere->center)) - (sphere->radius * sphere->radius);

delta = b * b - 4.0 * a * c;
```
Si le delta est inférieur à 0 alors il n'y a pas d'intersection, sinon la distance (aussi appelé temps "t") est donné par le minimum positif entre :
```
t1 = (-b - sqrt(delta)) / (2.0 * a);
t2 = (-b + sqrt(delta)) / (2.0 * a);
```
Dans le cas où l'on voudrait récupérer l'objet le plus proche touché par le rayon, on peut stocker un "t" dans le ray. 
Si le min(t1, t2) est inférieur à 0 alors l'intersection est derrière la camera.
Sinon on compare min(t1, t2) avec ray->t pour le mettre à jour si le résultat est inférieur.
Le point d'intersection est donc défini comme suit :
```
intersection->origin = ray->origin + (t * ray->dir);
```

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/IntersectSphere.png)
- Calculer la normal
&emsp;Le calcul de la normal d'une sphère correspond au vecteur normalisé allant du centre de la sphère vers le point d'intersection.
```
intersection.dir = intersection.origin - sphere->center;
```
Pour une bonne gestion des intersections dans le cas où la camera est à l'intérieur de l'objet, il faut vérifier que l'angle entre la normal au point d'intersection et la direction du rayon soit strictement supérieur à 0.
Si c'est le cas alors, la caméra est à l'intérieur de l'objet et il faut donc inverser la normal.
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

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/IntersectPlane.png)
- Calculer la normal
&emsp;La normal d'une intersection sur un plan correspond à la normal du plan. Aucun calcul nécessaire :D

### INTERSECTION CYLINDRE
![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/IntersectCylinder.png)

Pour les calculs du cylindre, il va falloir calculer un cylindre infini (un tube infini), puis "créer" deux plans aux limites du cylindre voulu pour permettre de calculer les intersections avec les caps.

- Résoudre l'équation du second degré (t1, t2)
Pour obtenir t1 et t2, il faut calculer l'intersection du rayon avec le corps infini sur le cylindre.
Nous devons résoudre l'équation du second degré correspondant à l'intersection avec le corps du cylindre :
```
tmp1 = ft_cross(ray->dir, cylindre->normal);
a = ft_dot(tmp1, tmp1);
tmp_vect = ray->origin - cylindre->origin;
tmp2 = ft_cross(tmp_vect, cylinder->normal);
b = 2.0 * ft_dot(tmp1, tmp2);
c = ft_dot(tmp2, tmp2) - (cylinder->diameter / 2.0) ^ 2.0;
d = b ^ 2.0 - 4.0 * a * c;
t1 = (-b - sqrt(d)) / (2.0 * a);
t2 = (-b + sqrt(d)) / (2.0 * a);
if (t1 > t2)
  ft_swap(t1, t2);
```

- Calculer les caps de bout de cylindre (t3, t4)
Les intersections t3 et t4 sont les intersections avec les plans pour limiter la taille du cylindre. On va centrer le point du cylindre pour que les plans soient à équidistance.
```
mid_vect = cylinder->normal * (cylinder->height / 2.0);
mid_point = cylinder->origin + mid_vect;
tmp1 = ft_dot(mid_point - ray->origin, cylinder->normal);
tmp2 = ft_dot(ray->dir, cylinder->normal);
t3 = (tmp1 + (cylinder->height / 2.0)) / tmp2;
t4 = (tmp1 - (cylinder->height / 2.0)) / tmp2;
if (t3 > t4)
  ft_swap(t3, t4);
```

- Calculer l'intersection
Une fois les quatre t calculaient, il nous faut maintenant décider lequel est le plus proche de notre origine de rayon.
Vous pouvez vous aider du schéma ci-dessus pour comprendre comment choisir le t.
```
if (t3 > t2 || t4 < t1)
  return ; // Pas d'intersection
t_final = fmax(t1, t3);
if (t_final < 0)
  t_final = fmin(t2, t4);
if (t_final <= 0 || t_final > ray->tmax)
  return ; // Pas d'intersection
```

- Calculer la normal
```
if (t3 < t1)
  ft_normal_body() // La normal est celle du corps du cylindre
else
  ft_normal_caps() // La normal est celle du cap touché

// ft_normal_caps()
if (ft_dot(ray->dir, cylinder->normal) > 0.0)
  intersection.normal = -cylinder->normal;
else
  intersection.normal = cylinder->normal;

// ft_normal_body()
// Il faut calculer la projection de l'intersection avec l'axe du cylindre et ensuite
// créer le vecteur entre cette projection et le point d'intersection.
```



![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/DiffuseCylinder.png)

Pour tous les calculs de lumière, il est recommandé de mettre toutes les couleurs ( RGB ) sur des ratios entre 0 et 1. Cela simplifie beaucoup les calculs et évite de saturer les couleurs.

### LUMIERE AMBIENTE
- A quoi correspond la lumière ambiante
&emsp;La lumière ambiante est une lumière hypothétique, elle est présente partout et à tout moment du rendu. Un objet ne peut donc jamais être totalement dans le noir.
Pour calculer la lumière ambiante dans miniRT, on utilise les propriétés données dans la scène.
```
t_color *color;

color = ft_clamp(ambient->color) * ambient->ratio;
```

### LUMIERE DIFFUSE
- A quoi correspond la lumière diffuse
&emsp;La lumière diffuse est la lumière d'un spot sur un objet, elle permet d'amplifier le rendu 3d d'un objet grâce à de légères ombres en fonction de l'angle des rayons de lumière. On va donc calculer l'angle créé entre le rayon allant de l'intersection vers la lumière et la normal de l'intersection.
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

&emsp;La couleur finale correspond à la somme de la lumière ambiante et de la lumière diffuse, multiplié par la couleur de l'objet.
Evidemment, si vous avez ramené les couleurs sur un ratio entre 0 et 1, alors il vous faut multiplier le résultat par 255.0 .
### OMBRES (HARD)
&emsp;Dans le ray tracing il y a plusieurs types d'ombres, les ombres "hard" qui sont les ombres créées par la présence d'un objet entre la lumière et le point d'intersection actuel. Et les ombres "soft", qui sont les ombres plus légères pour rendre plus réalistes les ombres "hard" ( la pénombre autour des ombres ).
Nous allons ici nous limiter aux ombres "hard", car nos points de lumières sont par défaut limités à un point fixe.

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/softhardShadow.jpg)
- Calcul des ombres
```
t_ray *shadow_ray;

shadow_ray->origin = intersection->point;
shadow_ray->dir = light->point - intersection->point;
```
Il vous faut donc trouver l'intersection avec ce ray, s'il y a une intersection alors l'objet est dans l'ombre et on va donc supprimer la lumière diffuse du calcul de la couleur finale.

Pour aller plus loin, je laisse les personnes intéressées faire leurs recherches personnels :D

### REDIMENSIONNER / PIVOTER / TRANSLATER
- Redimensionner (sphère : diamètre / cylindre : Diamètre, longueur)
- Pivoter (caméra, plan, cylindre)
- Translater (caméra, light, sphère, plan, cylindre)

&emsp;Pour les modifications des données des objets, nous avons utilisé des hooks de la mlx par simplicité, d'autres méthodes peuvent être utilisées comme une interface graphique ou un menu interactif fait avec la minilibx.
Par défaut, nous avons utilisé des valeurs de ± 1 pour les redimensionnements et les translations, mais des rotations de ±0.1 sur chaque axe.
Bien sur les valeurs peuvent être changées comme bon vous semble tant que le rendu reste correct et utilisable par l'utilisateur.


# BONUS
### LUMIERE SPECULAIRE => PHONG REFLECTION MODEL
- A quoi correspond la lumière spéculaire

&emsp;La lumière spéculaire est la réflexion de la lumière sur un objet, ce qui produit un effet "brillant" sans modifier totalement les spécificités des matériaux.

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/ReflectSpecular.png)
- Calcul de la lumière spéculaire

&emsp; Pour calculer la lumière spéculaire, nous allons devoir calculer l'angle de réflexion de la lumière pour savoir si la partie réflechie est transmise, vers la camera.

```
t_vec3  reflect;

reflect = ray.dir - (intersection.normal * (2.0 * ft_dot(light.dir, intersection.normal)));
```

Une fois le vecteur de réflexion calculé et normalisé, le ratio de cette lumière correspond au dot entre le vecteur de réflexion et la direction du ray de la camera inversé, mis à la puissance de la force de réflexion choisie. ([1 - 1000] Plus la force est haute et plus la réflexion sera précise et moins étalée.)
On peut ensuite calculer la couleur donnée par cette partie de la lumière:

```
t_color color;

color = light->rgb * light->ratio * reflect_ratio * Ratio_Reflexion_matiere;
```

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/EverythingPhongLight.png)

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/PhongShadow.png)
### CHECKERBOARD
- Checkerboard sphère
- Checkerboard plan
- Checkerboard cylindre
- Checkerboard cone

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/Checkerboard.png)
### MULTIPLE-SPOT DE LUMIERE + LUMIERE COLORE
- Calculs d'addition des lumières
- Calculs des mélanges de couleurs

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/MultipleSpot.png)
### INTERSECTION CONE
- Résoudre l'équation du second degré (t1, t2)
- Calculs des caps de haut et bas (t3, t4)
- Calcul de l'intersection
- Calcul de la normal

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/DiffuseConePlane.png)
### BUMB MAP
- A quoi correspond une normal map / Bump map
- Initialisation d'une bump map
- Exemple de calcul

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/img/BumpMap.png)
